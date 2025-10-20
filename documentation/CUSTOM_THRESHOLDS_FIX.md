# Custom Thresholds Clearing - Final Solution

## Problem
Manual slider adjustments were persisting even after changing the problem or risk level. They should only persist during normal navigation.

## Root Cause
The `customThresholds` data was being saved when users manually adjusted sliders, but was never being cleared when the problem or risk level changed. The slider component would reload these saved values on every page load, making manual adjustments "sticky" across problem changes.

## Solution

### Where customThresholds Are Cleared
Custom thresholds are now cleared in **two specific places** on the `alarm-overview.html` page:

1. **When Problem Changes** (`updateOrganCirclesBasedOnProblem` function):
```javascript
if (currentMedicalInfo.customThresholds) {
    console.log('🗑️ PROBLEM CHANGE: Clearing all customThresholds');
    delete currentMedicalInfo.customThresholds;
}
```

2. **When Risk Level Changes** (`updateOrganCirclesWithRisk` function):
```javascript
if (currentMedicalInfo.customThresholds) {
    console.log('🗑️ RISK LEVEL CHANGE: Clearing all customThresholds');
    delete currentMedicalInfo.customThresholds;
}
```

### How globalParametersChanged Event Works
The `globalParametersChanged` event is now **display-only**. It:
- ✅ Updates slider visual positions from global variables
- ✅ Does NOT clear customThresholds
- ✅ Does NOT track problems
- ✅ Is dispatched when sliders are manually adjusted (to update displays on other pages)

### Files Modified

1. **circulatoir-settings.html**
   - Removed customThresholds clearing from `globalParametersChanged` event handler
   - Event now only updates slider displays

2. **respiratory-settings.html**
   - Removed customThresholds clearing from `globalParametersChanged` event handler
   - Event now only updates slider displays

3. **other.html**
   - Removed customThresholds clearing from `globalParametersChanged` event handler
   - Event now only updates slider displays

4. **alarm-overview.html**
   - Added customThresholds clearing when problem changes
   - Added customThresholds clearing when risk level changes

## Expected Behavior

### Scenario 1: Manual Adjustments During Normal Navigation ✅
1. User adjusts HR slider to 80-110
2. Values saved to `customThresholds.circulatoir.HR`
3. User navigates to respiratory page
4. User navigates back to circulatory page
5. **Result**: Slider shows 80-110 (manual adjustment persisted)

### Scenario 2: Problem Change Resets Sliders ✅
1. User has manual adjustments (HR: 80-110)
2. User changes problem from "Respiratory" to "Sepsis" on alarm-overview
3. `customThresholds` are cleared
4. User navigates to circulatory settings
5. **Result**: Slider shows Sepsis defaults (70-120), not manual 80-110

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

