---
title: Learning Plan — 4-DOF Robotic Arm
created: 2026-10-02
tags:
  - robotics
  - learning-plan
  - moc
status: not-started
target: 4-DOF arm with 50cm reach on a 1m rail, finds cups scattered on a table and fills them
---

# Learning Plan — 4-DOF Robotic Arm

> **The project this is aimed at:** a 4-axis arm with 50cm reach, riding a 1m linear rail, that finds cups scattered on a table, positions a nozzle over each and fills it to a target level — no overfill, nothing on the table.
>
> *Changed on 2026-10-02: the task went from carrying a full cup to filling one ([[Spec]], v1b), then the rail, the longer reach and scattered cups were added ([[Spec]], v1c).*

This is a **learning** plan, not a build plan. Each module teaches one body of knowledge and ends with a checkpoint tied to the arm. The modules are ordered so that you are always building the next piece of the real machine — not doing six months of theory and then starting.

**Realistic total: 300–400 hours** if you are learning most of this from scratch. At 8 hours a week that's roughly 9–12 months. Front-load [[#M1 Kinematics and Rigid-Body Math|M1]] and [[#M3 Mechanics Statics and Gear Reduction|M3]]; they are where under-preparation is most expensive.

---

## Start here

| Note | Purpose |
|---|---|
| [[Spec]] | What you are building. **Fill in the accuracy target first** — it decides whether M8 is optional. |
| [[Torque Budget]] | The numbers that show the arm mostly lifts itself. First-pass estimates, revise after CAD. |
| [[Dispensing control]] | Why this is a filling arm, not a pointing arm. Sets your accuracy budget and cycle time. |
| [[Linear rail axis]] | The fifth axis. Easy to drive, hard to make stiff, and it makes the arm redundant. |
| [[Build Log]] | Measurements and failures. Newest first. |
| [[Modern Robotics]] | Resource index for every module. |

## How to use this plan

- Each module has **Concepts**, **Resources**, **Build exercise**, and a **Checkpoint**.
- The **Checkpoint is the module.** Reading the resource does not complete it. If you cannot pass the checkpoint, you have not learned the module, regardless of how many hours went in.
- Convert each `[[wikilink]]` into its own note as you go. This file is a map of content; the understanding lives in the notes you write.
- Resources are listed by **title, not URL** — editions and links rot. Search the title.

### Anti-patterns to avoid

- [ ] **Tutorial hopping.** Following a "build a robot arm" video series end-to-end teaches you to reproduce *that* arm. The checkpoints here are deliberately not reproducible from any single tutorial.
- [ ] **Theory-first paralysis.** Do not finish all of [[Modern Robotics]] before touching a motor. M1 and M2 are enough to start M3.
- [ ] **Buying parts early.** You cannot specify motors before M3. Parts bought before then are a sunk cost that will bias your design.
- [ ] **Skipping the single-joint testbed.** The most common failure mode for this class of project is a fully assembled arm whose shoulder cannot lift the arm's own weight at full reach.

---

## M0 — Orientation

**~5 hours.** Understand what you are committing to before you commit.

### Concepts
- Degrees of freedom; why 4 DOF gives you position + *one* orientation, not arbitrary pose
- The **palletizing configuration**: base yaw → shoulder pitch → elbow pitch → level-keeping wrist
- The rail as a fifth, **prismatic** (sliding) joint, and **redundancy**: having more joints than the task needs — see [[Linear rail axis]]
- Why "fill a cup" is a dispensing problem as much as a positioning one — see [[Dispensing control]]
- Open-loop vs closed-loop actuation, and what each costs

### Build exercise
Write a one-page spec for your arm in your own words: reach, payload, workspace, accuracy, cycle time. Keep it. You will revise it in M3 and be glad you have the original. → [[Spec]]

### Checkpoint
- [ ] You can explain, without notes, what each of the five axes contributes to holding a nozzle vertically over a cup, which one is spare, and which one a passive linkage could replace.
- [ ] You can state your own accuracy requirement in millimetres and justify the number from the task.

---

## M1 — Kinematics and Rigid-Body Math

**~40 hours.** The mathematical core. Everything downstream assumes it.

