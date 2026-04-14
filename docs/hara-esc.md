# HARA — Electronic Stability Control (ESC)

**Standard:** ISO 26262-3  
**Date:** 2026-04-08  
**Status:** Draft  
**Version:** 1.1  

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

> **Note:** Values marked [TBD] shall be determined during system-level functional safety requirements definition (ISO 26262-4) and confirmed via vehicle dynamics simulation and test.

---

### SG-001 — Unintended Individual Wheel Braking

**Safety Goal:** ESC shall not apply unintended individual wheel brake pressure exceeding [TBD bar] during normal driving conditions.

**Item:** Electronic Stability Control (ESC) — see Section 1 item boundary.

**Hazardous Events Addressed:**

| Hazard | Event | S  | E  | C  | ASIL |
|--------|-------|----|----|----|------|
| H-001  | Unintended single-wheel braking on dry highway — severe yaw moment → spin → rollover | S3 | E3 | C3 | D |
| H-002  | Asymmetric braking during hard straight-line stop — vehicle pulls sideways during ABS stop | S3 | E3 | C3 | D |

**ASIL:** **D** (worst-case hazard determines goal ASIL)

**ASIL Derivation:**

| Parameter | Rating | Rationale |
|-----------|--------|-----------|
| Severity | S3 | Rollover or oncoming-lane intrusion at highway speed is potentially fatal |
| Exposure | E3 | Highway/motorway driving occurs 10–50% of operating time |
| Controllability | C3 | Driver cannot recover a high-speed spin; reaction time insufficient |

**FTTI:** 50 ms
> Budget rationale: vehicle dynamics modelling indicates yaw divergence becomes unrecoverable within 80–100 ms at 100 km/h; 50 ms FTTI leaves 30–50 ms margin for driver/system reaction. Value subject to validation per OI-01.

**Safe State:** ESC braking function deactivated; base brake and ABS authority fully restored to driver.

- All ESC hydraulic pressure modulation commands removed immediately
- ESC ECU transitions to passive monitoring mode (no actuation)
- ABS and base brake system retain full authority unaffected
- ESC warning lamp illuminated on instrument cluster
- DTC stored with freeze-frame data (vehicle speed, yaw rate, wheel pressures at fault onset)
- ESC function remains deactivated until driver cycle or service reset

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| No unintended actuation under normal driving stimuli | SIL / HIL regression | Zero spurious brake commands across 10 000 test cycles |
| FTTI ≤ 50 ms from fault onset to safe state | HIL fault injection | Fault-to-safe-state time < 50 ms in all injected scenarios |
| Safe state persists until intentional reset | HIL state verification | ESC does not self-re-enable after safe state entry |
| Warning lamp activates within 500 ms of safe state entry | HIL / vehicle test | Lamp state confirmed via CAN signal and physical check |
| DTC stored with correct freeze-frame | Diagnostic read-back | DTC present and freeze-frame data plausible after fault |

---

### SG-002 — Failure to Detect and Correct Vehicle Instability

**Safety Goal:** ESC shall detect vehicle instability and initiate corrective braking within [TBD ms] across all operating conditions.

**Item:** Electronic Stability Control (ESC) — see Section 1 item boundary.

**Hazardous Events Addressed:**

| Hazard | Event | S  | E  | C  | ASIL |
|--------|-------|----|----|----|------|
| H-003  | Failure to activate during oversteer/understeer on icy curve — vehicle continues off-road | S3 | E2 | C3 | C |
| H-006  | Incorrect wheel selection during emergency evasive maneuver — yaw overshoot, vehicle spins | S3 | E2 | C3 | C |

**ASIL:** **C** (worst-case hazard determines goal ASIL)

**ASIL Derivation:**

| Parameter | Rating | Rationale |
|-----------|--------|-----------|
| Severity | S3 | Off-road excursion into barrier or ditch at speed is life-threatening |
| Exposure | E2 | Icy road or emergency evasive conditions occur 2–10% of operating time |
| Controllability | C3 | Loss of control on ice or during yaw overshoot is largely unrecoverable |

**FTTI:** 200 ms
> Budget rationale: yaw rate divergence on low-friction surfaces develops over approximately 300–500 ms; 200 ms FTTI provides approximately 100–300 ms for corrective intervention to take effect. Value subject to validation per OI-01.

**Safe State:** ESC deactivated with driver notification; driver retains full manual control with no hidden system interference.

- ESC actuation suspended; no corrective braking or torque commands issued
- ESC warning lamp illuminated to inform driver of degraded stability assist
- ABS and base braking retained at full authority
- DTC stored with freeze-frame data (yaw rate deviation, wheel speeds, road speed)
- System remains in degraded mode until driver cycle or service reset

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Instability detection within FTTI across all OS conditions | HIL with plant model (ice, wet, dry) | Detection confirmed before FTTI in 100% of test scenarios |
| Correct wheel selected for corrective braking | SIL back-to-back vs. reference model | Wheel selection matches reference in ≥ 99.9% of cases |
| No missed activation in oversteer / understeer scenarios | HIL fault injection | Zero missed activations in standardised test matrix |
| Safe state reached and maintained on detection fault | HIL fault injection | Safe state entered within FTTI; no spurious re-activation |

