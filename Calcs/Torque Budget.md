---
module: M3
status: draft
tags:
  - robotics
  - mechanics
  - calculation
---

# Torque Budget

Part of **M3** — see [[Robotic Arm Learning Plan]].

> [!info] Status
> These are **first-pass estimates**, not measured values. Arm self-weight is a guess until [[Design for FDM|CAD]] is done, and the payload is a guess until the nozzle and tube are weighed. Revise this note after M4, and again after the M5 testbed.

> [!note] Re-run for 500 mm reach on a rail (2026-10-02)
> Current geometry is [[Spec]], v1c: a 4-joint arm with 500 mm reach riding a 1 m linear rail, holding a nozzle. Earlier revisions are summarised below and kept in full at the bottom of this note.

## How the shoulder requirement has moved

| Revision | Payload | Reach | Static | Target | Self-weight share |
|---|---|---|---|---|---|
| v1 — carry a full cup | 0.75 kg | 0.40 m | 4.41 N·m | 6.6 N·m (1.5×) | 33% |
| v1b — hold a nozzle | 0.10 kg | 0.40 m | 1.92 N·m | 3.8 N·m (2×) | 77% |
| **v1c — nozzle, longer arm, rail** | 0.10 kg | 0.50 m | **2.78 N·m** | **5.6 N·m (2×)** | 79% |

Going from 0.40 m to 0.50 m is 25% more reach and **45% more shoulder torque**. A longer arm is heavier *and* has a longer lever, so the self-weight term grows roughly with the square of reach.

## Inputs

| Parameter | Value | Source |
|---|---|---|
| Reach | 0.50 m | spec, see [[Spec]] v1c |
| Payload | ~0.10 kg | **estimate** — nozzle, bracket and tube end. Weigh it |
| Tube along the arm | ~0.07 kg/m, filled | **estimate** — 6 mm bore silicone tube plus the water in it |
| Arm self-weight | ~0.90 kg | **estimate** — scaled from 0.75 kg at 0.40 m; links grow, the elbow motor does not |
| Self-weight effective radius | ~0.25 m | estimate |
| Moving mass on the rail | ~2.5 kg | **estimate** — arm, base and shoulder drives, carriage plate. The tank is installed separately |
| g | 9.81 m/s² | |

## Shoulder (J2) — worst case, arm horizontal

```
Payload:      0.10  kg × 0.50 m × 9.81 = 0.49 N·m
Self-weight:  0.90  kg × 0.25 m × 9.81 = 2.21 N·m
Tube:         0.035 kg × 0.25 m × 9.81 = 0.09 N·m
                                   Static = 2.78 N·m
```

**Self-weight is about 79% of the load.** The arm is mostly lifting itself. Every gram removed from the links, and every centimetre the elbow motor moves toward the shoulder, matters more than the payload does.

Not captured above: the tube's stiffness and drag, which resist joint motion and vary with pose. See [[Nozzle and tube routing]].

### Safety factor

| Safety factor | Requirement | Justification |
|---|---|---|
| 1.5× | 4.2 N·m | only valid if you choose to cap acceleration yourself |
| **2.0×** | **5.6 N·m** | **chosen** — nothing limits acceleration any more |

v1 used 1.5× because the spill constraint capped acceleration near 2 m/s². That cap came from the cup, and the cup is no longer carried, so the justification went with it.

### Gravity compensation is worth more now

The load is nearly constant: the nozzle is never picked up or put down. A spring can therefore cancel almost all of the 2.78 N·m, not just the self-weight share as in v1. See [[Gravity compensation]].

## All axes

| Axis | Motion | Static | Target (2×) | Approach |
|---|---|---|---|---|
| J0 | rail, 1 m travel | ~0 (horizontal, friction only) | ~20 N of belt force | NEMA 17 direct on a GT2 belt, no reduction — see [[Linear rail axis]] |
| J1 | base yaw | ~0 (no gravity moment) | ~1.9 N·m | NEMA 17 + ~5:1, thrust bearing |
| J2 | shoulder pitch | 2.78 N·m | **5.6 N·m** | NEMA 17 (59 N·cm) + ~15:1; worm is optional |
| J3 | elbow pitch | ~1.0 N·m | ~2.0 N·m | NEMA 17 + ~5:1 |
| J4 | wrist (nozzle vertical) | ~0.05 N·m | ~0.1 N·m | micro servo, or a passive parallelogram linkage |

### J0 derivation
Force to accelerate the moving mass at 2 m/s²: `2.5 × 2 = 5 N`, plus an allowance of ~5 N for bearing friction and drag-chain resistance → ~10 N, doubled to 20 N. A NEMA 17 on a 20-tooth GT2 pulley (pitch radius 6.4 mm) gives `0.59 / 0.0064 ≈ 93 N` at standstill. The rail is the easiest axis on the machine.

The water tank is installed separately and adds nothing to any axis. A tank on the carriage was considered — it would have raised the moving mass to ~3.8 kg — and dropped; see [[Linear rail axis]].

### J1 derivation
Two terms, both scaling with whatever acceleration you choose:

```
Own inertia:    (⅓ × 0.90 × 0.50² + 0.10 × 0.50²) × 4 rad/s² = 0.10 × 4 ≈ 0.40 N·m
Rail reaction:  (0.90 × 0.25 + 0.10 × 0.50) × 2 m/s²             ≈ 0.55 N·m
                                                      Worst case ≈ 0.95 N·m
```

The second term is new. When the carriage accelerates with the arm stretched out sideways, the arm's mass lags behind and tries to swing the base. The yaw drive has to hold against it.

### J3 derivation
Payload at ~0.27 m from elbow: `0.10 × 0.27 × 9.81 = 0.26 N·m`, plus forearm contribution ~0.7 N·m → ~1.0 N·m static.

### J4 derivation
Nozzle offset ~0.05 m from wrist axis: `0.10 × 0.05 × 9.81 = 0.05 N·m`.

## Reality check against hobby parts

| Option | Torque | Verdict at the shoulder |
|---|---|---|
| DS3218 hobby servo | 2.0 N·m stall | static load alone **exceeds** its stall figure |
| NEMA 17, direct drive | 0.5 N·m | **~11× short** |
| NEMA 17 (59 N·cm) + 10:1 belt or planetary | ~5 N·m after efficiency loss | **just short** of the 5.6 N·m target |
| NEMA 17 (59 N·cm) + 15:1 planetary | ~7.5 N·m after efficiency loss | **works**, backdrivable |
| NEMA 17 (59 N·cm) + 20:1 worm | ~8–9 N·m theoretical, less after efficiency loss | works, non-backdrivable, thinner margin than at 0.40 m |

**The shoulder needs real reduction**, and the extra 100 mm of reach pushed the ratio from ~10:1 back up to ~15:1. A spring that cancels most of the static load would bring it back down — which is the strongest argument yet for [[Gravity compensation]].

## Open questions

- [ ] Actual arm mass and centre of mass, from CAD rather than estimate
- [ ] Measured mass of nozzle, bracket and filled tube
- [ ] Actual moving mass on the rail
- [ ] Torque added by tube stiffness and drag across each joint's range
- [ ] Is the shoulder backdrivable, and if so what does the arm do on power loss?
- [ ] Measured motor temperature while holding the arm horizontal (M5 testbed)
- [ ] Spring rate and geometry for [[Gravity compensation]]

---

## v1b — nozzle at 400 mm reach, no rail (archived)

```
Payload:      0.10  kg × 0.40 m × 9.81 = 0.39 N·m
Self-weight:  0.75  kg × 0.20 m × 9.81 = 1.47 N·m
Tube:         0.028 kg × 0.20 m × 9.81 = 0.05 N·m
                                   Static = 1.92 N·m   →   3.8 N·m at 2×
```

| Joint | Static | Target (2×) | Approach |
|---|---|---|---|
| J1 base yaw | ~0 | ~0.6 N·m for inertia | NEMA 17 + modest reduction |
| J2 shoulder | 1.92 N·m | 3.8 N·m | NEMA 17 + ~10:1 belt or planetary |
| J3 elbow | ~0.72 N·m | ~1.4 N·m | NEMA 17 + ~5:1 |
| J4 wrist | ~0.05 N·m | ~0.1 N·m | micro servo or passive linkage |

## v1 — cup-carrying task (archived)

Kept for comparison. These figures assumed a 0.75 kg payload (a full ceramic mug) and an acceleration cap from [[Slosh dynamics]].

### Shoulder (J2)

```
Payload:      0.75 kg × 0.40 m × 9.81 = 2.94 N·m
Self-weight:  0.75 kg × 0.20 m × 9.81 = 1.47 N·m
                                  Static = 4.41 N·m
```

| Safety factor | Requirement | Justification |
|---|---|---|
| 2.0× | 8.8 N·m | conservative, assumes unconstrained dynamics |
| **1.5×** | **6.6 N·m** | **chosen** — valid because acceleration is capped near 2 m/s² |

### All joints

| Joint | Axis | Static | Target (1.5×) | Approach |
|---|---|---|---|---|
| J1 | base yaw | ~0 (no gravity moment) | ~2 N·m for inertia | NEMA 17 + modest reduction, thrust bearing |
| J2 | shoulder pitch | 4.41 N·m | **6.6 N·m** | NEMA 17 (59 N·cm) + ~20:1 **worm** — non-backdrivable |
| J3 | elbow pitch | ~2.1 N·m | ~3.2 N·m | NEMA 17 + ~10:1 |
| J4 | wrist (level-keeping) | ~0.37 N·m | ~0.8 N·m | single DS3218-class servo |

- J3: payload at ~0.22 m from elbow, `0.75 × 0.22 × 9.81 = 1.62 N·m`, plus forearm ~0.5 N·m → ~2.1 N·m static.
- J4: gripper + cup offset ~0.05 m from wrist axis, `0.75 × 0.05 × 9.81 = 0.37 N·m`.

### Reality check

| Option | Torque | Verdict |
|---|---|---|
| DS3218 hobby servo | 2.0 N·m stall | **3× short** at the shoulder |
| NEMA 17, direct drive | 0.5 N·m | **13× short** |
| NEMA 17 (59 N·cm) + 20:1 worm | ~8–9 N·m theoretical, less after efficiency loss | **works** |