### Concepts
- Homogeneous transformation matrices; composing and inverting them
- Rotation representations: matrices, Euler angles, quaternions — and when each fails
- **Denavit–Hartenberg parameters** and modern alternatives (product of exponentials)
- **Forward kinematics**: joint angles → end-effector pose
- **Inverse kinematics**: pose → joint angles. For a 4-DOF arm this has a *closed-form* solution — derive it yourself rather than reaching for a numerical solver
- Multiple solution branches (elbow-up vs elbow-down) and how to choose between them
- Prismatic joints in the chain, and **redundancy resolution**: the rail gives infinitely many solutions for one nozzle position, so you need a rule that picks the carriage position before the closed-form solution applies
- Workspace, reachability, and singularities
- The Jacobian; mapping joint velocities to end-effector velocity

### Resources
- **Modern Robotics: Mechanics, Planning, and Control** — Lynch & Park. The standard modern text; freely available as PDF, with an accompanying video course. Chapters 3–6 are your target.
- **Introduction to Robotics: Mechanics and Control** — Craig. Older and DH-centric, but the clearest treatment of DH parameters if Lynch & Park's screw theory feels abstract.
- **Robotics, Vision and Control** — Corke. More practical, worked examples in code.

### Build exercise
Derive the forward kinematics of *your* arm symbolically, rail included. Then derive the inverse kinematics by hand, on paper, using geometry and the law of cosines. Do not look up a solution for a generic arm.

### Checkpoint
- [ ] You can state your rule for choosing the carriage position for a given cup, and say what it optimises.
- [ ] Given the rail position and four joint angles, you can compute the nozzle position and orientation by hand for a simple pose, and your answer matches your code.
- [ ] Your IK function returns all valid solution branches and correctly reports when a target is unreachable.
- [ ] You can explain what happens to your IK near a singularity and how your code behaves there.

Related: [[Homogeneous transforms]], [[Denavit-Hartenberg parameters]], [[Inverse kinematics closed form]]

---

## M2 — Python Simulation Toolchain

**~25 hours.** Build the arm in software first. It is free to be wrong here.

### Concepts
- NumPy for transform math; why you vectorise rather than loop
- Visualising a kinematic chain in 3D (matplotlib is sufficient to start)
- Workspace sampling: Monte Carlo over joint ranges to find your *actual* reachable volume
- Self-collision checking, even crudely with capsules or spheres
- Physics simulation and what it does and does not tell you

### Resources
- **Robotics Toolbox for Python** — Peter Corke. Compare your own FK/IK against it to validate.
- **PyBullet** or **MuJoCo** for dynamics. Use these *after* your own kinematics works, as a check.
- **ikpy** — small, readable IK library. Read the source.

### Build exercise
Write a simulator that loads your link lengths and joint limits, plots the arm in 3D, animates a trajectory, and produces a workspace cloud. Overlay the footprint of the actual table you intend to use.

### Checkpoint
- [ ] Your workspace plot shows the *usable* volume after joint limits and self-collision are subtracted. Expect it to be noticeably smaller than the nominal 50cm sphere swept along the rail.
- [ ] You have confirmed from the plot that your reach actually covers the area you care about, at nozzle height above the tallest cup you intend to fill — or revised your link lengths because it did not.
- [ ] Your FK agrees with Robotics Toolbox to within floating-point error.

Related: [[Workspace analysis]], [[Self-collision checking]]

---

## M3 — Mechanics, Statics and Gear Reduction

**~45 hours.** The module that decides whether the arm works. Do not rush it.

### Concepts
- Static torque analysis: moments about each joint, worst-case pose
- Why torque demand is dominated by the arm's own weight as much as by payload
- **Gear reduction**: planetary, cycloidal, belt-and-pulley, worm. Ratio, efficiency, backlash, cost, and printability for each
- **Backdrivability** and why it is a safety property, not a performance one. A backdrivable shoulder lets the arm fall when power is lost — decide whether that is acceptable when it carries only a nozzle
- Worm gears: self-locking behaviour, lead angle, and the efficiency you trade for it
- **Gravity compensation** with springs or gas struts — sizing, and why it matters most for steppers (a stepper holding position draws full current doing nothing and heats accordingly)
- Bearings: radial vs thrust vs angular contact; preload; why the base needs a thrust bearing
- Linear motion: belts vs leadscrews, linear rails vs V-wheels, and moment loads on a carriage — why roll stiffness, not strength, decides how far the nozzle moves
- Structural stiffness, deflection under load, and why stiffness — not strength — is usually what limits a printed arm
- Backlash budgeting: how per-joint slop accumulates into end-effector error

