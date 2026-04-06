# HARA — Electronic Stability Control (ESC)

**Standard:** ISO 26262-3  
**Date:** 2026-04-06  
**Status:** Draft  
**Version:** 1.0  

---

## 1. Item Definition

**Item:** Electronic Stability Control (ESC)

**Functions:**
1. Detect vehicle loss of lateral stability (yaw rate / sideslip deviation)
2. Apply selective individual wheel braking to restore trajectory
3. Reduce engine torque to restore stability
4. Continuously monitor wheel speeds, steering angle, yaw rate, and lateral acceleration

**Item Boundary:**
- In scope: ESC ECU software and hardware, wheel speed sensors, yaw rate sensor, steering angle sensor, lateral accelerometer, hydraulic pressure actuators, powertrain torque interface
- Out of scope: ABS standalone function (though interaction is analysed), base brake hardware

---

## 2. Operational Situations

| ID    | Situation                    | Speed (km/h) | Road / Environment       |
|-------|------------------------------|--------------|--------------------------|
| OS-01 | Highway lane change          | 100–130      | Dry or wet               |
| OS-02 | Urban intersection turn      | 20–50        | Mixed, pedestrians       |
| OS-03 | Rural curve                  | 60–100       | Wet or icy               |
| OS-04 | Emergency evasive maneuver   | 60–130       | Any                      |
| OS-05 | Low-speed parking maneuver   | 0–10         | Dry                      |
| OS-06 | Motorway straight driving    | 100–150      | Dry, moderate traffic    |

---

## 3. Rating Scales

### Severity (S)
| Class | Description |
|-------|-------------|
| S0 | No injuries |
| S1 | Light to moderate injuries |
| S2 | Severe and life-threatening injuries (survival probable) |
| S3 | Life-threatening injuries (survival uncertain) or fatal |

### Exposure (E)
| Class | Description | Approximate probability |
|-------|-------------|------------------------|
| E0 | Incredibly unlikely | < 1% of operating time |
| E1 | Very low probability | ~1–2% |
| E2 | Low probability | ~2–10% |
| E3 | Medium probability | ~10–50% |
| E4 | High probability | > 50% |

### Controllability (C)
| Class | Description |
|-------|-------------|
| C0 | Controllable in general |
| C1 | Simply controllable (> 99% of drivers) |
| C2 | Normally controllable (> 90% of drivers) |
| C3 | Difficult to control or uncontrollable (< 90% of drivers) |

### ASIL Determination Matrix (S3 rows shown; full matrix per ISO 26262-3 Table 4)
| S  | E  | C1 | C2 | C3 |
|----|----|----|----|----|
| S3 | E2 | A  | B  | C  |
| S3 | E3 | B  | C  | D  |
| S3 | E4 | C  | D  | D  |
| S2 | E4 | A  | B  | C  |
| S2 | E3 | QM | A  | B  |

---

## 4. Hazardous Events

### H-001 — Unintended single-wheel braking on dry highway

| Field | Value |
|-------|-------|
| Malfunction | ESC spontaneously applies full brake pressure to one rear wheel |
| Operational situation | OS-01, OS-06 — highway at 100+ km/h |
| Hazardous event | Severe yaw moment → vehicle spins → rollover or oncoming-lane intrusion |
| Severity | **S3** — life-threatening / fatal |
| Exposure | **E3** — ESC active during highway driving (~10–50% of time) |
| Controllability | **C3** — driver cannot recover a high-speed spin |
| **ASIL** | **D** |

---

### H-002 — Asymmetric wheel braking during hard straight-line stop

| Field | Value |
|-------|-------|
| Malfunction | ESC misinterprets hard braking as instability; applies asymmetric wheel braking |
| Operational situation | OS-01, OS-06 — emergency braking at highway speed |
| Hazardous event | Vehicle pulls sharply sideways during ABS stop → lane departure or oncoming collision |
| Severity | **S3** |
| Exposure | **E3** |
| Controllability | **C3** — driver is already at limits of control during hard braking |
| **ASIL** | **D** |

---

### H-003 — Failure to activate during oversteer/understeer on icy curve

| Field | Value |
|-------|-------|
| Malfunction | ESC does not detect or respond to yaw rate deviation |
| Operational situation | OS-03 — rural curve, icy road |
| Hazardous event | Vehicle continues off-road trajectory → barrier or ditch impact |
| Severity | **S3** |
| Exposure | **E2** — icy road conditions (~2–10% of annual operating time) |
| Controllability | **C3** — loss of control on ice is largely unrecoverable |
| **ASIL** | **C** |

---

### H-004 — Unintended engine torque reduction at motorway cruise

| Field | Value |
|-------|-------|
| Malfunction | ESC commands maximum torque reduction without confirmed instability event |
| Operational situation | OS-06 — motorway at 120 km/h, moderate following traffic |
| Hazardous event | Sudden unexpected deceleration → rear-end collision by following vehicle |
| Severity | **S2** — severe injury |
| Exposure | **E4** — motorway cruising is a frequent driving scenario |
| Controllability | **C2** — driver partially aware; following driver has reduced reaction time |
| **ASIL** | **B** |

