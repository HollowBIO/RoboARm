---
module: M0
status: draft
tags:
  - robotics
  - spec
---

# Spec

Part of **M0** — see [[Robotic Arm Learning Plan]].

> [!tip] M0 deliverable
> Write this in your own words, then **do not edit the original**. Copy it to a new section when you revise after [[Torque Budget|M3]]. The gap between what you wanted and what the physics permitted is the most instructive artefact in the project.

## v1 — as intended (2026-10-02)

| Requirement | Value | Notes |
|---|---|---|
| Degrees of freedom | 4 | base yaw, shoulder pitch, elbow pitch, level-keeping wrist |
| Reach | 400 mm | nominal; usable workspace will be smaller — confirm in [[Workspace analysis]] |
| Payload | 750 g | a full ceramic mug |
| Task | pick up a full cup, move it, set it down without spilling | |
| Accuracy required | ? | **fill this in, in mm, and justify it from the task** |
| Cycle time | ? | bounded below by the acceleration cap — see [[Slosh dynamics]] |
| Workspace | ? | sketch the actual table area it must cover |

### Link lengths (provisional)
- Upper arm: ~220 mm
- Forearm: ~180 mm

### Known constraints going in
- 4 DOF gives position **plus one orientation** — cannot grasp a mug by the handle from an arbitrary direction. See [[Gripper design]].
- Shoulder must be **non-backdrivable** or the arm drops the cup on power loss. See [[Backdrivability and safety]].
- Acceleration capped near 2 m/s² by the spill condition, so the arm will be slow by design.
- Open-loop steppers give ~±1–2° per joint of backlash → roughly 15 mm of error at the gripper. Acceptable for an open table, **not** for placing into a holder. Encoders ([[Magnetic encoders]]) are the fix if accuracy matters.

## Open decisions

- [ ] Accuracy target in mm — this single number decides whether M8 is optional or mandatory
- [ ] Is placing into a tight holder/coaster in scope?
- [ ] Table area and mounting position
- [ ] Controller: off-the-shelf 3D printer board (BTT SKR-class) vs custom
- [ ] MCU: STM32 (on printer board) vs ESP32

## v1b — task changed to filling a cup (2026-10-02)

> [!note] Current task
> This section supersedes the **task** in v1. v1 is kept unedited above so the original intent stays on record.

The arm no longer carries liquid. It carries a **nozzle on the end of a tube**, positions it over a stationary cup, and a pump does the filling.

| Requirement | Value | Notes |
|---|---|---|
| Degrees of freedom | 4 | unchanged for now — see open decisions below |
| Reach | 400 mm | unchanged |
| Payload | nozzle + tube, mass ? | weigh it; [[Torque Budget]] assumes 0.10 kg until you do. Tube drag and stiffness act as load too |
| Task | position the nozzle over a stationary cup, fill to a target level, stop, withdraw without dripping outside the cup | |
| Wrist | keeps the nozzle pointing straight down | same level-keeping job as v1 |
| Accuracy required | ? | allowable error ≈ (cup opening diameter − stream diameter) / 2, less a margin. **Measure a real cup** |
| Fill stop method | ? | timed pump, flow meter, load cell under the cup, or level sensor |
| Cycle time | ? | now bounded by pump flow rate, not by an acceleration cap |
| Workspace | ? | the table area where a cup may stand |

### What this changes against v1
- The **2 m/s² acceleration cap is gone** — the arm carries no open liquid, so [[Slosh dynamics]] is archived and [[Dispensing control]] takes its place.
- Payload drops sharply. [[Torque Budget]] has been re-run as a first pass with an assumed 0.10 kg at the nozzle: shoulder ~1.9 N·m static, ~3.8 N·m target, about three-quarters of it the arm's own weight.
- The **non-backdrivable shoulder** was a safety requirement because of the cup. With a nozzle it is no longer obviously required — revisit in [[Backdrivability and safety]].
- A new subsystem appears: **pump, tube and fill sensing**, with water routed along an arm that also carries wiring.
- M7 shifts from slosh avoidance to dispensing control.

### Open decisions (v1b)
- [ ] Accuracy target in mm, from a measured cup opening — still decides whether M8 is optional
- [ ] How the arm knows when to stop filling
- [ ] Is the cup always in one fixed spot, or anywhere in the workspace? If anywhere, how does the arm find it?
- [ ] Pump type (peristaltic vs diaphragm) and where it is mounted
- [ ] Tube routing along the arm: bend radius, drag, and keeping water away from electronics
- [ ] Are 4 DOF still needed for a downward-pointing nozzle?
- [ ] Does the shoulder still need to be non-backdrivable?

## v1c — rail added, reach 500 mm, cups scattered (2026-10-02)

> [!note] Current geometry
> This section supersedes the **reach, axes and workspace** in v1b. The task in v1b — hold a nozzle over a cup and fill it — still stands.

The arm's base now rides a **1 m linear rail**, the reach grows to **500 mm**, and cups may stand **anywhere** in the workspace rather than in one known spot.

| Requirement | Value | Notes |
|---|---|---|
| Axes | 5 | rail (sliding) + base yaw, shoulder pitch, elbow pitch, wrist — see [[Linear rail axis]] |
| Rail travel | 1000 mm | rail itself must be longer by the carriage length |
| Reach | 500 mm | from the base axis; was 400 mm |
| Link lengths (provisional) | upper arm ~275 mm, forearm ~225 mm | scaled from 220 / 180 |
| Payload | nozzle + tube, mass ? | unchanged; [[Torque Budget]] assumes 0.10 kg |
| Task | find each cup scattered in the workspace, position the nozzle over it, fill to a target level, withdraw without dripping | |
| Cup location method | ? | typed-in coordinates first; camera later — see [[Cup localization]] |
| Accuracy required | ? | still (cup opening − stream) / 2 less a margin, but now shared by arm, rail, cup-location error and nozzle tilt |
| Fill stop method | ? | scattered cups rule out one load cell under a fixed spot |
| Cycle time | ? | pump flow rate plus rail travel, per cup |
| Workspace | nominal ~0.89 m² per side of the rail | where on the table is the rail mounted? Usable area is smaller |