### Resources
- **Shigley's Mechanical Engineering Design** — the reference for gears, shafts, bearings, and fasteners. Use it as a lookup, not a read-through.
- **507 Mechanical Movements** — Brown. Old, short, and excellent for seeing reduction and linkage options you would not have invented.
- Manufacturer engineering guides from bearing and belt suppliers (SKF, Gates) — free, rigorous, and specific.

### Build exercise
Produce a torque budget spreadsheet: every joint, worst-case static load, chosen reduction ratio, motor stall torque, and resulting safety factor. Then revise your M0 spec with what the numbers actually permit.

### Checkpoint
- [ ] You can calculate shoulder torque from scratch, including arm self-weight, and arrive at roughly 2.8 N·m static for this arm, most of it the arm itself.
- [ ] You can explain why 25% more reach cost 45% more shoulder torque.
- [ ] You can estimate how far the nozzle moves when the carriage rolls by 0.1° at full reach.
- [ ] You can justify your chosen reduction ratio and gear type for each joint, including the backdrivability decision at the shoulder.
- [ ] You have a backlash budget in degrees per joint and have converted it into millimetres of error at the nozzle.
- [ ] You have sized a gravity-compensation spring for the shoulder.

Related: [[Torque Budget]], [[Gear reduction types]], [[Backdrivability and safety]], [[Gravity compensation]], [[Linear rail axis]]

---

## M4 — CAD and Design for FDM

**~35 hours.** Turning the mechanical decisions into printable parts.

### Concepts
- Parametric modelling: drive dimensions from variables so link length is one number to change
- Assemblies, mates, and checking range of motion in CAD before printing
- **Design for FDM**: layer adhesion is the weak axis, so print orientation matters more than infill percentage for structural parts
- Designing around bought parts: bearing seats, belt paths, pulley clearance, fastener bosses, heat-set inserts
- Routing a tube and wiring through a drag chain on the rail and along the moving links: bend radius, slack at each joint, strain relief, flex life, and keeping water away from electronics
- Tolerances and fits for printed parts; test prints before committing to a large part
- Where to reinforce with bought aluminium extrusion rather than fighting FDM's limits

### Resources
- **FreeCAD** (open source), **Onshape** (free for public documents), or **Fusion 360** (free personal tier). Pick one and stay with it — switching mid-project costs weeks.
- **3D Printing Handbook** — Prusa. Genuinely good on orientation, strength, and failure modes.
- Your printer's own documentation on achievable tolerances.

### Build exercise
Model the shoulder joint completely: motor mount, reduction stage, bearing seats, and link interface. Print it. Measure how far reality diverges from the model.

### Checkpoint
- [ ] A fully parametric model where changing one link-length variable updates the assembly correctly.
- [ ] A printed shoulder bracket that holds a bearing with the correct fit on the first or second attempt.
- [ ] You can explain the print orientation you chose for each structural part in terms of load direction.

Related: [[Design for FDM]], [[Printed part tolerances]], [[Nozzle and tube routing]]

---

## M5 — Motors, Drivers and Power

**~30 hours.** Making things move, deliberately.

### Concepts
- Stepper fundamentals: full/half/microstepping, holding torque, and the **torque–speed curve** (torque collapses as speed rises — the single most misunderstood stepper property)
- Current limiting and why it, not voltage, sets stepper torque
- **TMC2209-class drivers**: UART configuration, StallGuard, quiet operation
- Missed steps: causes, detection, and why open-loop means silent failure
- Servos vs steppers vs BLDC for each joint, and why the wrist is the one joint a servo suits here
- Power budgeting: 24V supply sizing, inrush, decoupling, and separating motor power from logic power
- Limit switches, homing, **e-stop design**, and what the arm does when power is cut
- Driving a pump: switching an inductive load from a microcontroller, flyback protection, and PWM for flow control
- A belt-driven linear axis: steps per millimetre, belt tension, and why the rail needs no reduction when the shoulder needs 15:1
- Motor thermal behaviour under continuous holding load

### Resources
- **TMC2209 datasheet** and Trinamic's application notes. Dense but authoritative.
- **Practical Electronics for Inventors** — Scherz & Monk. Good breadth at the right level.
- **The Art of Electronics** — Horowitz & Hill. Reference, for when you need depth.
- Pololu's stepper-motor documentation — unusually clear on current limiting.

