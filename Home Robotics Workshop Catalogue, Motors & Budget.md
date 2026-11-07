# Home Robotics Workshop: Catalogue, Motors & Budget

Nov 2, 2026 · @Isaac

## Strategy: test many robots cheaply, then go deep on one

Start with 5 small "probe" robots (about ₹1–2.5 lakh in parts) on top of a ₹2–4 lakh basic workshop, show them publicly, and let paid interest (deposits, not likes) choose which family gets the next ₹5–10 lakh. All prices here are approximate Indian market ranges in ₹ and need checking against live quotes before you buy.

Why this works: almost every robot below is built from the same 5 motor families, plus a flight or motion controller, a battery and a printed or cut frame. A shared parts stock lets you pivot between families without wasting money.

| Phase | Goal | Spend (₹) | Gate to move on |
| --- | --- | --- | --- |
| 0. Foundations | Safe workspace, bigger printer, electronics bench | 1.7–4 L | You can print, solder, tune and test safely |
| 1. Probes | Build 5 small robots, one per family | 1–2.5 L | Each probe works and is filmed |
| 2. Demand test | Videos, waitlist page, small ad spend, show to local factories and colleges | 0.2–0.5 L | Paid deposits or repeat asks from 5+ real people for one type |
| 3. Go deep | Build the winner in 2 sizes, make it repeatable | 5–10 L | First paying customer |
| 4. Heavy and exotic | Piston and jet aircraft, naval, large arms | 10 L+ | Revenue from Phase 3 pays for it |

How to read demand: ignore likes. Count people who ask for a price, ask for a custom change, or pay a deposit. Factories want arms and gantries that remove a repetitive job. Hobbyists, schools and content creators want drones and kits. Farmers, surveyors and inspectors want long-range or heavy-lift drones.

Good to know: the part of your idea that is a business is the "worker" robots (pick and place, machine loading), because the customer saves wages. Drones are the fastest to build and show, but they are crowded and regulated.

## Motor refresher: 6 types cover every robot here

Pick by what the joint must do: spin fast (propellers, wheels), hold a precise position (arm joints), or push hard slowly (lifting, axes).

| Motor | How it works | Strong at | Weak at | Used in |
| --- | --- | --- | --- | --- |
| Brushed DC | Brushes switch the coil current; speed follows voltage | Cheap, simple, tiny | Wears out, noisy, low efficiency | Toy drones, small wheeled bots, grippers |
| Brushless DC (outrunner, BLDC) | Electronic controller (ESC) switches coils; magnets spin outside | Power per gram, efficient, long life | Needs an ESC; no precise position alone | Drone props, boat and plane propellers, EDF jets, hub-motor wheels |
| Brushless with position feedback (FOC servo, gimbal BLDC) | BLDC plus encoder and field-oriented control | Smooth torque, precise, back-drivable | Costly, needs tuning | Quality arm joints, legged robots, fast cobots |
| Stepper (NEMA 17, 23, 34) | Moves in fixed steps, usually without feedback | Cheap precise positioning, holds position at rest | Loses steps if overloaded, hot, noisy, slow at speed | XY gantries, SCARA, delta arms, 3D printers, CNC |
| Servo (hobby and smart bus servos) | Motor, gearbox and position sensor in one case | Easy to drive, compact | Weak gears, low payload; bus servos (Feetech, Dynamixel) cost more | Small arms, grippers, steering, desktop robots |
| Gear motor (planetary, worm) | DC or BLDC motor plus gearbox | High torque at low speed; worm gears self-lock | Backlash, heat, weight | Wheeled robots, linear actuators, lifting axes |

Two more parts matter as much as the motor:

- Reducers: strain-wave (harmonic) and cycloidal gearboxes give arms stiffness with near-zero backlash. They are the expensive part of a good 6-axis arm.
- Drivers: steppers need a driver (TMC2209, DM542, closed-loop types); BLDC needs an ESC or FOC driver (ODrive, VESC, SimpleFOC); servos need only a signal or bus.

Sizing rules of thumb:

