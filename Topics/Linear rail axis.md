---
module: M3
status: draft
tags:
  - robotics
  - mechanics
  - calculation
---

# Linear rail axis

Part of **M3** — see [[Robotic Arm Learning Plan]].

The arm's base rides a carriage along a 1 m track. This is a fifth axis, J0, and it is **prismatic**: it slides rather than rotates. It extends the workspace from one disc into a long strip so that cups scattered along the table are reachable.

> [!info] Status
> Moving mass and friction are **estimates**. Revise once the carriage is designed in CAD.

## Drive force

| Parameter | Value | Source |
|---|---|---|
| Travel | 1.0 m | [[Spec]] v1c |
| Moving mass | ~2.5 kg | estimate — arm, base and shoulder drives, carriage plate. The tank is not on the carriage |
| Acceleration | 2 m/s² | a choice, not a limit |
| Friction and drag chain | ~5 N | estimate — the chain now carries a filled tube as well as cables |

```
Inertia:   2.5 kg × 2 m/s² = 5 N
Friction:                    5 N
                     Total = 10 N   →   20 N at 2×
```

A NEMA 17 (0.59 N·m) on a 20-tooth GT2 pulley, pitch radius 6.4 mm:

```
F = 0.59 / 0.0064 ≈ 93 N at standstill
```

More than four times the target with no reduction at all. Unlike the shoulder, this axis carries no gravity load, which is why it is easy.

## The tank stays off the carriage

The water tank and pump are installed separately, beside the machine, and a tube follows the carriage through the drag chain ([[Spec]], v1e). A tank riding on the carriage was considered and dropped:

| Area | Tank on the carriage | Tank installed separately (chosen) |
|---|---|---|
| Tube run to the nozzle | ~0.8 m, ~22 ml held | ~2.5 m, ~70 ml held, flexing in the drag chain |
| Slosh | rocks the carriage near 3 Hz unless a pouch or baffles are used | none — the tank never moves |
| Capacity | ~3 cups per litre, a kilogram on the rail per litre | any size |
| Moving mass | ~3.8 kg full, falling as it drains | ~2.5 kg, constant |
| Carriage | larger, with tank and pump mounts | the arm's base only |

The separate tank costs one well-understood problem — a tube in a drag chain — and avoids several small ones that would have fed into nozzle movement. If the machine ever needs to be self-contained, a **soft pouch** on the carriage is the version to revisit: no free surface, so no slosh.

### What the drag chain has to do

- Carry the tube without bending it tighter than its minimum bend radius, so the chain is sized by the tube, not the cables.
- Survive the flexing: every rail move bends the same stretch of tube.
- Keep a leaking tube from wetting the cables that share the chain.

## Resolution and travel time

One pulley revolution moves the belt 40 mm. At 200 steps per revolution and 16× microstepping:

```
3200 microsteps / 40 mm = 80 steps/mm   →   0.0125 mm per microstep
```

Rail positioning error is negligible next to the arm's. For a full 1 m traverse at 2 m/s² up to 0.5 m/s:

```
Accelerate:  0.25 s over 0.0625 m
Cruise:      0.875 m / 0.5 m/s = 1.75 s
Decelerate:  0.25 s over 0.0625 m
                         Total ≈ 2.25 s
```

Small against a fill time of 12 s or more — see [[Dispensing control]].

## Carriage stiffness is the real problem

The arm stretched out sideways puts a twisting load on the carriage about the rail's long axis, equal to the shoulder's static torque (~2.8 N·m, see [[Torque Budget]]). Any rail can carry that. What matters is how far the carriage **rolls** under it, because the arm amplifies the angle:

```
Nozzle deflection ≈ reach × tan(roll)

0.1° of roll  →  500 mm × 0.00175 ≈ 0.9 mm
0.5° of roll  →  500 mm × 0.00873 ≈ 4.4 mm
```

A single narrow rail resists roll poorly. Two parallel rails, or one rail with a wide carriage on spaced bearings, resist it far better. This deflection draws on the same placement budget as everything else.

## Reaction on the base yaw

When the carriage accelerates with the arm out sideways, the arm's mass lags and tries to swing the base:

```
(0.90 × 0.25 + 0.10 × 0.50) kg·m × 2 m/s² ≈ 0.55 N·m
```

That is more than the yaw joint needs for its own motion. Either the yaw drive holds against it, or the rail accelerates more gently when the arm is extended.

## Redundancy

Five axes now serve a task that needs four: nozzle position (three numbers) plus keeping the nozzle vertical (one). The spare axis means **infinitely many joint combinations reach the same cup**, and the inverse kinematics needs a rule for choosing one. Candidates:

| Rule | Effect |
|---|---|
| Carriage directly opposite the cup | simplest; arm always works perpendicular to the rail, yaw barely moves |
| Least carriage travel from the current position | fastest between nearby cups |
| Keep the arm at mid-extension | best stiffness and accuracy; avoids both full stretch and the dead zone near the base |

Once the rule fixes the rail position, the remaining four joints have the same closed-form solution as before — see [[Inverse kinematics closed form]].

## Nominal workspace

Rail along one edge, arm reaching to one side:

```
Strip:      1.0 m × 0.5 m       = 0.50 m²
End caps:   ½ × π × 0.5²        = 0.39 m²
                        Nominal ≈ 0.89 m²   (double if both sides are reachable)
```

The usable area is smaller: joint limits, the dead zone close to the base, and the height the nozzle must clear all subtract from it. Confirm in [[Workspace analysis]].

## Questions to answer

- [ ] Belt or leadscrew for 1 m of travel, and why?
- [ ] Linear rail or V-wheels on extrusion: which gives the roll stiffness you need?
- [ ] How long must the rail be to give 1 m of travel once the carriage length is added?
- [ ] How much does the nozzle actually move when you push sideways on the extended arm?
- [ ] Which redundancy rule did you choose, and what does it optimise?
- [ ] What bend radius does your tube tolerate when full, and what drag chain does that require?
- [ ] How much force does the filled tube in the chain add to the rail's friction?

## Sources

- Your linear rail's datasheet — rated moments and stiffness per carriage
- Belt manufacturer design guides (Gates) — tension, tooth load, stretch
- Klipper and Marlin configuration references — steps per millimetre on a belt axis
