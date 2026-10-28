# Research: 3D Vehicle & Magical Transportation Builder

**Project:** Sophia's Transport Maker (`rocket-ship`)  
**Research date:** 2026-10-28  
**Status:** Technology and asset reconnaissance; no code integration performed yet.

## Executive finding

Build a **3D invention garage**, not just a car paint configurator. Kids should be able to start with a car, truck, boat, broom, rocket, flying carpet, or blank platform; change the shapes, materials, wheels and attachments; then drive, float, or fly the creation.

There is no single verified, permissively licensed library that delivers arbitrary vehicle assembly + material painting + physics + magical movement as a plug-and-play product. We can assemble a strong solution from existing pieces: **React Three Fiber, Drei, an editor/attachment layer, curated CC0 GLBs, Three.js physical materials, React Three Rapier, and local saved recipes**.

## 10 research tracks

| # | Area | Candidates | Finding / decision |
|---|---|---|---|
| 1 | 3D rendering | [Three.js](https://threejs.org/), [React Three Fiber](https://r3f.docs.pmnd.rs/), [Babylon.js](https://www.babylonjs.com/) | Prefer React Three Fiber (R3F) for a React-based website. Babylon is a valid alternative, not an additional renderer. |
| 2 | Existing car configurators | [FormDrive](https://github.com/nesdesignco/FormDrive), [CarFactory](https://github.com/Boguszera/CarFactory), [Custom Cars 3D](https://github.com/ViktorVelizarov/Custom-Cars-3D) | Study model-specific materials, wheel swaps, camera, animation, and UI. Most configure fixed meshes rather than invent new body shapes. |
| 3 | Assembly editors | [ThreeSpace](https://github.com/keiththomas1/threespace), [Sims-like House Builder](https://github.com/ch-bas/threejs-sims-house-builder), [Threepipe](https://threepipe.org/), [Drei controls](https://drei.docs.pmnd.rs/gizmos/transform-controls) | Reuse interaction patterns: select, move, rotate, scale, snap, undo/redo, save. Build vehicle-specific socket rules ourselves. |
| 4 | Modular vehicles | [Kenney Car Kit](https://kenney.nl/assets/car-kit), [KayKit City Builder Bits](https://kaylousberg.itch.io/city-builder-bits), [Quaternius Cars](https://quaternius.com/packs/cars.html), [AssetLoom Modular Car Kit](https://assetloom.io/assets/low-poly-modular-car-kit) | Start with Kenney/other CC0 kits; normalize dimensions, wheel pivots and socket positions during import. |
| 5 | Materials & lighting | [Poly Haven](https://polyhaven.com/), [ambientCG](https://ambientcg.com/), [MeshPhysicalMaterial](https://threejs.org/docs/#api/en/materials/MeshPhysicalMaterial) | Give kids glossy, metallic, wood, glass, pearlescent, crystal, glowing, and patterned finishes. Most shader effects need no heavy downloaded texture. |
| 6 | Shape customization | [React Three CSG](https://github.com/pmndrs/react-three-csg), [three-bvh-csg](https://github.com/gkjohnson/three-bvh-csg), Three.js primitives | Hybrid: parts library + procedural stretch/resize + optional additive/subtractive shape tools later. Avoid expensive CSG on every pointer-move. |
| 7 | Physics & movement | [React Three Rapier](https://github.com/pmndrs/react-three-rapier), [use-cannon](https://github.com/pmndrs/use-cannon), [Needle car demo](https://engine.needle.tools/samples/car-physics/) | Prefer Rapier rigid bodies/collisions, with deliberately forgiving arcade vehicle controllers. Use separate drive/hover/fly motion models. |
| 8 | Magic effects | [Drei Float](https://drei.docs.pmnd.rs/staging/float), [Trail](https://drei.docs.pmnd.rs/abstractions/trail), [Sparkles](https://drei.docs.pmnd.rs/staging/sparkles), [react-postprocessing](https://github.com/pmndrs/react-postprocessing) | Hover, glitter exhaust, engine glow, flight trails, portals, enchanted engines; configurable reduced-effects mode. |
| 9 | Save, share & export | [Dexie](https://dexie.org/), [GLTFExporter](https://threejs.org/docs/#examples/en/exporters/GLTFExporter) | Store the editable invention as JSON locally. A GLB export is optional and does **not** contain executable physics/gameplay logic. |
| 10 | Performance & mobile | [glTF Transform](https://gltf-transform.dev/), Draco/Meshopt/KTX2, lazy-loading | Preprocess GLBs; load only current vehicles and attachments; use pooled instancing and mobile graphics presets. |

## Existing codebases: reuse audit

### 1. FormDrive — best configurator architecture reference
- **Repository:** https://github.com/nesdesignco/FormDrive
- **Demo:** https://form-drive.vercel.app/
- **What it already demonstrates:** React/R3F, real PBR 3D cars, gloss and paint finishes, independent doors/windows/hood/trunk, wheel treatments, lights, camera presets and a studio interface.
- **Code license:** MIT, verified in repository `LICENSE` on 2026-10-28.
- **Asset caveat:** MIT covers **application source**, not the vehicle GLBs. The bundled Mustang, Tesla and concept vehicle have separate CC BY 4.0 notices; attribution requirements remain.
- **Use:** Architecture/UX reference; optionally reuse licensed application components with notices. **Do not** assume the vehicles are CC0.
- **Gap:** No general arbitrary snap-together invention system.

### 2. ThreeSpace — editor foundations
- **Repository:** https://github.com/keiththomas1/threespace
- **Code license:** MIT, verified in repository `LICENSE`.
- **Use:** Study selection, transforms, scene graph, property editing, serialization. Adapt toward guided child-friendly construction; not the main application framework by itself.
- **Gap:** Our own vehicle sockets, wheel/chassis logic, visual thumbnails and safeguards are still needed.

### 3. Sims-like House Builder — construction UX
- **Repository:** https://github.com/ch-bas/threejs-sims-house-builder
- **Reported license:** MIT in GitHub repository metadata; reconfirm license in the exact pinned commit before copying.
- **Use:** Snap/align, grouping, resize, undo, save/share and export patterns. Transfer the UX concepts from furniture/walls to wheels, wings, motors and hulls.
- **Gap:** Builds houses, not transport or vehicle physics.

### 4. Threepipe — high-level product configuration alternative
- **Website:** https://threepipe.org/
- **Repository:** https://github.com/repalash/threepipe
- **Reported license:** Apache-2.0; verify selected release and bundled examples.
- **Use:** Alternative to writing custom material inspectors, scene-viewer tools and serialization.
- **Decision:** Spike first: check whether the extra abstraction helps more than direct R3F controls.

### 5. CarFactory — useful surface control list
- **Repository:** https://github.com/Boguszera/CarFactory
- **Features described in README:** body paint, matte/gloss, wheels, headlights, glass tint/transparency, spoiler, metal finishes, localStorage persistence.
- **License:** No explicit root `LICENSE` found in initial audit; **do not copy code/assets** unless permission is confirmed.
- **Use:** Functionality checklist and interaction inspiration.

### 6. Vehicle Builder — part-configuration reference
- **Repository:** https://github.com/raptorcodedev/vehicle-builder
- **Relevant file:** https://github.com/raptorcodedev/vehicle-builder/blob/main/vehicleConfigs.js
- **Found in source:** wheel offset, wheelbase, tire/rim sizing, lift, defaults, per-vehicle add-ons and model paths.
- **License:** No explicit root license found; inspect architecture only.
- **Use:** Inform a typed, data-driven `VehicleDefinition` and attachment catalog.

### 7. Other examples to inspect, not automatically copy
- https://github.com/gorhorvat/product-configurator-3d — part selection/manipulation
- https://github.com/ViktorVelizarov/Custom-Cars-3D — multi-model paint/material configurator
- https://github.com/DyanGao/interactive_3d_car_configurator — R3F/Tailwind car UI
- https://github.com/manuelhintermayr/three-js-car-example — vehicle/physics implementation reference
- https://engine.needle.tools/samples/car-physics/ — touch/controller-friendly test-drive example

Treat the license of each repository **and each imported model/texture** as independent until reviewed. A public repository is not automatically open-source.

## Asset sources to shortlist

| Provider | Content | Licensing notes | Initial action |
|---|---|---|---|
| [Kenney Car Kit](https://kenney.nl/assets/car-kit) | Stylized cars, wheels and related pieces | CC0 (check pack download's included license) | Import for MVP |
| [KayKit](https://kaylousberg.itch.io/city-builder-bits) | Stylized world/vehicle props | Base pack reported CC0; paid extras can differ | Evaluate for workshop/test world |
| [Poly Haven](https://polyhaven.com/) | PBR materials, HDRIs, some 3D models | CC0 | Add 5–8 curated material/HDRI presets |
| [ambientCG](https://ambientcg.com/) | PBR surfaces | CC0 | Wood, fabric, stone, metals |
| [Poly Pizza](https://poly.pizza/) | Broad low-poly catalog | Mixed licenses per individual model | Discovery only until each asset is vetted |
| [Quaternius](https://quaternius.com/packs/cars.html) | Low-poly cars and environments | Site-wide license changed in Aug 2026; some legacy packs differ | Verify precise pack/version terms |
| [AssetLoom Modular Car Kit](https://assetloom.io/assets/low-poly-modular-car-kit) | Body shells + interchangeable wheels/parts | Paid/restrictive source redistribution conditions | Consider reference/purchase, not free baseline |
| [3DAssets.dev](https://3dassets.dev/) | GLB asset index/API | Verify each item’s declared license | Additional asset discovery |

### Proposed vehicle taxonomy
- **Land:** coupe, convertible, truck, jeep, racer, go-kart, bus, scooter, motorcycle, bicycle, wagon, train, tank-like treads.
- **Water:** boat, sailboat, submarine, hovercraft.
- **Air:** plane, helicopter, glider, balloon, blimp, rocket, spaceship.
- **Magic:** broom, flying carpet, enchanted carriage, floating teacup, dragon-shaped ship.
- **Everything-is-a-vehicle:** bathtub, sofa, giant shoe, pineapple, unicorn sled.

For MVP, hand-curate **a car, a boat, and a broom**. These force us to test drive, float, and fly without needing dozens of assets.

## Material system: proposed presets

| Preset | Three.js implementation | Notes |
|---|---|---|
| Glossy paint | roughness low, clearcoat | Tintable HSV/RGB |
| Matte paint | high roughness | Fast |
| Chrome / gold | metalness high + HDR lighting | Needs good environment |
| Wood | CC0 PBR texture maps | Tiling and UVs |
| Crystal / glass | transmission, IOR, thickness | Quality preset for mobile |
| Iridescent | physical material iridescence | Render compatibility test |
| Dragon scales | custom or tileable pattern | Could use simple normal/color maps |
| Lava / magic glow | emissive + optional bloom | Cap bloom for low-end devices |

Every attachable part should declare `materialSlots` (body, trim, glass, wheel, fabric, glow), and changing one slot should not recolor the entire model unless explicitly requested.

## Licensing and child-safety gates

1. Track each asset's **source URL, author, version/download date, license text, attribution, redistribution permission and texture dependencies** in an asset register.
2. Prefer clearly CC0 originals or original in-house assets for freely downloadable GLBs.
3. For CC BY models, provide required creator credit and license link wherever the model is used; assess exported copies separately.
4. Do not copy public-but-unlicensed GitHub code. Pin versions and retain MIT/Apache notices when code is used.
5. No unmoderated model upload, open chat, public gallery or external asset search in the child-facing MVP.
6. Save locally first; consider parent-reviewed sharing later. Avoid requiring child accounts or collecting identifying data.

## Architecture recommendation

```text
React + TypeScript + Vite
  ├── Three.js / React Three Fiber scene
  │   ├── AssetRegistry (approved GLBs, materials, previews, licenses)
  │   ├── VehicleAssembler (base + sockets + attachments)
  │   ├── MaterialPainter (per-part slots)
  │   ├── ShapeTools (scale/primitive shapes; CSG later)
  │   ├── MovementControllers (drive | float | fly)
  │   └── VFX (lights | sparks | hover | trails)
  ├── Zustand (editor state, undo/redo)
  ├── React UI (large tap-first controls)
  └── Dexie / IndexedDB (saved JSON recipes, thumbnails)
```

### Research conclusions and remaining tests

**Use immediately:** R3F + Drei, Kenney assets, MeshPhysicalMaterial presets, simple local save.  
**Prototype before selecting:** ThreeSpace vs direct controls vs Threepipe; Rapier vehicle controller; GLB asset processing/export; CSG performance.  
**Do not assume:** unlicensed configurator code can be copied; any model can be safely re-exported; physics rigs work on arbitrary custom shapes; desktop performance predicts iPad performance.

## Primary links

- https://github.com/nesdesignco/FormDrive
- https://github.com/keiththomas1/threespace
- https://github.com/ch-bas/threejs-sims-house-builder
- https://threepipe.org/
- https://github.com/pmndrs/react-three-fiber
- https://github.com/pmndrs/drei
- https://github.com/pmndrs/react-three-rapier
- https://github.com/pmndrs/react-three-csg
- https://github.com/gkjohnson/three-bvh-csg
- https://github.com/pmndrs/react-postprocessing
- https://kenney.nl/assets/car-kit
- https://polyhaven.com/
- https://ambientcg.com/
- https://gltf-transform.dev/

**Research boundary:** These are curated leads and documentation/code inspections, not an installation/performance test of every dependency. Validate licenses and current compatibility again when dependencies are added.
