# Custom Thresholds Clearing - Final Solution

## Problem
Manual slider adjustments were persisting even after changing the problem or risk level. Additionally, after clearing customThresholds, sliders were defaulting to "respiratoire insufficientie" values instead of the current problem's values.

## Root Causes (Three Bugs)
1. **customThresholds persistence**: The `customThresholds` data was being saved when users manually adjusted sliders, but was never being cleared when the problem or risk level changed.
2. **Global variables initialization bug**: The `initializeGlobalParameterVariables()` function was looking for the problem in global localStorage instead of using the patient-specific medical info, causing it to always fall back to "respiratoire insufficientie" defaults.
3. **Slider threshold configuration bug**: The `getThresholdsConfiguration()` and `getConfigData()` methods were hardcoded to use 'respiratoire-insufficientie' as the baseline problem, ignoring the patient's actual problem.

## Solution

### 1. Clear customThresholds When Configuration Changes
Custom thresholds are now cleared in **two specific places** on the `alarm-overview.html` page:

**When Problem Changes** (`updateOrganCirclesBasedOnProblem` function):
```javascript
if (currentMedicalInfo.customThresholds) {
    console.log('🗑️ PROBLEM CHANGE: Clearing all customThresholds');
    delete currentMedicalInfo.customThresholds;
}
```

**When Risk Level Changes** (`updateOrganCirclesWithRisk` function):
```javascript
if (currentMedicalInfo.customThresholds) {
    console.log('🗑️ RISK LEVEL CHANGE: Clearing all customThresholds');
    delete currentMedicalInfo.customThresholds;
}
```

### 2. Fix Global Variables Initialization
Updated `initializeGlobalParameterVariables(patientId)` in `shared-data-manager.js` to:
- Accept a `patientId` parameter
- Get the problem from `medicalInfo.selectedProblem` for that patient
- Use the correct problem's Matrix-based defaults instead of always defaulting to "respiratoire insufficientie"

```javascript
initializeGlobalParameterVariables(patientId = null) {
    let currentProblem = '';
    let currentRiskLevel = 'low';
    
    if (patientId) {
        const medicalInfo = this.getPatientMedicalInfo(patientId);
        currentProblem = medicalInfo?.selectedProblem || '';
        currentRiskLevel = medicalInfo?.selectedRiskLevel || 'low';
    }
    
    // Use problem-specific Matrix defaults...
}
```

### 3. Fix Slider Threshold Configuration
Updated `getConfigData(patientId)` and `getThresholdsConfiguration(overallRiskLevel, currentProblem)` in `shared-data-manager.js` to:
- Accept patient-specific parameters
- Get the problem from `medicalInfo.selectedProblem` for that patient
- Use the patient's actual problem as the baseline for threshold configuration

Also updated `getThresholdsByTags(tags, patientId)` in `shared-data-manager.js` and the slider component to pass the `patientId` when loading thresholds.

```javascript
getConfigData(patientId = null) {
    let currentProblem = '';
    if (patientId) {
        const medicalInfo = this.getPatientMedicalInfo(patientId);
        currentProblem = medicalInfo?.selectedProblem || '';
    }
    // Use patient's actual problem for threshold configuration...
}

getThresholdsConfiguration(overallRiskLevel = 'low', currentProblem = '') {
    const normalProblem = currentProblem || 'respiratoire-insufficientie';
    const normalRanges = this.getMatrixBasedBaseRanges(normalProblem, overallRiskLevel);
    // Build configuration using patient's problem as baseline...
}
```

### 3. globalParametersChanged Event (Display-Only)
The `globalParametersChanged` event is now **display-only**. It:
- ✅ Updates slider visual positions from global variables
- ✅ Does NOT clear customThresholds
- ✅ Does NOT track problems
- ✅ Is dispatched when sliders are manually adjusted (to update displays on other pages)

### Files Modified

1. **alarm-overview.html**
   - Added customThresholds clearing when problem changes
   - Added customThresholds clearing when risk level changes

2. **shared-data-manager.js**
   - Fixed `initializeGlobalParameterVariables(patientId)` to use patient-specific problem
   - Now correctly loads problem-based Matrix defaults instead of always using "respiratoire insufficientie"

3. **circulatoir-settings.html**
   - Removed customThresholds clearing from `globalParametersChanged` event handler
   - Event now only updates slider displays

4. **respiratory-settings.html**
   - Removed customThresholds clearing from `globalParametersChanged` event handler
   - Event now only updates slider displays

5. **other.html**
   - Removed customThresholds clearing from `globalParametersChanged` event handler
   - Event now only updates slider displays

## Expected Behavior

### Scenario 1: Manual Adjustments During Normal Navigation ✅
1. User adjusts HR slider to 80-110
2. Values saved to `customThresholds.circulatoir.HR`
3. User navigates to respiratory page
4. User navigates back to circulatory page
5. **Result**: Slider shows 80-110 (manual adjustment persisted)

### Scenario 2: Problem Change Resets to Correct Defaults ✅
1. Patient has problem "Hartfalen" with manual adjustments (HR: 80-110)
2. User changes problem to "Sepsis" on alarm-overview
3. `customThresholds` are cleared
4. User navigates to circulatory settings
5. `initializeGlobalParameterVariables(patientId)` is called
6. Function gets problem "Sepsis" from patient's medicalInfo
7. Sets global variables to Sepsis Matrix defaults (e.g., HR: 70-120)
8. Slider initialized with Sepsis defaults
9. **Result**: Slider shows Sepsis defaults (70-120), not manual 80-110 ✅

### Scenario 3: Risk Level Change Resets Sliders ✅
1. User has manual adjustments (HR: 80-110)
2. User changes risk level from "Low" to "High" on alarm-overview
3. `customThresholds` are cleared
4. User navigates to circulatory settings
5. **Result**: Slider shows High-risk defaults, not manual 80-110

### Scenario 4: Tag Changes (Sepsis/Pneumonie) ⚠️
Tag changes (sepsis/pneumonie) do NOT clear customThresholds. This is intentional:
- Tags are additional conditions that modify base ranges
- Manual adjustments on top of tag-modified ranges should persist
- Only major configuration changes (problem/risk) clear manual adjustments

## Testing Checklist

- [ ] Manual adjustments persist when navigating between pages
- [ ] Manual adjustments are cleared when changing problem
- [ ] Manual adjustments are cleared when changing risk level  
- [ ] Tag selections (sepsis/pneumonie) do NOT clear manual adjustments
- [ ] `globalParametersChanged` event updates displays without clearing data
- [ ] Sliders show correct default ranges after problem/risk changes

## Key Files to Monitor

1. `alarm-overview.html` - Where clearing happens
2. `circulatoir-settings.html` - Circulatory sliders
3. `respiratory-settings.html` - Respiratory sliders
4. `other.html` - Temperature slider
5. `js/slider-component.js` - Slider component that loads/saves customThresholds

