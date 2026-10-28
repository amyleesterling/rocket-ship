# rocket-ship 🚀

**Soph's Transport Maker** — a playful 3D vehicle and magical transportation invention studio for kids.

Pick a car, boat, broom, rocket or something much stranger. Change the shape, swap wheels, attach wings and engines, paint parts with realistic or impossible materials, then **drive, float or fly** your creation. Save inventions to a private local gallery and keep tinkering.

> **Project status:** Research and implementation planning. The interactive builder is not implemented yet.

## Start here

- 📚 **[Research: 3D engines, configurators, assets, materials, physics and licenses](docs/RESEARCH.md)** — ten research tracks, source code examples, asset catalogs, licenses and recommendations.
- 🛠️ **[Build plan: architecture, phases and acceptance tests](docs/BUILD_PLAN.md)** — the product spec, first playable milestone, component/data model, safety and performance guardrails.

## The idea

**Choose → Build → Paint → Test → Save**

- **Choose a base:** a car, truck, boat, broom, rocket, flying carpet, or blank shape.
- **Build:** snap on wheels, wings, seats, sails, propellers, rockets, balloons and enchanted attachments.
- **Paint:** glossy paint, chrome, metal, wood, crystal, iridescence, glass, scales and magical glow — independently on different parts.
- **Test:** steer a vehicle, float across water, or fly over a whimsical world with forgiving arcade physics.
- **Save:** reopen and remix designs from a local, child-safe invention gallery.

## Proposed stack

React + TypeScript + Vite · Three.js + React Three Fiber + Drei · Zustand · React Three Rapier · curated CC0 GLB assets · Dexie/IndexedDB · glTF Transform.

Start by learning from [FormDrive](https://github.com/nesdesignco/FormDrive) (car configurator), [ThreeSpace](https://github.com/keiththomas1/threespace) (3D editor), and [Kenney Car Kit](https://kenney.nl/assets/car-kit) (starter models). Check the separate licenses on source code and every bundled 3D asset before reuse.

## Milestones

| Phase | Outcome |
|---|---|
| 0 — Technical spike | One approved car GLB, camera controls and three finishes, tested on touch |
| 1 — First playable garage | Paint, wheel swap, drive, save and reopen |
| 2 — Universal invention studio | Snap-on parts, body shapes, boat floating and broom flying |
| 3 — Magical polish | VFX, material library, worlds and a gallery |
| 4 — Extensions | More vehicles, challenges and parent-reviewed sharing |

**Immediate next step:** Render one approved Kenney car in React Three Fiber, make body/wheel selections work on touch, and apply matte, chrome and crystal materials. See [full plan](docs/BUILD_PLAN.md).

---

*Research collected October 28, 2026. Documentation is not a claim that the described features have already been built.*