- Drones: total thrust at full throttle should be at least 2 times the takeoff weight (3 times for agile or payload-carrying). Lower KV with bigger props lifts more and runs more efficiently; higher KV with small props is fast and light.
- Arms: joint torque is payload weight times distance from the joint, plus the arm's own weight. Size motors for 2 times that. Stepper joints work to roughly 5 kg payload; beyond that, use FOC servos.
- Wheeled robots: pick motor torque from total weight, wheel radius and the steepest slope you need; a safe start is 24 V gear motors or hub motors.
- Test every motor and prop pair on a thrust stand before you fly or load it.

## Robot catalogue: 4 families, 28 models

IDs: A = Aerial, M = Manipulator (arms and gantries), G = Ground, N = Naval. Cost is the approximate parts cost of one prototype in ₹. Difficulty runs 1 (weekend) to 5 (team-years).

### A. Aerial

| ID | Name | Size and payload | Motors | Brain | Cost (₹) | Diff. |
| --- | --- | --- | --- | --- | --- | --- |
| A1 | Micro Whoop | Under 100 g, indoor fun | 4 brushed or 1S BLDC (0802–1103) | Integrated FC board | 4–10 k | 1 |
| A2 | Spark 5" | 250–700 g, FPV and fun | 4 BLDC 2306, 1800–2500 KV, 4S–6S | Betaflight FC and 4-in-1 ESC | 15–30 k | 2 |
| A3 | Camera Quad 7" | 1–1.5 kg, long-range video | 4 BLDC 2807, 1300 KV, 6S | FC with GPS | 30–60 k | 2 |
| A4 | Hex 12–15" | 3–6 kg, carries 1–3 kg | 6 BLDC 4014, 400–600 KV, 6S | Pixhawk class, ArduPilot | 0.8–1.5 L | 3 |
| A5 | Octo Lifter 22–30" | 10–20 kg, carries 5–10 kg with gripper | 8 BLDC 6215–8318, 100–170 KV, 12S–14S | Pixhawk or Cube, redundant power | 2.5–5 L | 4 |
| A6 | Foam Cruiser | 2 m wing, 1–2 hr flight | 1 BLDC 3548, 650 KV, 4S–6S | Pixhawk Lite or Matek | 15–40 k | 2 |
| A7 | VTOL Mapper | 2.5 m, hover plus plane | 4 lift motors plus 1 pusher | Pixhawk, ArduPilot QuadPlane | 1–3 L | 4 |
| A8 | Piston Plane | 3 m, 20–50 cc petrol engine | 1 piston engine, electric start | Autopilot plus engine telemetry | 0.8–2 L | 4 |
| A9 | EDF Jet | 90–120 mm fan, 300+ km/h | 1 BLDC ducted fan, 6S–12S | Receiver plus gyro | 0.2–0.5 L | 3 |
| A10 | Turbine Jet (later) | 1–5 kg thrust, kerosene | Micro gas turbine | ECU plus fuel system | 3–10 L | 5 |

### M. Manipulators

The "S" assembly robot you remember is the SCARA (Selective Compliance Assembly Robot Arm): rigid up and down, flexible sideways, so it presses parts into holes quickly. It is the classic assembly arm, and the right target for small-parts work.

| ID | Name | Reach and payload | Motors | Best at | Cost (₹) | Diff. |
| --- | --- | --- | --- | --- | --- | --- |
| M1 | Desk Arm 5-axis | 30 cm, 200–500 g | 5–6 smart bus servos (Feetech) | Demos, teaching, light pick and place | 15–60 k | 1 |
| M2 | Gantry XY | 0.5–1 m, 2–5 kg | 2 NEMA 23 steppers, belts or screws | Loading machines, trays, pallets | 30–80 k | 2 |
| M3 | Gantry XYZ plus Gripper | 1–2 m, 5–10 kg | 3 NEMA 23 steppers, ball screw Z | Storage racks, sorting, machine tending | 0.6–1.5 L | 3 |
| M4 | Delta Picker | 20–30 cm, up to 500 g | 3 NEMA 17 or servos, 3 parallel arms | Very fast sorting and packing | 40–80 k | 3 |
| M5 | SCARA Light | 30–50 cm, 1–3 kg | 3–4 steppers or FOC servos plus planetary reducers | Small assembly, screwing, inserting | 0.5–2 L | 4 |
| M6 | Arm 6-axis Medium | 60–90 cm, 3–10 kg | 6 FOC servos with strain-wave or cycloidal reducers | General worker, welding aid, paint, machine tending | 1.5–6 L | 5 |
| M7 | Used Industrial Arm | 1–2 m, 10–50 kg | Industrial servos (Fanuc, ABB, Yaskawa, KUKA) | Heavy loading, palletizing | 2–8 L used | 3 |
| M8 | Grippers set | Parallel, vacuum, soft, magnetic | Servo, solenoid or vacuum pump | Fits every arm above | 5–50 k | 2 |