### What this changes against v1b
- **Shoulder torque rises 45%** for 25% more reach: ~2.8 N·m static, ~5.6 N·m target. The reduction ratio goes from ~10:1 back up to ~15:1. See [[Torque Budget]].
- **One axis is now spare.** Five axes serve a task that needs four, so the inverse kinematics needs a rule for where to park the carriage.
- **Open-loop error grows with reach**: the ~15 mm estimated at 400 mm becomes roughly 19 mm at 500 mm.
- **The arm must find the cups.** This was an open question in v1b and is now a requirement. It adds a perception module (M11) to the plan.
- **Carriage roll becomes an error source**: 0.1° of roll moves the nozzle about 0.9 mm at full reach.
- **The tube run is longer** — roughly 2.5 m with slack — and must follow a moving base as well as four joints.

### Decisions settled
- [x] Cups may stand anywhere in the workspace, not in one fixed spot
- [x] Travel along the table by linear rail, not a wheeled base

### Open decisions (v1c)
- [ ] Accuracy target in mm, from a measured cup opening
- [ ] How the arm finds each cup
- [ ] How the arm knows when to stop filling, given cups are not on a fixed sensor
- [ ] Where the rail is mounted, and whether the arm reaches one side or both
- [ ] Rule for choosing the carriage position for a given cup
- [ ] Rail type and drive: linear rail or V-wheels, belt or leadscrew
- [ ] With the rail able to line up with any cup, is base yaw still needed?
- [ ] Pump type and mounting; tube routing through a drag chain
- [ ] Does the shoulder need to be non-backdrivable?
- [ ] Cups all one size, or mixed?

## v1d — water tank on the carriage (2026-10-02)

> [!note] Adds to v1c
> Everything in v1c stands. This section only moves the water supply.

The tank and pump ride on the rail carriage instead of sitting beside the machine. Only cables run through the drag chain; the tube starts at the carriage and runs up the arm to the nozzle.

| Requirement | Value | Notes |
|---|---|---|
| Tank location | on the carriage, not on the rotating base | low and centred over the rail, or offset as a counterweight — see [[Linear rail axis]] |
| Tank capacity | ? | 1 L assumed in the calculations: about three 300 ml cups per refill |
| Tank type | ? | rigid with a lid, or a soft pouch that cannot slosh |
| Pump location | on the carriage, next to the tank | |
| Tube run | ~0.8 m, carriage to nozzle | was ~2.5 m |

### What this changes against v1c
- **The rail carries more**: ~3.8 kg with a full tank against ~2.5 kg, needing ~25 N of belt force instead of ~20 N. A NEMA 17 on a belt still has nearly four times that.
- **The arm's joints are unaffected.** Shoulder, elbow and wrist torques in [[Torque Budget]] do not change.
- **Slosh is back, in the tank.** A 1 L rigid tank sloshes near 3 Hz when the rail accelerates and rocks the carriage.
- **Less liquid in the tube**: ~22 ml instead of ~70 ml, so less to prime and less to drip.
- **Drain-back replaces siphoning as the likely fault**: with the tank low, the tube can empty back into it when the pump stops.
- **Water now sits beside the motors and drivers** on the carriage, so a leak there matters more.
- **The machine needs refilling**, and needs to know when the tank is nearly empty.

### Open decisions (v1d)
- [ ] Tank capacity: more cups per refill against more mass on the rail
- [ ] Rigid tank or pouch
- [ ] How the tank is refilled, and how low level is detected
- [ ] Pump type that prevents drain-back, or a check valve

## v1e — tank installed separately (2026-10-02)

> [!note] Reverses v1d
> The tank goes back off the carriage. v1c stands in full; v1d is kept above as a record of an option that was considered and dropped.

The tank and pump sit beside the machine. A tube runs from the pump through the drag chain to the carriage, then up the arm to the nozzle.

| Requirement | Value | Notes |
|---|---|---|
| Tank location | separate from the machine, stationary | below nozzle height, so a fault drains back instead of siphoning |
| Tank capacity | ? | no longer limited by rail load |
| Pump location | beside the tank | |
| Tube run | ~2.5 m, pump to nozzle | through the drag chain, alongside the cables |

### Why
Compared in [[Linear rail axis]]. A separate tank costs one well-understood problem, a tube flexing in a drag chain. A carriage tank added slosh, a changing rail load, refills every few cups, low-level detection and a larger carriage — several of which would have shown up as nozzle movement.

### What this changes against v1d
- Moving mass on the rail returns to ~2.5 kg and stays constant; belt force ~20 N.
- No slosh anywhere on the machine.
- ~70 ml sits in the tube instead of ~22 ml: more to prime, more that can drip.
- The drag chain is sized by the tube's bend radius and must keep a leak away from the cables.

### Open decisions (v1e)
- [ ] Tank capacity and where it stands
- [ ] Pump type that prevents drain-back, or a check valve
- [ ] Tube material and bore that survive repeated flexing in the drag chain

## v2 — after M3

*(revise here once [[Torque Budget]] is done and arm mass is known from CAD)*
