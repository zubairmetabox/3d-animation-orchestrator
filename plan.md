# Scroll-Based Model Animation Plan

## Project
3D Animation Orchestrator

## Goal
Animate the loaded `.glb` model based on page scroll position, with predictable timeline control and editable settings in the existing Customize sidebar.

## Product Behavior (Target)
- As the user scrolls, the model animation advances from start to end.
- Scrolling back reverses animation state.
- Behavior is smooth, deterministic, and frame-rate independent.
- Works whether the model has embedded animation clips or not.

## Technical Approach
### 1) Add a scroll progress source
- Create a scroll container/section with enough height (e.g. `300vh`) while canvas remains sticky/full viewport.
- Compute normalized progress `p` in `[0..1]` from scroll position.
- Expose `p` to the scene layer via React state/store.

### 2) Add animation engine layer
- For GLBs with clips:
  - Use `THREE.AnimationMixer`.
  - Select active clip (default first clip).
  - Drive mixer time directly: `mixer.setTime(p * clip.duration)`.
- For GLBs without clips:
  - Add fallback transform timeline (position/rotation/scale interpolation).

### 3) Add scroll animation settings (new UI section)
- `Enable Scroll Animation` (toggle)
- `Mode`: `scrub` (direct mapping) or `smoothed`
- `Start Offset` / `End Offset` (where in page scroll animation starts/ends)
- `Playback Direction`: normal/reverse
- `Smoothing` (if smoothed mode)
- `Clip` selector (if multiple embedded clips)
- `Fallback Transform Preset` (if no clips)

### 4) Update JSON config schema
- Add `scrollAnimation` object:
  - `enabled`, `mode`, `start`, `end`, `reverse`, `smoothing`, `activeClip`, `fallbackPreset`
- Ensure Apply/Format/Copy/Save/Load continues working with backward compatibility.

## Implementation Plan
## Phase 1: Core scroll plumbing
- Add scroll container + progress calculation hook.
- Show debug progress text temporarily in sidebar to validate correctness.

## Phase 2: Clip scrubbing integration
- On model load, detect `gltf.animations`.
- Initialize/cleanup `AnimationMixer` and action.
- Scrub clip time with scroll progress.

## Phase 3: Fallback timeline
- If no clips, apply transform interpolation to model root from progress.
- Add a simple preset set: `none`, `rotateY`, `liftAndRotate`, `scalePulse`.

## Phase 4: UI and config integration
- Add new “Scroll Animation” card in Customize panel.
- Bind controls to runtime behavior.
- Extend JSON config parsing/clamping/sanitization.

## Phase 5: Quality and edge cases
- Handle model replacement cleanly (dispose old mixer/actions).
- Clamp and guard invalid ranges (`start < end`).
- Preserve camera/orbit behavior with sticky canvas.

## Risks / Hiccups to Watch
- Large models + high-frequency updates can stutter.
  - Mitigation: keep calculations minimal, avoid re-renders per frame where possible.
- Some clips assume original frame progression and can look odd when scrubbed.
  - Mitigation: offer clip selector and fallback mode.
- Scroll container changes can break current layout.
  - Mitigation: implement with isolated wrapper and keep existing sidebars anchored.

## Acceptance Criteria
- Scroll visibly drives model animation forward/backward.
- Embedded animation clips scrub correctly.
- Fallback transform animation works when no clips exist.
- Settings are controllable in UI and persisted via JSON tools.
- No regression to current upload, orbit, lighting, and config workflows.

## Suggested Build Order (Single PR)
1. Scroll progress hook + sticky viewport structure.
2. Mixer clip scrubbing.
3. Fallback transform animation.
4. UI controls + JSON schema update.
5. Final polish and manual validation checklist.