### G. Ground

| ID | Name | Size and payload | Motors | Brain | Cost (₹) | Diff. |
| --- | --- | --- | --- | --- | --- | --- |
| G1 | Roller | 20 cm, 1 kg | 2–4 N20 or TT gear motors | ESP32 or Arduino | 2–8 k | 1 |
| G2 | Mecanum Rover | 40–60 cm, 10–30 kg | 4 gear motors 12–24 V with encoders | Raspberry Pi plus motor controllers, ROS 2 | 30–80 k | 3 |
| G3 | Tracked Rover | 60 cm, 20–50 kg, rough ground | 2 high-torque gear or BLDC motors | Pi plus Jetson, camera | 40–90 k | 3 |
| G4 | Warehouse AMR | 80 cm, 100–300 kg | 2 BLDC hub motors, 36–48 V | LiDAR, ROS 2 Nav2 | 1.5–4 L | 4 |
| G5 | Line-Follow AGV | 1 m, up to 500 kg on fixed paths | 2 gear motors, 24–48 V | Magnetic tape sensors, PLC or MCU | 0.8–2 L | 3 |
| G6 | Rover + Arm | G2 or G4 with M1, M5 or M6 on top | Combination | ROS 2 MoveIt | 0.6–5 L | 5 |
| G7 | Quadruped (later) | 30–60 cm, 5–20 kg | 12 FOC servos | Custom control | 2–8 L | 5 |

### N. Naval (later stage)

| ID | Name | Size and payload | Motors | Brain | Cost (₹) | Diff. |
| --- | --- | --- | --- | --- | --- | --- |
| N1 | RC Speedboat | 0.8–1 m | 1 water-cooled BLDC | RC receiver | 15–40 k | 2 |
| N2 | Survey Boat | 1.2–2 m, GPS missions | 2 BLDC thrusters | ArduPilot Rover or Boat | 0.6–1.5 L | 3 |
| N3 | ROV Underwater | 30 m depth, camera | 6–8 sealed thrusters | Pixhawk plus tether | 0.8–2.5 L | 4 |

Notes on the choices:

- A9 (EDF jet) teaches high-speed jet flight with a cheap electric motor; do this years before A10.
- M7 deserves a look: a used industrial arm is often cheaper and more reliable than a home-built 6-axis, and lets you sell the work (loading, palletizing) before you build your own.
- M2 and M3 gantries are the best "worker" starting point. They are easy to build, accurate, and map directly to factory jobs like loading a CNC or a press.
- G6 is the natural product mix: a rover base with an arm on top. A gantry on a rover (rails on top) is more stable than a long arm.

## Workshop equipment by phase

Phase 0 costs about ₹1.7–4 lakh (the printer is the biggest choice) and covers what difficulty 1–3 builds need. Buy Phase 1 items only when a probe needs them. Prices are approximate.

### Phase 0: Foundations

| Item | Why | Cost (₹) |
| --- | --- | --- |
| Enclosed CoreXY printer, 300–350 mm bed, hardened nozzle | Larger frames, arm links, ASA, PETG, nylon-carbon parts | 0.6–1.5 L |
| Filament: PETG, ASA, PA-CF, TPU | Structural parts, drone frames, soft grippers | 8–15 k |
| Soldering station, hot air, fume extractor | Wiring, ESCs, connectors | 8–15 k |
| Bench power supply (30 V, 10 A) | Test motors and boards safely | 6–12 k |
| Multimeter, calipers, hex and torx sets, crimpers | Daily work | 8–15 k |
| Smart LiPo charger (300–1000 W), LiPo bags, metal fireproof box | Safe battery charging and storage | 10–25 k |
| Entry oscilloscope (100 MHz) and logic analyzer | Debug motor drivers and buses | 15–40 k |
| Thrust stand (load cell, power meter) | Pick motor and prop pairs | 15–60 k |
| Dev boards: Raspberry Pi 5, ESP32, STM32, Arduino | Brains for every robot | 10–20 k |
| Safety: ABC and CO2 extinguishers, glasses, respirator, e-stops, smoke alarm | Fire, dust and moving-arm protection | 8–15 k |
| Benches, shelves, bins, 15 A power line, UPS | A place where nothing falls apart | 20–50 k |

