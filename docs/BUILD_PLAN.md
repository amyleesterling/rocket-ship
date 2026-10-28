# Build Plan: Sophia's Transport Maker

**Status:** Planning; no implementation shipped as part of this documentation update  
**Updated:** 2026-10-28  
**Companion:** [Research / reusable libraries and assets](./RESEARCH.md)

## The north star

A delightful, touch-first **3D vehicle invention garage** for kids. Start with a car, boat, broom, rocket, or simple primitive chassis. Swap and invent parts, alter shapes, paint with real and magical materials, press **GO!**, and use the creation in a small test world. Every creation can be saved and reopened for further tinkering.

Design ethos: expressive, beautiful, immediate, child-readable, playfully unscientific physics. **Let kids make impossible things and then make those things move.**

## Product principles / non-goals

1. **Vehicle-first creativity:** The default path is effortless customization, with optional increasingly advanced freeform building.
2. **Anything can be a vehicle:** Land, water, air, magic and ridiculous household objects share the same save/attachment model.
3. **No reading required for core play:** Large pictorial buttons, safe undo, immediate previews, simple prompts.
4. **Materials are first-class:** Separate paint/finish/texture per mesh/material slot; don't settle for a single body color selector.
5. **Build, then test:** At least one playable movement mode in the first shipped iteration.
6. **Local-first, child-safe:** No login, chat, public profiles, advertising or uploads in MVP.
7. **Browser-first:** iPad/phone touch, desktop mouse/keyboard; acceptable fallback graphics.
8. **Asset safety:** Curated models and tracked licenses only.

Not in MVP: multiplayer, public gallery, user-uploaded arbitrary models, photoreal racing simulation, real-time mesh sculpting, complex multiplayer economies, online accounts.

## Suggested implementation stack

- **Application:** React + TypeScript + Vite (lightweight single-page app).
- **3D:** Three.js + React Three Fiber + Drei (orbit camera, helpers, transforms).
- **State:** Zustand, with an explicit history stack for undo/redo.
- **Physics:** React Three Rapier for colliders, rigid bodies, triggers; custom arcade movement controllers for car/boat/broom.
- **Assets:** Manually approved GLB catalog beginning with Kenney Car Kit; original lightweight broom/boat as needed.
- **Finishes:** MeshStandardMaterial / MeshPhysicalMaterial, curated CC0 PBR textures and a lightweight emissive/bloom effects profile.
- **Persistence:** Dexie (IndexedDB) or small localStorage adapter for first spike; save JSON recipes rather than whole GLBs.
- **Build pipeline:** glTF Transform for mesh/texture optimization; optional GLB export using GLTFExporter.
- **Tests:** Vitest for configuration/state, Playwright for desktop/mobile UI, screenshots with visual baseline where practical.
- **Hosting:** Existing project's preferred GitHub Pages or Vercel deployment; defer final host decision until repo setup.

## Main UX: five steps

`CHOOSE → BUILD → PAINT → TEST DRIVE / FLY → SAVE`

1. **Choose:** Big picture cards for Car, Truck, Boat, Broom, Rocket, and 'Start with a Shape' (MVP can begin with Car / Boat / Broom).
2. **Build:** A carousel of bodies, wheels/feet, engines, roofs, seats, wings, propellers, fins, sails and magical attachments. Press/tap then snap at highlighted sockets.
3. **Paint:** Select part or entire invention; choose palette and material preset, adjust hue, optionally apply stickers.
4. **Test:** Hit a single bright GO! button; mode-specific simple controls (steer/forward/brake for land, float/turn for water, forward/turn/ascend for fly).
5. **Save:** Name/thumbnail/autosave, then resume from a gallery. Parent-approved sharing may come later.

### Interface layout

- Center: 3D viewport with gentle rotate/zoom; light sky-blue garage, floor grid, shadows.
- Top: Home / Undo / Redo / Save / Help.
- Bottom: large mode tabs **Build / Paint / Test**.
- Left or bottom sheet: visual thumbnails of parts in the selected category.
- Contextual popover: material or part-specific controls (size, color, rotate, duplicate, delete).
- Two complexity levels: **Easy** (guided sockets) and **Inventor** (free position, rotation, scale, primitive shapes).
- Do not rely on hover for mobile or child-accessibility.

