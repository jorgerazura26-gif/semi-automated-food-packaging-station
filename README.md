# semi-automated-food-packaging-station
Semi-automated vertical packaging station for small-scale food producers. Integrates bag handling, gravimetric dosing with PD control, Venturi-based vacuum extraction, and thermal sealing. Distributed control via 6 ESP32 microcontrollers over ESP-NOW. Built in 8 weeks for Fungívora.

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Architecture](#2-system-architecture)
3. [Bag Handling](#3-bag-handling)
4. [Gravimetric Dosing & Weighing](#4-gravimetric-dosing--weighing)
5. [Vacuum Extraction & Sealing](#5-vacuum-extraction--sealing)
6. [Distributed Control](#6-distributed-control)
7. [Results](#7-results)
8. [Team](#8-team)

---

## 1. Introduction

This project was developed in collaboration with Fungívora, a small Mexican startup that produces chorizeta, a mushroom-soy chorizo analog. Their packaging process was entirely manual, limiting production to around 25 bags per hour. The goal was to design and implement a semi-automated station capable of weighing and packaging the product with minimal manual intervention, reducing inconsistencies and increasing throughput.

The proposed system integrates mechanical, pneumatic, and electronic components across eight mechatronic subsystems, all coordinated through a distributed wireless control architecture.

---

## 2. System Architecture

The station is a vertical, frame-mounted assembly built on Bosch aluminum profiles. All subsystems share a common 12 VDC / 40 A bus for actuators and a 24 VDC / 10 A rail for the NEMA 23 stepper and pneumatic solenoid valves.

The control architecture consists of six ESP32 microcontrollers, each assigned a dedicated role in the packaging sequence, communicating wirelessly via ESP-NOW. The full cycle covers eight stages: bag loading, bag placement, bag opening, dosing and weighing, station transfer, in-bag vacuum extraction, vacuum pressing, and thermal sealing.

---

## 3. Bag Handling

Bags are transported into the station by a NEMA 17-driven belt and V-slot rail carriage. A gear-linkage gripper actuated by MG995 servomotors holds and positions each bag throughout the cycle. Bag opening is achieved by four silicone suction cups pressed against both sides of the bag by pneumatic pistons, evacuated by a 12 VDC vacuum pump. An XGZP6847A pressure sensor confirms the bag is fully open before dosing begins.

---

## 4. Gravimetric Dosing & Weighing

A NEMA 23 stepper drives an auger screw that dispenses chorizeta into a weighing chamber isolated by a servo-actuated gate. A 5 kg load cell with an HX711 24-bit ADC streams real-time weight data at 80 SPS.

Dispensing follows a two-phase PD control strategy:

- **Bulk phase:** the auger runs at full speed while the measured weight is below 90% of the target.
- **Fine phase:** a PD controller (Kp = 6, Kd = 20) progressively decelerates the auger as the target is approached, preventing overshoot. Since dispensed material cannot be recovered, the response is deliberately over-damped.

The floor gate opens only after dual confirmation: weight target met and bag-open signal received from the suction cup node.

---

## 5. Vacuum Extraction & Sealing

After filling, a Venturi vacuum nozzle is inserted into the bag mouth and pneumatic press bars clamp around it. Compressed air drives the Venturi effect, extracting air from the bag without the need for a second vacuum pump. The system reaches in-bag pressures of approximately −50 kPa, with an acceptance threshold set at −40 kPa.

Thermal sealing uses two nichrome resistance elements connected in series (2 Ω total, 72 W at 12 V), producing a double weld line across the bag mouth. The sealing cycle runs for 24 seconds, followed by a 2-second cooling period under pneumatic clamp pressure.

---

## 6. Distributed Control

Six ESP32 microcontrollers communicate via ESP-NOW, a connectionless peer-to-peer protocol with sub-millisecond latency that requires no router or access point. Each node has a dedicated role:

| Node | Role |
|---|---|
| Button_esp | User interface: START, RESET, EMERGENCY commands |
| Nema_esp | Main orchestrator: coordinates the full packaging sequence |
| Gripper_esp | Controls grippers, linear actuators, and bag detection |
| Weighing_esp | Controls dosing unit: load cell, auger, PD control, gate |
| Suction_esp | Controls vacuum pump and monitors bag-open pressure |
| Mosfet_esp | Controls 8 MOSFETs for solenoid valves, vacuum clamp, sealing clamp, and thermal sealer |

Each stage is initiated only after the previous one reports completion, ensuring the bag is correctly placed, opened, filled, evacuated, and sealed before release.

---

## 7. Results

Performance was evaluated over 10 bags across three criteria: cycle time, vacuum sealing success, and dosing accuracy.

| Metric | Result |
|---|---|
| Average cycle time | 96.7 s per bag |
| Production throughput | ~45 bags/hour (80% improvement over 25 bags/hour manually) |
| Mean dosing weight | 100.9 g (target: 100 g) |
| Dosing standard deviation | ±3.3 g |
| Bags within ±5 g tolerance | 9 out of 10 |
| Vacuum level achieved | ~−50 kPa |
| Overall acceptance rate | 60% (primary rejection cause: vacuum sealing) |

---

## 8. Team

Developed at ITESM Campus Querétaro:

- Jorge Julián Razura Piña
- Emiliano González González
- Joshua Arturo Ayala Argudín
- Joshua Joseph Foord Galdoz