### Phase 1: Probe builds

| Item | Why | Cost (₹) |
| --- | --- | --- |
| Desktop CNC router (3018 upgraded or 6040) | Cut carbon-fibre sheet, aluminium and acrylic for frames | 0.3–1 L |
| Diode laser (20–40 W) | Fast prototypes in acrylic, wood, cardboard | 30–80 k |
| Drill press, band saw, bench vice | General fabrication | 20–50 k |
| Flight gear: radio (EdgeTX), goggles, props, spare parts | Drone testing | 25–60 k |
| Safety net or test cage for drones | Keep first flights contained | 10–30 k |
| Linear rails, ball screws, belts, bearings stock | Gantries and arms | 20–50 k |
| Camera, lights, tripod | Demo videos, which are your marketing | 20–40 k |

### Phase 2 and beyond

| Item | Why | Cost (₹) |
| --- | --- | --- |
| Benchtop mill and lathe | Custom metal parts, shafts, mounts | 0.8–2.5 L |
| Large format printer (500 mm and up) | Wings, boat hulls, rover bodies | 1.5–5 L |
| Vacuum bagging kit (composites) | Strong light airframes, boat hulls | 0.5–1.5 L |
| MIG and TIG welder | Steel rover chassis, large frames | 0.5–1.5 L |
| Engine test cell, fuel storage (piston and turbine) | Only when A8 or A10 starts | 1–3 L |
| Water tank (for N-series tests) | Boat and ROV work | 0.2–0.6 L |

Space and power: aim for 150–300 sq ft with good ventilation, a separate circuit for printers and chargers, and a hard floor. Keep battery charging away from wood and paper.

Software (all free or free for personal use): Fusion 360 or FreeCAD for design, KiCad for circuit boards, ArduPilot with Mission Planner or QGroundControl for drones, ROS 2 with MoveIt and Nav2 for arms and rovers, PlatformIO for microcontrollers.

Buy-or-outsource rule: until you know which family wins, order machined or sheet-metal parts from online CNC and fabrication services instead of buying a mill or lathe.

## Recommended start, total cost and rules

Build these 5 probes first: A1 Micro Whoop, A2 Spark 5", M1 Desk Arm, M2 Gantry XY and G2 Mecanum Rover. Together they use 4 of the 6 motor types, cost about ₹1–2.5 lakh in parts, and cover fun drones, a worker arm, a worker gantry and a mobile base. When M1 works, mount it on G2 to test the mixed rover-and-arm idea (G6) at almost no extra cost.

| Item | Parts (₹) |
| --- | --- |
| A1 Micro Whoop | 4–10 k |
| A2 Spark 5" | 15–30 k |
| M1 Desk Arm | 15–60 k |
| M2 Gantry XY | 30–80 k |
| G2 Mecanum Rover | 30–80 k |
| Probes total | 1–2.6 L |
| Phase 0 workshop | 1.7–4.2 L |
| Phase 1 tools (optional at first) | 1.5–4 L |
| Everything above | 4–11 L |

Lean path: spend about ₹3–4 lakh by buying Phase 0 only, skipping the CNC router and laser, and ordering cut parts from outside. Add tools when a specific probe is blocked without them.

India rules to check before flying (approximate, from memory; confirm on the DGCA and Digital Sky sites):

- Drone Rules 2021: very small drones (up to 250 g) for non-commercial use are lightly regulated; heavier ones need registration and must respect green, yellow and red airspace zones.
- Selling or flying commercially adds certification and licence steps, and complete drones are barred from import, though components are not.
- Radio gear must use permitted frequencies; LiPo batteries are restricted on flights; petrol and kerosene need safe storage. A10 turbines and A8 engines will draw extra scrutiny.

Next steps:

1. Tell me your budget, city, which 3D printer you own, and who you could show demos to.
2. I turn this into a sourcing list with live Indian prices and links for the Phase 0 items.
3. Pick the first two probes and I write a build plan, parts list and wiring for each.