## Proposed model and data contracts

The key investment is a **universal, data-driven attachment graph** so we don't hand-code every vehicle.

```ts
type MoveMode = 'drive' | 'float' | 'fly';
type SocketType = 'wheel' | 'wing' | 'engine' | 'seat' | 'roof' |
                  'mast' | 'decor' | 'universal';

type MaterialPreset =
  | 'glossy' | 'matte' | 'chrome' | 'gold' | 'wood'
  | 'crystal' | 'iridescent' | 'scales' | 'glow';

interface SocketDefinition {
  id: string;
  type: SocketType;
  position: [number, number, number]; // local coordinates
  rotation?: [number, number, number];
  mirrorOf?: string;
  sizeClass?: 'small' | 'medium' | 'large';
}

interface CatalogItem {
  id: string;
  label: string;
  category: 'body' | 'wheel' | 'engine' | 'wing' | 'decor' | 'shape';
  glbUrl?: string; // approved/locally bundled GLB
  compatibleSockets: SocketType[];
  materialSlots: string[];
  sockets?: SocketDefinition[];
  bounds?: [number, number, number];
  licensingRef?: string;
}

interface PartInstance {
  instanceId: string;
  catalogId: string;
  parentInstanceId?: string;
  socketId?: string;
  position: [number, number, number];
  rotation: [number, number, number];
  scale: [number, number, number];
  materials: Record<string, {
    preset: MaterialPreset;
    color: string;
    textureId?: string;
  }>;
}

interface SavedInvention {
  schemaVersion: 1;
  id: string;
  name: string;
  baseCatalogId: string;
  moveMode: MoveMode;
  parts: PartInstance[];
  createdAt: string;
  updatedAt: string;
  thumbnailDataUrl?: string;
}
```

**Design note:** The schema above is an implementation starting point, not production final. Add schema validation/migrations before user data accumulates. Keep copyrighted asset binaries out of saved JSON and public exports.

### Universal mounting rules

- Every base chassis defines named attachment sockets; compatible thumbnails highlight suitable locations.
- Small children tap a part to preview and attach it; advanced drag-and-snap is optional.
- Mount transforms are expressed in **parent-local coordinates**; an item can itself expose sockets.
- Mirrored placements place pairs of wheels/wings in a single action; also permit deliberately asymmetric creations.
- Basic collision/intersection warnings can be playful but should not block imagination.
- Allow goofy combinations with a `universal` socket; do not make the rules so strict that a broom cannot get headlights.
- Wheel size and mounting position should affect visual and collider dimensions; visual-only materials should not affect basic physics in MVP.

## Motion design

**Drive:** Steering + throttle + brake, approximate wheel radius/wheelbase, gravity, simple damping and self-right when upside down.  
**Float:** Water-plane buoyancy approximation, slow yaw, propeller/bobbing; no computational fluid simulation.  
**Fly:** Arcade 6-DOF-lite controls, ascend/descend, altitude cap, drift and automatic stabilization; broom/carpets work even with no visible engine.

Make **movement independent of chassis model**. A car with wings can choose `fly`; a bath tub with wheels can use `drive`. Kids should be able to switch modes rather than be forced to satisfy engineering rules.

## Prioritized delivery milestones

### Phase 0 — Repo & technical spike
**Objective:** Avoid selecting a stack that fails on touch devices or on common phones.

- [ ] Add basic Vite + React + TypeScript app scaffold, formatting and scripts.
- [ ] Mount R3F Canvas + Drei OrbitControls with proper touch gestures.
- [ ] Load one approved GLB, rotate it, focus camera, show shadows/lighting.
- [ ] Try 3 physical material presets (matte, chrome, crystal) with low/high graphics modes.
- [ ] Bench on a representative iPad/iPhone and desktop; record visual/performance observations.
- [ ] Audit/copy legally usable license notices for any imported library or model.

**Exit:** 3D object reliably loads and can be rotated/zoomed on touch + mouse, without console errors.

