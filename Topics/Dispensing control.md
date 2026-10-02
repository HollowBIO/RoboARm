---
module: M7
status: draft
tags:
  - robotics
  - dispensing
  - calculation
---

# Dispensing control

Part of **M7** — see [[Robotic Arm Learning Plan]].

The problem that makes this a cup-filling arm rather than a generic pointing arm. The arm's job ends when the nozzle is over the cup; from then on the pump, the stream and the stop condition decide whether the cup is filled or the table is. Replaces [[Slosh dynamics]], which applied to the earlier cup-carrying task.

> [!info] Status
> Worked examples use a typical mug and round pump figures. They are **illustrations, not your numbers** — measure your cup and read your pump's datasheet.

## Placement budget

The stream must land inside the cup opening. For an opening of diameter `D` and a stream of diameter `d`:

$$e_{max} = \frac{D - d}{2} - \text{margin}$$

| Target | Opening `D` | Stream `d` | Allowable error before margin |
|---|---|---|---|
| Typical mug | 80 mm | 6 mm | **37 mm** |
| Narrow glass | 50 mm | 6 mm | 22 mm |
| Bottle neck | 25 mm | 6 mm | 9.5 mm |

Compare against the open-loop arm error estimated in [[Spec]] — roughly 19 mm at 500 mm reach. A mug still leaves room; a bottle neck does not. This is the comparison that decides whether M8 is optional.

The arm is no longer the only thing spending this allowance. With cups scattered and the base on a rail, the budget is shared:

| Error source | Rough size | Where it is worked out |
|---|---|---|
| Arm joints, open-loop | ~19 mm | [[Spec]] |
| Cup location | depends on method — a few mm to tens of mm | [[Cup localization]] |
| Carriage roll | ~1 mm per 0.1° | [[Linear rail axis]] |
| Nozzle tilt | see below | |

These do not simply add — they are unlikely to all be at their worst in the same direction at once — but a mug's 37 mm no longer looks generous.

### Nozzle tilt spends the same budget

A nozzle that is not vertical throws the stream sideways. For a fall height `H` and tilt `φ`, the offset is at most:

$$x \le H \tan\varphi$$

With the nozzle 20 mm above the rim of a 95 mm deep empty mug, `H ≈ 115 mm`:

```
φ = 2°  →  x ≈ 115 × 0.035 ≈  4 mm
φ = 5°  →  x ≈ 115 × 0.087 ≈ 10 mm
```

So wrist level error and position error draw on the same allowance. Lowering the nozzle into the cup shortens `H` and shrinks both this term and splash.

## Flow rate and fill time

$$t_{fill} = \frac{V}{Q}$$

| Pump class | Flow `Q` | Time for 300 ml |
|---|---|---|
| Small peristaltic | ~100 ml/min | 180 s |
| Small diaphragm | ~1.5 L/min | 12 s |

Fill time now sets the cycle time, not arm speed. The arm's move is a small fraction of either figure.

## Stream velocity and splash

Exit velocity from a nozzle of bore area `A`:

$$v = \frac{Q}{A}$$

For `Q = 1.5 L/min = 25 ml/s` through a 6 mm bore (`A = 0.283 cm²`):

```
v_exit   = 25 / 0.283 ≈ 88 cm/s ≈ 0.9 m/s
v_impact = √(0.88² + 2 × 9.81 × 0.115) ≈ 1.7 m/s   (into an empty mug)
```

At 100 ml/min the same bore gives about 0.06 m/s — the stream dribbles and clings to the nozzle rather than leaving cleanly. Bore diameter must be matched to flow rate.

## Knowing when to stop

| Method | Measures | Weakness |
|---|---|---|
| Timed pump | nothing — assumes `Q` is constant | drifts with tube wear, supply level, air in the line; overflows a smaller cup |
| Flow meter | volume delivered | does not know the cup's size or what was already in it |
| Load cell under the cup | mass in the cup (1 ml water ≈ 1 g) | cup must sit on the sensor, which scattered cups do not; stream impact disturbs the reading |
| Level sensor at the nozzle | distance to the liquid surface | foam, splashing, and cup geometry |

The first two are **open-loop** on the cup: they control what leaves the nozzle, not what is in the cup. The last two close the loop — see [[Fill level sensing]].

## Overrun

Liquid keeps arriving after the stop decision:

$$V_{overrun} \approx Q \cdot (t_{sense} + t_{stop}) + V_{in\,flight}$$

At 25 ml/s with 0.2 s of combined sensing and pump-stop delay, and roughly 1 ml in the air:

```
V_overrun ≈ 25 × 0.2 + 1 ≈ 6 ml
```

In an 80 mm mug (surface area ≈ 50 cm²) the level rises at about 5 mm/s, so 6 ml is roughly 1 mm of level. With 15 mm of freeboard there are about 3 seconds between "full" and "overflowing". Filling machines handle this with a **two-stage fill**: fast until near the target, then a trickle.

## After the stop

- **Dripping.** The nozzle holds liquid that falls after the pump stops. Wait over the cup, or reverse a peristaltic pump briefly to pull it back.
- **Siphoning.** If the supply sits higher than the nozzle, liquid can keep flowing with the pump off. A peristaltic pump blocks this by pinching the tube; many diaphragm pumps do not. See [[Pump and tubing]].
- **Withdrawal path.** Move up before moving sideways, so a late drip lands in the cup.
- **Tube volume.** A 6 mm bore holds about 28 ml per metre. The run from the separately installed pump to the nozzle, through the drag chain and up the arm, is roughly 2.5 m, so about 70 ml sits in the tube. It must be primed before the first dose is accurate, and it is what drips, siphons or drains back if anything lets it.
- **Drain-back.** The tank's height decides the fault: above the nozzle it siphons, below the nozzle the tube runs back into the tank when the pump stops and the next dose comes up short. Below is the safer side — a short dose is better than a flooded table — and a peristaltic pump or a check valve holds the column in place.

## What still matters about the arm's motion

The acceleration cap from the cup-carrying task is gone: nothing open is being carried. What remains is **settling** — a printed arm rings after a fast stop, and the nozzle swings with it. Either wait for it to settle before pumping or use [[S-curve trajectories]] to excite less in the first place. Trapezoidal profiles are an acceptable starting point here; they were not for the cup.

## Questions to answer

- [ ] What is your cup's opening diameter, and what placement error does that allow?
- [ ] What flow rate does your pump actually deliver through your tube and nozzle? (Time a measured volume.)
- [ ] How repeatable is a timed dose over twenty fills?
- [ ] How much arrives after the stop command, and how long does the nozzle drip?
- [ ] How long does the nozzle take to settle after a move, and does pumping before then matter?

## Sources

- Your pump's datasheet — the flow-versus-head curve
- Any introductory fluid mechanics text — continuity (`Q = vA`) and free-fall of a jet
- Biagiotti & Melchiorri, *Trajectory Planning for Automatic Machines and Robots* — residual vibration and jerk limitation