---

### H-005 — Continuous unintended ESC interventions in urban driving

| Field | Value |
|-------|-------|
| Malfunction | ESC continuously applies micro-braking or torque cuts during normal cornering |
| Operational situation | OS-02 — urban intersection, low-speed turning |
| Hazardous event | Unexpected deceleration/pull into pedestrian crossing or cyclist lane |
| Severity | **S2** |
| Exposure | **E4** — urban driving is high-frequency |
| Controllability | **C2** |
| **ASIL** | **B** |

---

### H-006 — Incorrect wheel selection during emergency evasive maneuver

| Field | Value |
|-------|-------|
| Malfunction | ESC applies braking to wrong axle/wheel, amplifying yaw instead of correcting it |
| Operational situation | OS-04 — evasive maneuver at 80 km/h |
| Hazardous event | Yaw overshoot → vehicle spins off road |
| Severity | **S3** |
| Exposure | **E2** — emergency evasive maneuvers are infrequent |
| Controllability | **C3** |
| **ASIL** | **C** |

---

### H-007 — ESC inadvertently disables ABS during combined braking/steering

| Field | Value |
|-------|-------|
| Malfunction | ESC state machine conflict suspends ABS authority during combined maneuver |
| Operational situation | OS-04 — emergency brake + steer |
| Hazardous event | Wheel lock-up → loss of steering authority → inability to avoid obstacle |
| Severity | **S3** |
| Exposure | **E2** |
| Controllability | **C2** — driver may partially compensate with steering |
| **ASIL** | **B** |

---

## 5. ASIL Summary

| ID    | Hazardous Event                                              | S  | E  | C  | ASIL |
|-------|--------------------------------------------------------------|----|----|----|------|
| H-001 | Unintended single-wheel braking on highway                   | S3 | E3 | C3 | **D** |
| H-002 | Asymmetric braking during hard stop on highway               | S3 | E3 | C3 | **D** |
| H-003 | Failure to activate on icy curve                             | S3 | E2 | C3 | **C** |
| H-004 | Unintended torque reduction at motorway cruise               | S2 | E4 | C2 | **B** |
| H-005 | Continuous unintended interventions in urban driving         | S2 | E4 | C2 | **B** |
| H-006 | Wrong wheel braking during evasive maneuver                  | S3 | E2 | C3 | **C** |
| H-007 | ESC disables ABS during combined braking/steering maneuver   | S3 | E2 | C2 | **B** |

---

## 6. Safety Goals

| SG ID  | Safety Goal                                                                                                                 | ASIL  | Safe State                                              | FTTI   |
|--------|-----------------------------------------------------------------------------------------------------------------------------|-------|---------------------------------------------------------|--------|
| SG-001 | ESC shall not apply unintended individual wheel brake pressure exceeding [TBD bar] during normal driving conditions         | **D** | Remove all ESC braking commands; deactivate ESC function | 50 ms  |
| SG-002 | ESC shall detect vehicle instability and initiate corrective braking within [TBD ms] across all operating conditions        | **C** | Disable ESC; illuminate ESC warning lamp to driver       | 200 ms |
| SG-003 | ESC shall not command engine torque reduction greater than [TBD %] without a confirmed instability event                    | **B** | Restore full torque demand; deactivate ESC              | 100 ms |
| SG-004 | ESC shall not reduce ABS control authority during a combined braking and steering maneuver                                  | **B** | ABS retains priority; ESC suspends all interventions    | 20 ms  |

> **Note:** Values marked [TBD] shall be determined during system-level functional safety requirements definition (ISO 26262-4) and confirmed via vehicle dynamics simulation and test.

---

## 7. Open Issues

| ID    | Issue                                                                 | Owner | Target Date |
|-------|-----------------------------------------------------------------------|-------|-------------|
| OI-01 | FTTI values for SG-001 and SG-002 require vehicle dynamics validation | TBD   | TBD         |
| OI-02 | Threshold values in SG-001, SG-003 to be derived from FSR phase       | TBD   | TBD         |
| OI-03 | Exposure for OS-03 (icy road) to be confirmed with fleet data         | TBD   | TBD         |

---

## 8. Next Steps

1. **FSR derivation** — Derive Functional Safety Requirements from each Safety Goal (ISO 26262-4)
2. **ASIL decomposition** — SG-001 (ASIL D) candidate for decomposition: software monitor ASIL B(D) + hardware watchdog ASIL B(D)
3. **HSI definition** — Define sensor accuracy requirements (yaw rate, wheel speed, steering angle) that SW safety requirements depend on
4. **FMEA linkage** — Each safety goal maps to a top-level failure mode in the ESC software DFMEA
5. **FTA** — Construct fault trees for SG-001 and SG-002 to size diagnostic coverage targets

---

## Revision History

| Version | Date       | Author | Changes       |
|---------|------------|--------|---------------|
| 1.0     | 2026-04-06 | —      | Initial draft |