### Phase 1 — First playable garage (MVP)
**Objective:** A child can invent a car and immediately enjoy it.

- [ ] Import at least one car and normalize orientation, dimensions and wheel pivot coordinates.
- [ ] Build a visual category picker and materials palette.
- [ ] Swap **two or more wheel styles** and **three material finishes**.
- [ ] Independently paint body vs wheels vs glass.
- [ ] Add Undo / Redo and Reset.
- [ ] Add **GO!** driving loop: ground plane, steering, acceleration, braking, obstacle collisions, reset vehicle.
- [ ] Save and reopen a named creation locally with a thumbnail.
- [ ] Make all primary controls work on mobile without hover.

**Acceptance demo:** Pick a car → paint it purple chrome → swap wheels → drive around a tiny test area → save → reload → the same invention returns.

### Phase 2 — Universal assembly + three transportation families
**Objective:** Prove this is an inventor, not a car colorizer.

- [ ] Implement socket-based `VehicleAssembler` and editable part graph.
- [ ] Add body/engine/wheel/wing/decor categories with preview ghosts and snap highlights.
- [ ] Add **boat** and **broom** base chassis alongside car.
- [ ] Introduce float and fly movement controllers.
- [ ] Add attach/detach/duplicate/rotate/resize for selected parts.
- [ ] Support at least **12 interchangeable attachment models** with working transform and save semantics.
- [ ] Add a primitive shape drawer: cube, sphere, cylinder, cone, tube.
- [ ] Provide a clear **Easy** / **Inventor** toggle.
- [ ] Complete automated tests for schema serialization and undo/redo.

**Acceptance demo:** A broom with a rocket motor and a car steering wheel can be assembled, painted, flown, saved and reopened; a boat can float and steer.

### Phase 3 — Magical polishing
**Objective:** Turn a useful builder into something kids will adore.

- [ ] Add themed parts: butterfly wings, propellers, sails, balloons, levitation gems, exhaust shapes and magical trim.
- [ ] Add materials: iridescent, glowing, wood, dragon scales, translucent candy glass.
- [ ] Add moderated/preset effects: sparkles, vapor trails, engine lights, floating animations, sound toggle.
- [ ] Add charming worlds: workshop/garage, sunny test track, water cove, sky islands.
- [ ] Add creation gallery with duplicate, rename and category filters.
- [ ] Add high-quality thumbnails and screenshot/print feature.
- [ ] Optimize all shipping models and textures; ensure graceful no-bloom/reduced-motion alternatives.

**Acceptance demo:** A kid can make and fly a magical transportation object, take its portrait and see it in the gallery without assistance.

### Phase 4 — Extensions (not MVP commitments)
- [ ] More vehicle archetypes: helicopter, train, submarine, hoverboard, rocket, flying carpet, enchanted carriage, giant-shoe car.
- [ ] More realistic engine/wheel/tire tuning as an optional 'Engineer' tab.
- [ ] CSG cutouts/booleans, procedural body curves and decals.
- [ ] Course-building or invention challenges.
- [ ] Parent-reviewed share URLs / 3D GLB export with license and moderation checks.
- [ ] Optional creative challenge cards: 'Build a boat that can fly' or 'Give a broom a steering wheel'.

## First engineering backlog (in recommended order)

| Priority | Issue-sized task | Definition of done |
|---|---|---|
| P0 | Bootstrap React/Vite/R3F repo | Dev server and production build work |
| P0 | Create safe asset register | Each bundled asset has URL, author, license, obligations |
| P0 | Import/normalize car GLB | Correct up/front, bounded scale, preview thumbnail |
| P0 | Render and select vehicle parts | Click/tap highlights the selected mesh |
| P0 | Material slot editor | Change body/wheel independently, exact state survives re-render |
| P0 | Paint palette & part catalog UI | Functional on desktop and touch |
| P0 | Save/restore JSON recipes | Round-trip + migration unit test |
| P0 | Undo/redo | 20+ successive edits reversible without state corruption |
| P0 | Simple car test track | Steer, brake, reset, ground collision |
| P1 | Generic socket resolver | Candidate attachment snaps predictably |
| P1 | Boat and float controller | Floats, turns and resets |
| P1 | Broom and fly controller | Fly, turn, ascend/descend, stabilize |
| P1 | Attachments and basic shapes | Mix across chassis types |
| P1 | Invention gallery | Thumbnail, rename, duplicate, reopen |
| P2 | Magical VFX and worlds | Quality controls + reduced motion respected |
| P2 | Export GLB / sharing | Only with license and safety review |

