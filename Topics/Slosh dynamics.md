---
module: M7
status: archived
tags:
  - robotics
  - dynamics
  - calculation
---

# Slosh dynamics

> [!warning] Archived (2026-10-02)
> This note belongs to the original **cup-carrying** task. The arm now holds a nozzle over a stationary cup and carries no open liquid, so nothing below constrains the design. Its place in M7 is taken by [[Dispensing control]]. Kept because the reasoning is sound and would apply again if the arm ever moves a filled cup.
>
> It came close to applying once more: a water tank on the rail carriage would slosh near 3 Hz by the frequency formula below. That was one reason the tank is installed separately instead — see [[Linear rail axis]].

Was part of **M7** — see [[Robotic Arm Learning Plan]].

The constraint that makes this a cup-carrying arm rather than a generic 750 g arm. It limits acceleration, which in turn **relaxes** the torque requirement in [[Torque Budget]].

## Static spill condition

Under lateral acceleration `a`, the liquid surface tilts to an angle θ where:

$$\tan\theta = \frac{a}{g}$$

Spill occurs when the tilted surface reaches the rim. For a cylindrical cup of radius `r` with freeboard `h` (vertical distance from liquid surface to rim):

$$\tan\theta_{max} = \frac{h}{r} \quad\Rightarrow\quad a_{max} = g\cdot\frac{h}{r}$$

### Worked example — typical mug

| Parameter | Value |
|---|---|
| Radius `r` | 0.04 m |
| Freeboard `h` | 0.015 m |
| `h/r` | 0.375 |
| **Static spill limit** | **3.68 m/s²** |

## Why the practical limit is lower

The static condition assumes a quasi-static tilt. Real motion excites the liquid's **first slosh mode**, producing a standing wave that reaches the rim well below the static limit.

First antisymmetric mode for a cylindrical container:

$$f_1 \approx \frac{1}{2\pi}\sqrt{\frac{1.84\,g}{r}}$$

For `r = 0.04 m`:

```
f1 ≈ (1/2π) × √(1.84 × 9.81 / 0.04)
   ≈ (1/2π) × √451
   ≈ 3.4 Hz
```

So **~3.4 Hz is the frequency to avoid** in the acceleration profile.

## Design consequences

- **Cap end-effector acceleration at ~1.5–2 m/s²** — roughly half the static limit, leaving margin for the resonant component.
- **Use jerk-limited S-curve profiles, never trapezoidal.** Trapezoidal profiles step acceleration discontinuously, which is broadband excitation — it feeds energy straight into the 3.4 Hz mode. See [[S-curve trajectories]].
- **Consider notch-filtering or [[Input shaping]]** around 3.4 Hz once the mode is confirmed by measurement.
- **Hold the wrist actively level** through the whole trajectory. Tilt consumes freeboard directly: at `h/r = 0.375`, a 2° wrist error eats roughly 13% of the budget before any acceleration is applied.
- **Low acceleration is why the 1.5× safety factor is defensible** in [[Torque Budget]] rather than the 2× a dynamically unconstrained arm would need.

## Time cost

At `a ≤ 2 m/s²`, a 0.40 m move takes at minimum about 0.9 s (accelerate half, decelerate half). Jerk limiting adds to this. The arm will be visibly slow — that is correct and unavoidable for this task, and it is a reason not to pay for a fast drivetrain.

## Questions to answer

- [ ] Measure your actual cup's radius and intended fill level — how much freeboard do you really have?
- [ ] Does the slosh frequency estimate hold when measured? (Film the surface, count oscillations.)
- [ ] How does `f1` shift as the cup empties, and must the trajectory adapt?
- [ ] Is a lid or a taller cup an acceptable cheat for early testing?

## Sources

- Lynch & Park, *Modern Robotics*, Ch. 9 — trajectory generation
- Biagiotti & Melchiorri, *Trajectory Planning for Automatic Machines and Robots* — S-curves and jerk limitation
- Gantry-crane anti-sway literature — the input-shaping results transfer directly
