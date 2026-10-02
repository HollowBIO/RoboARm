---
status: reference
tags:
  - robotics
  - resource
---

# Modern Robotics

**Modern Robotics: Mechanics, Planning, and Control** — Kevin M. Lynch & Frank C. Park.

The standard modern text. Freely available as a PDF from the authors, with an accompanying video lecture series and a software library. Primary resource for [[Robotic Arm Learning Plan|M1]] and M7.

## Relevant chapters

| Ch. | Topic | Module |
|---|---|---|
| 3 | Rigid-body motions, transforms | M1 — [[Homogeneous transforms]] |
| 4 | Forward kinematics | M1 |
| 5 | Velocity kinematics and the Jacobian | M1 |
| 6 | Inverse kinematics | M1 — [[Inverse kinematics closed form]] |
| 9 | Trajectory generation | M7 — [[S-curve trajectories]] |
| 11 | Robot control | M8 — [[PID tuning]] |

## Note on approach

This book uses **screw theory / product of exponentials** rather than Denavit–Hartenberg. It is the better formulation, but if it feels abstract on first contact, read Craig's *Introduction to Robotics* for [[Denavit-Hartenberg parameters]] and come back. Do not fight the abstraction for weeks — the arm does not care which convention you use.

## Other resources

- **Introduction to Robotics: Mechanics and Control** — Craig. Older, DH-centric, clearest on DH specifically.
- **Robotics, Vision and Control** — Corke. More practical, worked code examples. Pairs with his Python Robotics Toolbox. Its vision chapters cover M11.
- **OpenCV documentation** — the camera calibration and ArUco marker tutorials. M11 — [[Camera calibration]].
- **Multiple View Geometry in Computer Vision** — Hartley & Zisserman. Depth reference for homographies and camera models. M11.
- **Your linear rail's datasheet** — rated moments and stiffness. M3 — [[Linear rail axis]].
- **Shigley's Mechanical Engineering Design** — gears, shafts, bearings. A lookup reference, not a read-through. M3.
- **507 Mechanical Movements** — Brown. Short, old, excellent for seeing reduction options you would not invent. M3.
- **3D Printing Handbook** — Prusa. Genuinely good on orientation and failure modes. M4.
- **Practical Electronics for Inventors** — Scherz & Monk. Right level for M5.
- **The Art of Electronics** — Horowitz & Hill. Depth reference.
- **Making Embedded Systems** — Elecia White. Structuring embedded code. M6.
- **Trajectory Planning for Automatic Machines and Robots** — Biagiotti & Melchiorri. Thorough on jerk limitation. M7.
- **Your pump's datasheet** — the flow-versus-head curve sets fill time and nozzle bore. M5 and M7 — [[Dispensing control]].
- **HX711 datasheet** — load-cell amplifier, if you stop the fill by weight. M8 — [[Fill level sensing]].
- **Feedback Systems** — Åström & Murray. Free, rigorous introduction to control. M8.
- **Klipper source code** — study its stepper and motion modules. Its Pi-plus-MCU split is exactly the target architecture. M6.
