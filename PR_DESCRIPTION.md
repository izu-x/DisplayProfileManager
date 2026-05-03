# Pull Request: Seamless "Blink-Free" Display Profile Switching

## Description
This PR significantly optimizes the display profile application process to eliminate monitor blinking (driver re-syncing) when switching between profiles with identical resolutions or layouts. It transitions the application from a multi-stage legacy API approach to a modern, atomic `SetDisplayConfig` sequence.

## Key Changes

### 1. Atomic "Single-Step" Application
*   **Consolidated API Calls**: Resolution, Refresh Rate, and Layout (Topology) are now consolidated into a single atomic `SetDisplayConfig` transaction. This mimics the behavior of the native Windows "Display Settings" app, allowing the driver to transition all states simultaneously without multiple blinks.
*   **Source/Target Mode Pre-processing**: Updated `ApplyDisplayTopology` and `ApplyDisplayPosition` to manually inject resolution and refresh rate data into the source/target mode buffers before they are sent to the Windows display subsystem.

### 2. Deep-State "Smart Skip" Logic
*   **System State Snapshots**: All major display application methods now take a snapshot of the current system configuration before performing any work.
*   **Multi-Attribute Comparison**: Implemented deep comparison checks for:
    *   Active/Inactive monitor paths (Topology)
    *   Monitor rotation
    *   Source resolution (Width/Height)
    *   Target refresh rate (Rational Numerator/Denominator)
    *   Desktop positions (X/Y coordinates)
*   **Zero-Call Redundancy**: If the current system state matches the target profile's settings, the application now skips the expensive `SetDisplayConfig` call entirely. This makes switching between identical profiles instantaneous.

### 3. Optimized HDR Management
*   **Smart Toggling**: In addition to layout checks, the application now verifies the current HDR state of each monitor. HDR toggling is only performed if the target state differs from the current state, preventing unnecessary re-syncs.
*   **Reliable State Detection**: Improved the logic for detecting HDR support and current state to ensure accuracy on high-end panels (like the Samsung Odyssey G9).

### 4. Robustness & Fallback
*   **Safety Reverts**: Maintained a fallback mechanism in the event of an atomic configuration failure. The application will attempt to roll back to the previous stable state if a driver rejects a complex combined layout change.
*   **Clearer Logging**: Added detailed debug logging to trace exactly which display attributes triggered a change or why a change was skipped.

## Related Issues
- Resolves: Monitor blinking/flickering on profile switch between identical or similar resolutions.
- Resolves: Redundant HDR re-syncs during profile switches.

## Testing Performed
- [x] Verified atomic resolution/layout switching on multi-monitor setups.
- [x] Confirmed "Smart Skip" logic prevents all API calls when profiles are identical.
- [x] Verified HDR state preservation during seamless transitions.

---