### Build exercise
**Single-joint testbed.** One NEMA 17, your chosen reduction, one TMC2209, a 50cm stick weighted to match your estimated arm mass, and the nozzle and filled tube at the end. Lift it. Hold it horizontal for ten minutes. Measure motor temperature. Separately, on the bench: switch the pump from the MCU and time a measured volume.

### Checkpoint
- [ ] The testbed lifts a representative arm-plus-nozzle load at 50cm and holds it without skipping steps.
- [ ] Motor temperature stabilises at a safe level while holding — if it does not, your gravity compensation or current limit is wrong.
- [ ] You can set a driver current limit correctly from the motor's rated current and explain what happens if you set it too high or too low.
- [ ] You know what the joint does when power is cut, and have decided whether that is acceptable.
- [ ] Cutting power stops the flow: the pump stops and nothing siphons.

> [!warning] Gate
> Do not proceed past this checkpoint. Everything after it assumes the actuation works. A failure here invalidates M4's mechanical design and is far cheaper to discover now.

Related: [[Stepper torque-speed curve]], [[TMC2209 configuration]], [[E-stop design]], [[Pump and tubing]]

---

## M6 — Embedded Firmware and Real-Time Control

**~40 hours.** The layer where timing matters.

### Concepts
- Why step generation is **hard real-time** and belongs on a microcontroller, never on Linux
- Timer-driven interrupts, step pulse timing, and jitter
- Multi-axis coordinated motion: generating steps for five axes — one linear, four rotary — that arrive together
- Acceleration ramping in firmware
- Serial protocol design: framing, checksums, acknowledgements, and what happens when a message is lost
- Splitting responsibility: **STM32/ESP32** owns real-time stepping, limits, e-stop, and pump cutoff; **Raspberry Pi 4** owns kinematics, planning, and interface
- Reading existing implementations — **Klipper** and **Marlin** are both worth studying for how they structure step generation

### Resources
- **Klipper** source code, particularly its stepper and trapezoidal-motion modules. The Pi-plus-microcontroller split is exactly your architecture.
- **Marlin** source for a more conventional all-on-MCU approach.
- STM32 HAL documentation, or the ESP-IDF timer and RMT peripheral docs if you go ESP32.
- **Making Embedded Systems** — Elecia White. Strong on structuring embedded code that stays maintainable.

### Build exercise
Firmware that accepts "move the rail and these four joints to these positions at this speed" over serial and executes it with coordinated, ramped motion, and that runs the pump on command with a hard upper limit on run time. Plus a Python host library that speaks the protocol.

### Checkpoint
- [ ] All five axes start and stop together, with no axis arriving early.
- [ ] A deliberately corrupted serial message is rejected rather than acted on.
- [ ] Hitting a limit switch mid-motion stops the arm immediately, in firmware, without the Pi's involvement.
- [ ] The pump stops by itself, in firmware, if the host goes silent mid-fill.
- [ ] You can explain why you did not put step generation on the Pi.

Related: [[Real-time step generation]], [[Serial protocol design]], [[Pi plus MCU architecture]]

---

## M7 — Trajectory Generation and Dispensing Control

**~30 hours.** This is the module that makes it a cup-filling arm specifically.

### Concepts
- Joint-space vs Cartesian-space trajectories, and when the distinction matters
- Trapezoidal vs **S-curve (jerk-limited)** velocity profiles
- **Residual vibration**: a printed arm rings after a fast stop and the nozzle swings with it, so settle time is part of cycle time- Input shaping to avoid exciting a known structural resonance
- The **placement budget**, `(opening − stream) / 2`, shared between arm error, cup-location error, carriage roll and nozzle tilt
- Flow rate, nozzle bore and stream velocity, and why bore must be matched to flow
- **Stop conditions**: timed dose, flow meter, weight, level — and which of these are open-loop on the cup
- **Overrun**: liquid that arrives after the stop decision, and two-stage (fast, then trickle) filling
- Dripping, siphoning, and withdrawal paths
- Via points, blending, and smooth path stitching

