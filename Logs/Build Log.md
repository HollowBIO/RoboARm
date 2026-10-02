---
status: active
tags:
  - robotics
  - log
---

# Build Log

Newest entries at the top. See [[Robotic Arm Learning Plan]] for the module structure.

> [!tip] What to log
> Log **failures and measurements**, not intentions. "Printed shoulder bracket, bearing seat 0.3 mm too tight, reprinted at 0.15 mm clearance" is worth keeping. "Worked on CAD" is not.
>
> Measurements that belong here and nowhere else: motor temperatures, measured backlash per joint, actual part masses, print clearances that worked, pump flow rates, dose volumes and overrun you measured. These are the numbers that turn the estimates in [[Torque Budget]] into facts.

---

## 2026-10-02 — Tank back off the carriage

Reversed the entry below after comparing the two. Tank and pump are installed separately; a tube follows the carriage through the drag chain.

Reason: the separate tank is the easier build. It costs one well-understood problem (a ~2.5 m tube flexing in a drag chain) and avoids slosh, a changing rail load, refills every few cups and a larger carriage.

Recorded in [[Spec]] under v1e; comparison kept in [[Linear rail axis]]. Rail moving mass back to ~2.5 kg. Next: choose a tube that tolerates the flexing and size the drag chain from its bend radius.

---

## 2026-10-02 — Water tank moved onto the carriage

Tank and pump will ride on the rail carriage instead of sitting beside the machine.

Reason: *(fill in)*

Consequences recorded in [[Spec]] under v1d and worked through in [[Linear rail axis]]. Moving mass on the rail rises from ~2.5 kg to ~3.8 kg with a full 1 L tank; arm joint torques unchanged. Tube run drops from ~2.5 m to ~0.8 m. The tank sloshes near 3 Hz, so [[Slosh dynamics]] is partly relevant again. Next: choose tank capacity and whether it is rigid or a pouch.

---

## 2026-10-02 — Rail added, reach extended to 500 mm, cups scattered

Still nothing built or bought. Added a 1 m linear rail under the base so the arm can travel along the table, extended reach from 400 mm to 500 mm, and decided cups may stand anywhere in the workspace.

Reason for the longer reach: *(fill in)*

Consequences recorded in [[Spec]] under v1c. [[Torque Budget]] re-run: shoulder ~2.8 N·m static against 1.9 N·m at 400 mm — 45% more torque for 25% more reach. [[Linear rail axis]] added with first-pass drive and stiffness numbers. Scattered cups mean the arm has to find them, so the plan gains M11 (perception). Next: decide where the rail mounts on the table and how the first version will be told where the cups are.

---

## 2026-10-02 — Task changed: filling a cup, not moving one

Changed the arm's task before anything was built or bought. Old task: pick up a full cup and move it without spilling. New task: position a nozzle over a stationary cup and fill it with a pump.

Reason: *(fill in — why filling instead of moving?)*

Consequences recorded in [[Spec]] under v1b. The acceleration cap from [[Slosh dynamics]] no longer applies; that note is archived and [[Dispensing control]] replaces it. [[Torque Budget]] re-run as a first pass with an assumed 0.10 kg payload: shoulder ~1.9 N·m static against 4.4 N·m before. Learning plan updated to match. Next: measure a real cup opening and set the accuracy target; weigh the nozzle and tube.

---

## 2026-10-02 — Vault set up

Learning plan drafted. Nothing built yet. Next: [[Spec]] — specifically the accuracy target, since that decides whether closed-loop control (M8) is optional or mandatory.

---
