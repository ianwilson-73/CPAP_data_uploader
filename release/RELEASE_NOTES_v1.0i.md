# CPAP AutoSync v1.0i

**Stable release based on v1.0i-beta3.**
OTA update from v1.0i-beta3 is supported.

---

## Bug Fixes

### Log Display: Out-of-Order Timestamps After Reboot

**Symptom:** The Web UI Logs tab showed interleaved lines from a previous boot mixed with current-boot lines after a device reboot, producing confusing out-of-order timestamps.

**Root cause:** When a reboot was detected, the client-side JavaScript attempted to stitch pre-reboot context by searching backwards in the log buffer for boot separators and splicing them together. This client-side log stitching was unreliable — it could merge lines from different boots in the wrong order.

**Fix:** On reboot detection, the client now clears its buffer entirely and triggers a full `/api/logs/full` backfill from the server. The server provides correctly-ordered multi-boot history (including pre-reboot context from NAND syslog), eliminating the need for any client-side cross-boot log stitching.

---

## Improvements

### System Tab Graphs: Time-Based Rendering with Gap Detection

**Problem:** The Heap History and CPU Load graphs on the System tab used index-based positioning with no timestamps. When the browser tab was backgrounded (e.g., switching apps on mobile), polling paused but old data points remained in the array. On return, the graph displayed stale data as if it were continuous recent activity, creating a misleading picture.

**Fix:**
- Data points now carry timestamps at collection time
- Points older than 2 minutes are pruned by age, not by count
- X-axis positioning is time-proportional (maps to the fixed 2-minute window)
- Gaps >6.5 seconds (2x the poll interval) break the SVG path — missing periods show as empty space rather than interpolated lines
- Switching between tabs (e.g., Dashboard → System) preserves data continuity for short absences

---

## Files Changed

- `include/web_ui.h` — Log display fix + time-based graph rendering with gap detection (client-side JavaScript only)