### Resources
- **Modern Robotics** — Lynch & Park, Chapter 9 (trajectory generation).
- **Trajectory Planning for Automatic Machines and Robots** — Biagiotti & Melchiorri. Thorough on S-curves and jerk limitation.
- Literature on **input shaping** for vibration suppression — developed for cranes and flexible arms, and it applies directly to a springy printed arm.
- Your pump's datasheet, for the flow-versus-head curve.

### Build exercise
A trajectory generator producing jerk-limited profiles with configurable limits, plus a dispensing routine that fills to a target volume in two stages. Measure dose repeatability over twenty fills before trusting it.

### Checkpoint
- [ ] You can calculate the allowable nozzle placement error for a given cup opening and stream diameter.
- [ ] You can predict fill time and overrun from flow rate and stop delay, and your measurements agree.
- [ ] Your generator enforces position, velocity, acceleration, *and* jerk limits simultaneously.
- [ ] The arm fills a cup to a target level with nothing on the table — the real checkpoint.

Related: [[Dispensing control]], [[S-curve trajectories]], [[Input shaping]]

---

## M8 — Closed-Loop Control and Encoders

**~30 hours.** Optional until it isn't. Required if the nozzle must hit a small opening, or if the fill must stop on what is actually in the cup.

### Concepts
- Open-loop limitations: missed steps are invisible, and backlash is unmeasured
- **AS5600-class magnetic encoders**: absolute position, mounting, magnet alignment, resolution
- Absolute vs incremental sensing, and why absolute means homing-free startup
- PID control: each term's role, tuning method, and the failure mode each causes when wrong
- Feedforward, and why gravity compensation in software complements it
- Calibration: joint zero offsets, link-length identification, and compensating measured backlash
- Closing the loop on the fill itself: a load cell or level sensor in place of a timed dose

### Resources
- **Feedback Systems** — Åström & Murray. Freely available; the clearest introduction to control that is still rigorous.
- **AS5600 datasheet** — short, and mounting tolerance matters more than you would expect.
- **Modern Robotics**, Chapter 11 (robot control).

### Build exercise
Add an encoder to one joint. Measure the actual backlash and repeatability. Close the loop and compare positioning error against open-loop.

### Checkpoint
- [ ] You have *measured*, not estimated, your per-joint backlash in degrees.
- [ ] You can state your arm's repeatability in millimetres at the nozzle, from measurement.
- [ ] You can tune a PID loop and explain which term you changed and why for each adjustment.

Related: [[Magnetic encoders]], [[PID tuning]], [[Calibration procedures]], [[Fill level sensing]]

---

## M9 — Integration, Calibration and Safety

**~25 hours.** Where a pile of working subsystems becomes a machine.

### Concepts
- Homing sequences and startup state
- Full-system calibration: establishing the base frame, measuring link lengths as built rather than as designed
- Error handling across the stack: what happens when IK fails mid-motion, when serial drops, when a limit trips
- Nozzle handling: confirming a cup is present before pumping, managing the drip afterwards, and a withdrawal path that keeps late drips in the cup
- Supply handling: knowing the tank is not empty before starting a fill, and priming the tube after a refill
- Safety: workspace limits in software, e-stop that works with firmware halted, and what the arm and the pump do on power loss mid-fill
- Logging and introspection — you cannot debug motion you cannot replay

### Build exercise
End-to-end task, with cup positions typed in as table coordinates: for each cup, move the carriage, confirm the cup is present, approach, lower the nozzle, fill to the target level, stop, wait out the drip, withdraw. Then return home. Run fifty fills and log every failure. Finding the cups without being told is M11.

### Checkpoint
- [ ] Fifty consecutive fills within your volume tolerance, with nothing on the table, across at least five cup positions spanning the rail and the reach.
- [ ] Every failure mode you encountered is logged, understood, and either fixed or documented.
- [ ] Someone else can start the arm from cold using only your written procedure.

Related: [[Homing and calibration]], [[Nozzle and tube routing]], [[Safety systems]]

---

## M10 — ROS 2 (Deliberately Late)

**~40 hours.** Only after the arm works.

### Why late
ROS 2 on a 2GB Pi 4 is tight, and the framework will absorb weeks that belong in M3 and M5. Plain Python over a serial protocol gets you a moving arm far sooner, and porting afterwards is straightforward *because* you will already understand every layer it abstracts. Learning ROS 2 first means debugging your kinematics and your middleware simultaneously.