## Suggested source tree

```text
src/
  app/
    App.tsx
    routes.ts
  components/
    BuilderToolbar.tsx
    CatalogPanel.tsx
    MaterialPalette.tsx
    GarageGallery.tsx
    SaveDialog.tsx
  scene/
    GarageScene.tsx
    Lighting.tsx
    CameraRig.tsx
    VehicleAssembler.tsx
    AttachmentPreview.tsx
    TestWorld.tsx
    controllers/
      DriveController.ts
      FloatController.ts
      FlyController.ts
  assets/
    catalog.ts
    definitions/
      car.ts
      boat.ts
      broom.ts
    materialPresets.ts
    asset-license-register.json
  state/
    editorStore.ts
    history.ts
  storage/
    database.ts
    serialize.ts
    migrations.ts
  utils/
    snap.ts
    gltf.ts
    bounds.ts
public/
  models/
  textures/
docs/
  RESEARCH.md
  BUILD_PLAN.md
```

## QA and performance guardrails

- Target responsive ~30 FPS on a representative recent iPad/phone, ~60 FPS on reasonable desktop; these are **targets**, not measured guarantees.
- Frame large costs: model/texture load size, draw calls, physics steps, transparent/reflective surfaces.
- Benchmark on real touchscreen Safari, not only desktop browser emulation.
- Ensure meshes, textures, physics bodies and event handlers are disposed when changing vehicles.
- Preserve edits across refresh and navigation; avoid persistence in memory only.
- Ensure rendering remains usable when WebGL hardware/performance features are limited.
- Add a pause/menu option, sound mute and reduced-motion/reduced-effects controls.
- Prevent external links, shopping, chat and random download behavior from the child play experience.
- Test graceful behavior on missing assets and damaged saved recipes.

## Risks and mitigations

| Risk | Likely impact | Mitigation |
|---|---|---|
| Third-party asset/license ambiguity | Cannot legally redistribute models or downloaded creations | CC0-first asset pack; register provenance; isolate attribution |
| Existing car models aren't modular | Wheels/doors may be baked into a single mesh | Begin with known modular kits; preprocess custom attachment anchors in Blender or GLB pipeline |
| User-built shapes break vehicle physics | Wheels collide, unsupported parts cause instability | Simplified collision proxies and controller-specific dynamics |
| Complex shaders overwhelm phones | Low FPS, crashes | Material quality levels, limited transparency and bloom |
| Too many editing controls for young children | Frustration | Easy/Inventor modes; image-first UI and one-tap undo |
| Saving becomes incompatible after updates | Lost inventions | Version every saved recipe and add migration tests |
| Public gallery opens safety/privacy issues | Moderation and child-data risks | Offline/local gallery initially; adult review before any public sharing |

## Decisions needed before building

1. **Style:** Bright, dimensional toy-like art (recommended), not photoreal commercial-car showroom.
2. **MVP roster:** One car + boat + broom (recommended), with later additions.
3. **First test area:** A tiny sunny garage/test track, then water/sky areas.
4. **Audience:** Designed for young children on touch screens, but enjoyable to adults.
5. **License posture:** CC0-first assets; adapt MIT/Apache code with notices; no unlicensed copying.
6. **UI priority:** One-tap swaps first, freeform manipulation second.

## Next concrete action

**Build the technical spike:** R3F scene + one approved Kenney car model + clickable wheel parts + 3 materials + touch orbit controls, then test it on an iPad. This settles the biggest unknowns early, creates a visible starting point, and supplies a working foundation for the rest of the invention garage.

---

Research inventory: [RESEARCH.md](./RESEARCH.md).  
This document is a plan; it does not claim the website is already implemented.