---

### SG-003 — Unintended Engine Torque Reduction

**Safety Goal:** ESC shall not command engine torque reduction greater than [TBD %] without a confirmed instability event.

**Item:** Electronic Stability Control (ESC) — see Section 1 item boundary.

**Hazardous Events Addressed:**

| Hazard | Event | S  | E  | C  | ASIL |
|--------|-------|----|----|----|------|
| H-004  | Unintended engine torque reduction at motorway cruise — sudden deceleration → rear-end collision | S2 | E4 | C2 | B |
| H-005  | Continuous unintended ESC interventions in urban driving — unexpected deceleration near pedestrians | S2 | E4 | C2 | B |

**ASIL:** **B** (worst-case hazard determines goal ASIL)

**ASIL Derivation:**

| Parameter | Rating | Rationale |
|-----------|--------|-----------|
| Severity | S2 | Rear-end collision or pedestrian contact at low-to-medium speed causes severe injury; survival probable |
| Exposure | E4 | Motorway cruising and urban driving collectively represent > 50% of operating time |
| Controllability | C2 | Driver is partially aware and can react; following driver has reduced reaction time |

**FTTI:** 100 ms
> Budget rationale: unexpected deceleration at motorway speed (120 km/h) presents rear-collision risk within approximately 200–400 ms depending on following distance and driver reaction; 100 ms FTTI allows sufficient intervention margin.

**Safe State:** Full powertrain torque authority restored; ESC torque reduction interface deactivated.

- ESC torque reduction request to powertrain set to zero / withdrawn
- Powertrain ECU resumes normal torque management without ESC override
- ESC torque-reduction channel flagged as unavailable; function disabled
- ESC warning lamp illuminated
- DTC stored with torque demand and vehicle speed at fault onset

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| No torque reduction commanded absent instability confirmation | SIL regression | Zero unconfirmed torque reduction requests across 5 000 test cycles |
| Torque restored within FTTI on spurious command detection | HIL fault injection | Torque restoration latency < 100 ms in all injected scenarios |
| Torque reduction magnitude does not exceed [TBD %] threshold | SIL / HIL signal monitoring | All commanded reductions ≤ defined threshold |
| Continued operation without ESC torque channel does not affect base drivability | Vehicle test | No drivability defects noted in structured drive evaluation |

---

### SG-004 — ESC Suppression of ABS Authority

**Safety Goal:** ESC shall not reduce ABS control authority during a combined braking and steering maneuver.

**Item:** Electronic Stability Control (ESC) — see Section 1 item boundary.

**Hazardous Events Addressed:**

| Hazard | Event | S  | E  | C  | ASIL |
|--------|-------|----|----|----|------|
| H-007  | ESC state machine conflict suspends ABS authority — wheel lock-up → loss of steering during emergency brake + steer | S3 | E2 | C2 | B |

**ASIL:** **B**

**ASIL Derivation:**

| Parameter | Rating | Rationale |
|-----------|--------|-----------|
| Severity | S3 | Wheel lock-up during combined braking and steering results in loss of directional control; potentially fatal |
| Exposure | E2 | Emergency combined brake-and-steer events occur 2–10% of operating time |
| Controllability | C2 | Driver may partially compensate with steering corrections; not fully uncontrollable |

**FTTI:** 20 ms
> Budget rationale: ABS cycle time is typically 10–15 ms; allowing one ABS cycle of impaired authority before correction keeps the window within a single control period. Loss of ABS authority for more than one cycle risks wheel lock-up and steering loss.

**Safe State:** ABS retains unconditional priority; all ESC hydraulic interventions suspended.

- ESC wheel-pressure modulation commands immediately inhibited
- ABS control loop regains sole authority over wheel cylinder pressures
- ESC state machine reset to idle; no further interventions until next ignition cycle
- ESC warning lamp illuminated
- DTC stored with ESC state and ABS arbitration status at fault onset

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| ABS authority not interrupted during any combined brake + steer input | HIL with combined pedal + steering stimuli | ABS wheel-slip control active and uninterrupted across all combined maneuver scenarios |
| Safe state reached within FTTI = 20 ms | HIL fault injection (state machine conflict injection) | ABS priority restored < 20 ms from conflict detection in all scenarios |
| ESC does not re-engage during ABS active cycle | HIL signal monitoring | Zero ESC hydraulic commands observed while ABS active flag is set |
| No degradation to ABS stopping distance | HIL performance test | Stopping distance within ± 5% of ABS-only baseline |

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
| 1.1     | 2026-04-08 | —      | Section 6 Safety Goals expanded to structured per-goal format per ISO 26262-3 skill template: added item reference, hazardous event traceability, ASIL derivation table (S/E/C), FTTI budget rationale, detailed safe state description, and verification criteria for SG-001 through SG-004 |