### Concepts
- Nodes, topics, services, actions; the parameter system
- `tf2` and coordinate frame management
- **URDF**: describing your arm so the ecosystem can use it
- `ros2_control` and hardware interfaces
- **MoveIt 2**: motion planning, collision checking, and what it buys you over your own IK
- RViz for visualisation; Gazebo for simulation

### Resources
- Official ROS 2 tutorials, current LTS distribution.
- **MoveIt 2** documentation and its example configurations.
- **Programming Robots with ROS** — Quigley, Gerkey & Smart. ROS 1, but the concepts carry.

### Checkpoint
- [ ] A URDF of your arm that visualises correctly in RViz with accurate joint limits.
- [ ] MoveIt 2 planning and executing a collision-free motion on the real hardware.
- [ ] You can articulate what ROS 2 gained you — and what it cost in latency and complexity.

Related: [[URDF]], [[MoveIt 2]], [[ros2_control]]

---

## M11 — Finding the Cups (Perception)

**~35 hours.** Cups stand anywhere, so something has to tell the arm where. Numbered after M10 only to avoid renumbering — it depends on M9, not on ROS 2, and can come before M10.

### Why after M9
Get fifty good fills at typed-in coordinates first. Then, when a cup is missed, you know whether the arm or the camera was wrong. Bringing up vision and motion together means debugging both at once.

### Concepts
- The pinhole camera model, lens distortion, and **camera calibration** (measuring the camera's own geometry)
- **Homography**: a mapping from image pixels to coordinates on a flat plane. It works here because the table is flat
- **Parallax on the rim**: the homography is only true for points on the table, and the nozzle aims at the rim, which is not. A camera 1 m up sees a 95 mm tall rim, 400 mm off-centre, shifted by about 42 mm — more than the whole placement budget for a mug
- Fiducial markers (printed patterns such as ArUco) to fix the table's coordinate frame and tie it to the rail's zero
- Detecting a cup: circles and ellipses, colour, or a marker on the cup
- Alternatives to a camera: a distance sensor on the arm that sweeps for cups; marked positions on the table
- Cup-location error as one more share of the placement budget from M7

### Resources
- **OpenCV documentation** — camera calibration and ArUco tutorials.
- **Robotics, Vision and Control** — Corke. The vision half of the book.
- **Multiple View Geometry in Computer Vision** — Hartley & Zisserman. Reference, for when a homography misbehaves.

### Build exercise
A fixed overhead camera that reports the table coordinates of every cup's rim centre. Place a cup at ten tape-measured positions across the workspace and record the error at each.

### Checkpoint
- [ ] You have a measured cup-location error in millimetres, at the centre of the view and at its edges.
- [ ] You can explain the rim parallax correction and show it working on a tall cup near the edge.
- [ ] The camera's coordinates and the rail's coordinates agree: a point reported by the camera is where the nozzle goes.
- [ ] The arm fills three cups placed arbitrarily, with no coordinates typed in.

Related: [[Cup localization]], [[Camera calibration]]

---

## Dependency order

```
M0 ──> M1 ──> M2 ──┐
                   ├──> M3 ──> M4 ──> M5 ──[GATE]──> M6 ──> M7 ──> M9
                   │                                         │
                   └─────────────────────────────────────────> M8 (optional) ──> M9
                                                                     │
                                                                     └──> M11 and M10 (either order)
```

M1 and M3 are the two load-bearing modules. M5's testbed is a hard gate. M8 is optional for "timed fill of a wide mug in a known spot" and required for "fill a narrow opening" or "stop on the measured level". M11 is what turns "fill cups at typed-in positions" into "fill cups wherever they are".

---

## Progress

- [ ] M0 — Orientation
- [ ] M1 — Kinematics and Rigid-Body Math
- [ ] M2 — Python Simulation Toolchain
- [ ] M3 — Mechanics, Statics and Gear Reduction
- [ ] M4 — CAD and Design for FDM
- [ ] M5 — Motors, Drivers and Power ← **gate**
- [ ] M6 — Embedded Firmware and Real-Time Control
- [ ] M7 — Trajectory Generation and Dispensing Control
- [ ] M8 — Closed-Loop Control and Encoders *(optional)*
- [ ] M9 — Integration, Calibration and Safety
- [ ] M10 — ROS 2 *(after the arm works)*
- [ ] M11 — Finding the Cups *(after M9; before or after M10)*
