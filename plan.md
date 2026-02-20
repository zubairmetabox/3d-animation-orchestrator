# Scroll Animation Plan (Phased Delivery)

## Project
3D Animation Orchestrator

## Delivery Strategy
Ship in small, safe increments. Each phase must be fully stable before moving to the next.

## Current Status
- Phase 1: Completed
- Phase 2: Completed (Animate mode has full-width timeline shell + merged layer timeline rows)
- Phase 3: Completed (keyframe data model + write/overwrite path)
- Next: Phase 4 (track rendering from actual keyframe data)

## New Product Requirements Added
- Add a `Preview` action to test real scroll behavior as an end-user would experience it.
- Define what happens when timeline animation reaches the end.
- Add an export path for animation data so authored motion can be used on a website.

---

## Animation Completion Behavior (Definition)
When scroll reaches timeline end:
- Default behavior: hold final keyframed state.
- If user scrolls back up: reverse through timeline states.
- Optional future behaviors:
  - loop
  - clamp at start/end
  - ping-pong

For initial implementation: **hold final state + reversible on scroll back**.

---

## Preview UX (Definition)
Add a `Preview` button/toggle that:
- switches to animate runtime behavior
- hides or locks editing interactions
- allows real page scroll to drive timeline
- optionally shows a lightweight HUD (`current vh`, `progress`)
- has clear `Exit Preview` action to return to editing

Preview is not the same as Edit/Animate authoring toggle:
- Edit/Animate = authoring mode semantics
- Preview = runtime simulation mode for validation

---

## Export Strategy (Definition)
There are 2 export targets:

### A) Export Animation Data (Recommended first)
Export JSON payload containing:
- cameraView
- timeline length in `vh`
- tracks + keyframes + easing
- metadata/version

Use this JSON in a small runtime player package/component on website.

### B) Export Baked 3D Animation (Future)
Bake keyframes into GLTF animation clips and export `.glb`.
- higher complexity
- more fragile across arbitrary layer/property mappings

Initial plan: implement **A first**, then evaluate B later.

---

## Phase 0: Baseline Guardrails (No New UX)
### Scope
- Freeze current behavior as baseline.
- Add a short manual regression checklist for:
  - upload
  - layer list
  - transform controls
  - history/undo/redo
  - export

### Exit Criteria
- Baseline flows pass before Phase 1 starts.

---

## Phase 1: Mode System + Camera Contract (Completed)
### Scope
- Add global mode toggle: `Orbit` / `Animate`.
- Orbit mode:
  - orbit/pan/zoom enabled.
- Animate mode:
  - orbit/pan/zoom disabled.
  - camera uses captured authored view from Orbit mode.
- Auto-capture on switch to Animate.

### Exit Criteria
- Switching modes does not alter authored view unexpectedly.
- Animate mode camera is fixed and deterministic.

---

## Phase 2: Timeline Shell + VH Length (Completed)
### Scope
- Add full-width AE-style timeline shell in Animate mode.
- Add `timelineLengthVh` input.
- Add read-only playhead position.
- Display current timeline position (`vh` + normalized progress).
- Merge layer listing with timeline rows.
- Keep layers/modifier hierarchy legible (indent + shading + guides).

### Exit Criteria
- Timeline shell is stable and scalable (zoom + resize).
- Layer hierarchy is readable and modifier rows are accessible.

---

## Phase 3: Keyframe Data Model + Write Path Only (Completed)
### Scope
- Add in-memory keyframe track model:
  - `layerId`
  - `propertyId`
  - ordered `keyframes[]` with `{ atVh, value }`
- Add keyframe icon per modifier row in timeline.
- Clicking keyframe icon writes or overwrites at current timeline position.
- Add indicator state for keyframed properties.

### Exit Criteria
- Keyframes persist in state across mode switches.
- Overwrite at same timeline position works reliably.
- No playback interpolation yet.

---

## Phase 4: Timeline Track Rendering (Visual from Real Keyframes)
### Scope
- Render true keyframe markers in track lanes from stored keyframe data.
- Show only active property tracks with keyframes (or provide filter).
- Keep marker selection state only (no drag yet).

### Exit Criteria
- Marker positions match `atVh` precisely.
- Marker visibility reflects actual stored data.

---

## Phase 5: Playback Engine (Linear Interpolation + End State)
### Scope
- Evaluate keyframes from timeline position.
- Apply interpolated values in Animate mode.
- Enforce completion behavior:
  - hold final state at end
  - reverse when moving back

### Exit Criteria
- Timeline position deterministically drives animated values.
- End-of-animation behavior matches definition.

---

## Phase 6: Preview Mode
### Scope
- Add `Preview` button.
- Enter runtime-like preview state:
  - editing controls disabled/hidden
  - real page scroll drives timeline exactly like production behavior
- Add `Exit Preview`.

### Exit Criteria
- User can validate full scroll narrative before export.
- Preview and authoring states switch cleanly without losing edits.

---

## Phase 7: Persistence and Config Integration
### Scope
- Extend config JSON with:
  - mode
  - cameraView
  - timeline length (`vh`)
  - animation tracks/keyframes
  - preview/export metadata if needed
- Support Apply/Save/Load/Copy with backward compatibility.

### Exit Criteria
- Saved config restores authored view and animation setup.
- Older configs still load safely.

---

## Phase 8: Export v1 (Animation Data JSON)
### Scope
- Add `Export Animation` action.
- Produce versioned JSON bundle for runtime playback on site.
- Add `Copy Runtime Snippet` or starter integration instructions.

### Export Payload (v1)
```ts
{
  version: 1,
  cameraView: {...},
  timeline: { lengthVh: number },
  tracks: [...],
  defaults: {...}
}
```

### Exit Criteria
- Exported payload can be loaded by a simple runtime player and reproduces authored animation behavior.

---

## Phase 9: Keyframe Editing Essentials
### Scope
- Edit/delete keyframes.
- Optional retime by dragging markers.

### Exit Criteria
- Users can refine timing without recreating keyframes.

---

## Phase 10: Export v2 Investigation (Optional)
### Scope
- Feasibility study for baking authored timeline into `.glb` animation clips.
- Decide go/no-go based on stability and complexity.

### Exit Criteria
- Written technical decision and prototype outcome.

---

## Phase 11: UX Polish + Hardening
### Scope
- Timeline keyboard shortcuts.
- Better keyframe affordances.
- Performance pass for larger scenes.
- Final regression pass.

### Exit Criteria
- Stable experience for normal production usage.
