# Functional Safety Requirements — Electronic Stability Control (ESC)

**Standard:** ISO 26262-4  
**Date:** 2026-04-08  
**Status:** Draft  
**Version:** 1.2  
**Parent document:** [HARA — ESC](hara-esc.md)

---

## Derivation Approach

Each Functional Safety Requirement (FSR) is derived from a Safety Goal in the HARA.
FSRs specify *what* the system must do at the functional level to achieve each safety goal —
without prescribing implementation. ASIL is inherited from the parent safety goal.

Every FSR addresses at minimum:
- **Prevention** — constraining normal system behaviour to avoid the hazard
- **Detection** — monitoring for the fault condition
- **Reaction** — the action taken when the fault is confirmed

Each FSR includes a **Category** field (Prevention / Detection / Reaction), **Allocation** (SW / HW),
and a structured **Verification Criteria** table (criterion / method / pass condition) for
explicit traceability. Final allocation is confirmed during system architectural design (ISO 26262-4 §7).

---

## FSRs from SG-001 (ASIL D)

**Safety Goal:** ESC shall not apply unintended individual wheel brake pressure exceeding [TBD bar] during normal driving conditions.

**Item:** Electronic Stability Control (ESC) — see [HARA §1 item boundary](hara-esc.md).

**Hazardous Events Addressed:** H-001 (unintended single-wheel braking on highway), H-002 (asymmetric braking during hard straight-line stop) — see [HARA §4](hara-esc.md).

**ASIL:** **D** (worst-case hazard determines goal ASIL)

**ASIL Derivation:**

| Parameter | Rating | Rationale |
|-----------|--------|-----------|
| Severity | S3 | Rollover or oncoming-lane intrusion at highway speed is potentially fatal |
| Exposure | E3 | Highway/motorway driving occurs 10–50% of operating time |
| Controllability | C3 | Driver cannot recover a high-speed spin; reaction time insufficient |

**FTTI:** 50 ms  
> Budget rationale: vehicle dynamics modelling indicates yaw divergence becomes unrecoverable within 80–100 ms at 100 km/h; 50 ms FTTI leaves 30–50 ms margin for driver/system reaction. Subject to validation per OI-01.

**Safe State:** ESC braking function deactivated; base brake and ABS authority fully restored to driver.

- All ESC hydraulic pressure modulation commands removed immediately
- ESC ECU transitions to passive monitoring mode (no actuation)
- ABS and base brake system retain full authority unaffected
- ESC warning lamp illuminated on instrument cluster
- DTC stored with freeze-frame data (vehicle speed, yaw rate, wheel pressures at fault onset)
- ESC function remains deactivated until driver cycle or service reset

---

### FSR-001-01 — Wheel brake pressure plausibility monitoring

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW + HW  
**Category:** Prevention + Detection

**Requirement:**

The ESC system shall continuously compare the commanded brake pressure for each individual wheel against a plausibility envelope. The plausibility envelope shall be derived from driver brake demand, vehicle speed, lateral acceleration, yaw rate, and the current operating mode (including the hard-braking inhibit mode defined in FSR-001-09). A commanded pressure that deviates from the plausibility envelope by more than [TBD bar] for longer than [TBD ms] shall be classified as an unintended intervention and shall trigger the fault reaction defined in FSR-001-03.

> **Note:** Threshold values [TBD bar] and [TBD ms] shall be determined from vehicle dynamics simulation and confirmed via HIL test (see OI-FSR-001-01a, OI-FSR-001-01b).

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Plausibility envelope computed correctly for all input combinations | Unit test — equivalence class + boundary value | All test vectors produce envelope output matching reference |
| Commanded pressure above threshold correctly classified as unintended | SIL back-to-back vs reference model | Classification matches reference in all test scenarios |
| Fault reaction triggered within detection window on threshold breach | HIL fault injection per wheel channel | Fault reaction activated within [TBD ms] window in all injected channels |
| No false positives during normal driving manoeuvres | HIL regression — normal driving stimulus library | Zero spurious unintended-pressure classifications across 10 000 cycles |

---

### FSR-001-02 — Independent monitoring channel for brake command path

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**ASIL Decomposition:** ASIL D satisfied by B(D) + B(D) per ISO 26262-9 §5 (main control path + independent monitor)  
**Allocation:** SW (monitor path) + HW (separate monitor channel)  
**Category:** Detection

**Requirement:**

The ESC shall implement a monitoring function that is independent of the main brake command calculation path. The monitoring function shall observe the output brake pressure commands and shall suppress transmission of any command classified as unintended per FSR-001-01 within the time budget allocated from the 50 ms FTTI. The following independence requirements apply to enable the ASIL B(D) + ASIL B(D) decomposition: (a) the main control path shall be developed to ASIL B(D); (b) the monitoring path shall be developed to ASIL B(D); (c) the two paths shall not share source code, calibration data sets, or hardware execution context (processor core, memory region, clock domain); (d) a Dependent Failure Analysis (DFA) per ISO 26262-9 §7 shall confirm freedom from common-cause failures between the two paths; (e) fault injection testing shall verify that a fault in either path independently causes the safe state to be entered.

> **Open issue OI-02:** ASIL D decomposition confirmation and DFA completion required before system architecture baseline is frozen.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Main path and monitor path are architecturally independent | Architecture review + code independence audit | No shared source code, calibration data, or hardware execution context confirmed |
| Fault in main path alone causes safe state | HIL fault injection — main path only | Safe state entered; monitor path integrity confirmed unaffected |
| Fault in monitor path alone causes safe state | HIL fault injection — monitor path only | Safe state entered; main path integrity confirmed unaffected |
| No common-cause failures between paths | DFA review per ISO 26262-9 §7 | All identified CCFs mitigated; path independence confirmed |

---

### FSR-001-03 — Deactivation within FTTI on unintended pressure detection

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW + HW  
**Category:** Reaction

**Requirement:**

Upon detection of an unintended wheel brake pressure command, the ESC shall: (a) set all wheel brake pressure commands to zero within the SW reaction time budget (SW budget = 50 ms minus the hydraulic release time specified in the HSI — see OI-04); (b) transition to the deactivated state (see FSR-001-07 for persistence requirements); (c) the hydraulic actuator shall physically release wheel brake pressure within the hydraulic release budget specified in the HSI, such that the combined SW reaction time plus hydraulic release time does not exceed 50 ms from the moment of fault detection. The 50 ms budget applies to the complete physical chain and is non-negotiable.

> **Open issue OI-04:** HSI specification for hydraulic valve release time required from HW team. Until confirmed, the SW reaction time budget cannot be finalised.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| End-to-end reaction time from fault detection to physical pressure release ≤ 50 ms | HIL end-to-end timing — fault injection + pressure transducer on hydraulic line | Measured fault-to-release time < 50 ms in all injected fault scenarios |
| SW reaction time within allocated budget (50 ms − hydraulic release time) | WCET analysis | Measured SW WCET ≤ (50 ms − HSI-confirmed hydraulic release time) |
| Hydraulic pressure physically released within HSI budget | HIL hydraulic bench test | Pressure transducer confirms pressure below release threshold within HSI-specified budget |
| Deactivated state entered immediately following command suppression | HIL state monitoring | ESC state = deactivated confirmed before end of FTTI window |

---

### FSR-001-04 — Power-on self-test of brake command path

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW  
**Category:** Prevention

**Requirement:**

At every ECU power-on, the ESC shall execute a self-test of the brake pressure command path before enabling ESC function. The self-test shall verify sensor read-back, actuator enable logic, and monitoring channel operation. ESC shall not activate if any self-test step fails.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Self-test executes at every power-on before ESC enable | HIL power cycle test | Self-test sequence confirmed logged; all steps completed before ESC enable flag set |
| ESC does not activate if any self-test step fails | HIL — inject failure at each self-test step | ESC remains disabled; DTC stored; warning lamp active for each injected step failure |
| Sensor read-back validated during self-test | Unit test with mock sensor fault | Implausible sensor read-back classified as self-test failure; ESC blocked |
| Monitoring channel validated during self-test | Unit test + HIL | Monitor channel fault detected during self-test; ESC blocked |

---

### FSR-001-05 — ESC warning lamp activation on deactivation

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW  
**Category:** Reaction

**Requirement:**

When ESC transitions to the deactivated state due to a detected fault, the ESC warning lamp shall be illuminated within the 50 ms FTTI window (i.e., lamp activation shall be part of the fault reaction sequence that completes within 50 ms of fault detection). A corresponding DTC shall be stored in non-volatile memory within one diagnostic task cycle of fault detection. The lamp state shall be maintained for the duration of the deactivated state as defined in FSR-001-07.

> **Open issue OI-FSR-001-05:** Confirm lamp activation is achievable within the portion of the 50 ms FTTI remaining after SW and hydraulic budgets are allocated.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Warning lamp activated within 50 ms FTTI window from fault detection | HIL fault injection + oscilloscope / CAN timestamp | Lamp activation confirmed < 50 ms from fault detection in all injected scenarios |
| DTC stored in NVM within one diagnostic task cycle of fault detection | UDS DTC read-back — Service 0x19 | DTC present; DTC status byte correct immediately after fault injection |
| Lamp maintained throughout deactivated state | HIL state monitoring | Lamp signal active for full duration of deactivated state |
| Lamp extinguishes only after authorised reset per FSR-001-08 | HIL + UDS reset test | Lamp remains active until authorised reset procedure; extinguishes only following reset |

---

### FSR-001-06 — Hydraulic actuator release confirmation

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW + HW  
**Category:** Detection

**Requirement:**

The ESC shall monitor the physical brake pressure at each wheel channel via a pressure sensor or equivalent feedback mechanism. Following issuance of a zero-pressure command (as required by FSR-001-03), the ESC shall verify that the measured wheel pressure falls below [TBD bar] within the hydraulic release time budget defined in the HSI. If the measured pressure does not fall to within the release tolerance within the budget, the ESC shall classify this as a hydraulic actuator fault and shall set a separate DTC for the affected channel. The ESC shall remain in the deactivated state and shall not re-enable ESC function while this fault is active.

> **Open issue OI-FSR-001-06:** Pressure feedback sensor availability and accuracy must be confirmed with the HW team and specified in the HSI. If direct pressure feedback is not available, an alternative actuator release confirmation mechanism (e.g., valve current sense) must be agreed and documented.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Measured pressure falls below release threshold within hydraulic release budget after zero-pressure command | HIL — pressure transducer measurement following zero-pressure command | Pressure confirmed < [TBD bar] within HSI-specified budget in all scenarios |
| Stuck-valve fault (pressure held above threshold) correctly detected and classified | HIL fault injection — pressure held above threshold | Correct stuck-valve DTC set for affected channel; ESC remains deactivated |
| ESC does not re-enable while actuator fault DTC is active | HIL state monitoring | Zero ESC re-enable attempts observed while actuator fault is active |
| Pressure sensor accuracy consistent with detection requirements | HSI review + sensor specification | Sensor accuracy confirmed in HSI as adequate for [TBD bar] release threshold |

---

### FSR-001-07 — Safe state persistence after fault detection

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW  
**Category:** Reaction

**Requirement:**

Following transition to the deactivated state as required by FSR-001-03, the ESC shall maintain the deactivated state and shall not autonomously re-enable ESC function without an explicit re-enable sequence. The re-enable sequence shall require at minimum one of: (a) a successful power cycle in which the start-up self-test defined in FSR-001-04 passes with the relevant fault no longer active; or (b) an authorised diagnostic reset via a UDS Security Access level appropriate for the vehicle variant. The ESC shall not re-enable based solely on a transient fault no longer being detected.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| ESC does not autonomously re-enable after fault injection removal | HIL state monitoring — fault injected then removed | ESC remains deactivated; no re-enable attempt observed |
| ESC re-enables after authorised power cycle (fault cleared) | HIL power cycle test — fault removed + power cycle | ESC enable confirmed following clean power cycle with fault no longer active |
| ESC re-enables after authorised diagnostic reset | HIL + UDS Security Access reset | ESC enable confirmed following diagnostic reset at correct access level |
| Transient fault clearance alone does not trigger re-enable | HIL — rapid fault inject and remove | ESC remains deactivated; explicit reset required before re-enable |

---

### FSR-001-08 — Non-volatile retention of deactivated state across power cycle

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW  
**Category:** Reaction

**Requirement:**

The deactivated state and the associated DTC shall be stored in non-volatile memory at the time of fault detection. Following any power cycle or ECU reset, the ESC shall read the non-volatile state at start-up. If a deactivation record is present and the DTC has not been explicitly cleared by an authorised service operation, the ESC shall not enable ESC function. The non-volatile deactivation record shall be protected by a CRC or equivalent integrity mechanism to detect corruption. A corrupted record shall be treated as a deactivation condition.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Deactivation state and DTC persisted to NVM at fault detection | HIL + NVM read-back | NVM record confirmed written; DTC present immediately after fault injection |
| ESC does not re-enable after power cycle with active deactivation record | HIL power cycle + state monitoring | ESC remains deactivated after power cycle; no actuation observed |
| ESC re-enables after authorised DTC clear followed by power cycle | HIL + UDS DTC clear (Service 0x14) + power cycle | ESC enable confirmed on power cycle after authorised DTC clear |
| NVM corruption (CRC mismatch) treated as deactivation condition | HIL NVM corruption injection (force CRC mismatch) | ESC remains deactivated; NVM corruption DTC set |

---

### FSR-001-09 — Hard-braking false-detection inhibit

**Parent Safety Goal:** SG-001 — ASIL D (see [section header above](#fsrs-from-sg-001-asil-d))  
**Allocation:** SW  
**Category:** Prevention

**Requirement:**

The ESC shall implement a hard-braking detection function that identifies when the driver is applying a high-deceleration braking demand (threshold: [TBD m/s² longitudinal deceleration] OR [TBD bar master cylinder pressure]). When the hard-braking condition is active, the ESC shall inhibit individual wheel braking interventions that are based solely on yaw rate or lateral acceleration deviations, to prevent false-positive instability detection caused by the asymmetric wheel speed signatures inherent to heavy straight-line braking (hazardous event H-002). The inhibit shall apply only to ESC-commanded asymmetric pressure; ABS authority as defined in FSR-004-01 shall be unaffected.

> **Open issue OI-FSR-001-09:** Hard-braking threshold values ([TBD m/s²], [TBD bar]) shall be derived from vehicle dynamics simulation and confirmed via HIL test.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Hard-braking inhibit correctly engaged at defined threshold | SIL back-to-back vs reference model — hard-braking manoeuvre profiles | Inhibit flag matches reference in all hard-braking test vectors |
| No asymmetric ESC wheel pressure command issued during inhibit | HIL — straight-line brake at highway speed | Zero asymmetric ESC pressure commands observed when inhibit active |
| ABS authority unaffected by hard-braking inhibit | HIL — combined ABS + hard-braking scenario | ABS wheel-slip control uninterrupted while inhibit active |
| Inhibit releases correctly when hard-braking condition ends | SIL + HIL — brake release manoeuvre | ESC resumes normal yaw rate monitoring after hard-braking condition clears |

---

## FSRs from SG-002 (ASIL C)

**Safety Goal:** ESC shall detect vehicle instability and initiate corrective braking within [TBD ms] across all operating conditions.

**Item:** Electronic Stability Control (ESC) — see [HARA §1 item boundary](hara-esc.md).

**Hazardous Events Addressed:** H-003 (failure to activate during oversteer/understeer on icy curve), H-006 (incorrect wheel selection during emergency evasive maneuver) — see [HARA §4](hara-esc.md).

**ASIL:** **C** (worst-case hazard determines goal ASIL)

**ASIL Derivation:**

| Parameter | Rating | Rationale |
|-----------|--------|-----------|
| Severity | S3 | Off-road excursion into barrier or ditch at speed is life-threatening |
| Exposure | E2 | Icy road or emergency evasive conditions occur 2–10% of operating time |
| Controllability | C3 | Loss of control on ice or during yaw overshoot is largely unrecoverable |

**FTTI:** 200 ms  
> Budget rationale: yaw rate divergence on low-friction surfaces develops over approximately 300–500 ms; 200 ms FTTI provides approximately 100–300 ms for corrective intervention to take effect. Subject to validation per OI-01.

**Safe State:** ESC deactivated with driver notification; driver retains full manual control.

- ESC actuation suspended; no corrective braking or torque commands issued
- ESC warning lamp illuminated to inform driver of degraded stability assist
- ABS and base braking retained at full authority
- DTC stored with freeze-frame data (yaw rate deviation, wheel speeds, road speed)
- System remains in degraded mode until driver cycle or service reset

---

### FSR-002-01 — Continuous yaw rate deviation calculation

**Parent Safety Goal:** SG-002 — ASIL C (see [section header above](#fsrs-from-sg-002-asil-c))  
**Allocation:** SW  
**Category:** Prevention

**Requirement:**

The ESC shall continuously calculate the deviation between the measured yaw rate and the driver-intended yaw rate (derived from steering angle, vehicle speed, and vehicle model). The calculation shall execute at a minimum rate of [TBD Hz] and shall complete within the task WCET budget.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Yaw rate deviation calculated correctly across all input combinations | SIL back-to-back vs reference model | Output matches reference within tolerance across all test scenarios |
| Calculation completes within task WCET budget | WCET analysis | Measured WCET ≤ allocated SW budget |
| Calculation executes at minimum required rate | HIL task timing measurement | Calculation period ≤ 1 / [TBD Hz] in all operating modes |

---

### FSR-002-02 — Instability detection threshold and timing

**Parent Safety Goal:** SG-002 — ASIL C (see [section header above](#fsrs-from-sg-002-asil-c))  
**Allocation:** SW  
**Category:** Detection

**Requirement:**

When the yaw rate deviation exceeds [TBD deg/s] for longer than [TBD ms], or the lateral acceleration exceeds [TBD m/s²], the ESC shall classify the vehicle as unstable and shall initiate corrective wheel braking within [TBD ms] of classification. Thresholds shall be calibratable per vehicle variant.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Instability correctly classified at defined thresholds | HIL dynamic manoeuvres — step steer, sine sweep, icy curve simulation | Classification matches reference for all standardised test scenarios |
| Corrective braking initiated within [TBD ms] of instability classification | HIL timing measurement | Response time from classification flag to first pressure command < [TBD ms] |
| Threshold values correctly applied from variant calibration data | SIL variant calibration test | All threshold values configurable and applied correctly from calibration |

---

### FSR-002-03 — Sensor plausibility monitoring

**Parent Safety Goal:** SG-002 — ASIL C (see [section header above](#fsrs-from-sg-002-asil-c))  
**Allocation:** SW  
**Category:** Detection

**Requirement:**

The ESC shall continuously monitor the plausibility of yaw rate sensor, lateral accelerometer, steering angle sensor, and all four wheel speed sensors. Plausibility checks shall include: range validation, rate-of-change validation, and cross-sensor consistency checks. A sensor classified as implausible shall be excluded from the stability calculation; if the minimum sensor set for reliable detection is unavailable, ESC shall deactivate.

> **Open issue OI-03:** Minimum sensor set for degraded mode must be defined with the systems team.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Out-of-range sensor correctly classified as implausible | HIL sensor fault injection — range violation per channel | Implausible classification triggered for all out-of-range injected values |
| Rate-of-change violation correctly detected | HIL sensor fault injection — step change | Implausible classification triggered for step changes exceeding rate limit |
| Cross-sensor inconsistency correctly detected | HIL multi-sensor fault injection | Inconsistency detected when sensor deviates from cross-sensor model |
| ESC deactivates when minimum sensor set unavailable | HIL — faults injected on multiple sensors simultaneously | ESC deactivates within FTTI; warning lamp active; DTC stored |

---

### FSR-002-04 — Degraded mode notification

**Parent Safety Goal:** SG-002 — ASIL C (see [section header above](#fsrs-from-sg-002-asil-c))  
**Allocation:** SW  
**Category:** Reaction

**Requirement:**

If ESC function is degraded or deactivated due to a sensor fault, the ESC warning lamp shall be activated and a DTC shall be stored. The driver shall be informed that ESC is unavailable within [TBD ms] of the degradation condition being confirmed.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Warning lamp activates within [TBD ms] of degradation confirmation | HIL fault injection + CAN / oscilloscope measurement | Lamp activation < [TBD ms] from degradation detection in all scenarios |
| DTC stored with correct status on degradation | UDS DTC read-back — Service 0x19 | DTC present; DTC status byte consistent after fault injection |
| Lamp maintained throughout degraded / deactivated state | HIL state monitoring | Lamp signal active for full duration of degraded state |

---

## FSRs from SG-003 (ASIL B)

**Safety Goal:** ESC shall not command engine torque reduction greater than [TBD %] without a confirmed instability event.

**Item:** Electronic Stability Control (ESC) — see [HARA §1 item boundary](hara-esc.md).

**Hazardous Events Addressed:** H-004 (unintended torque reduction at motorway cruise), H-005 (continuous unintended interventions in urban driving) — see [HARA §4](hara-esc.md).

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

---

### FSR-003-01 — Torque reduction gating on confirmed instability

**Parent Safety Goal:** SG-003 — ASIL B (see [section header above](#fsrs-from-sg-003-asil-b))  
**Allocation:** SW  
**Category:** Prevention

**Requirement:**

The ESC shall only issue an engine torque reduction request when a vehicle instability condition has been confirmed per FSR-002-02. Torque reduction requests shall be inhibited during all other vehicle states.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Torque reduction request not issued when instability condition inactive | Unit test — all state machine paths | Zero torque reduction requests generated in absence of confirmed instability flag |
| Torque reduction correctly issued when instability is confirmed | SIL scenario testing — confirmed instability stimuli | Torque reduction issued within timing requirement for all confirmed instability scenarios |
| State machine transitions fully covered | Unit test — state machine branch coverage | 100% branch coverage of torque gating state machine achieved |

---

### FSR-003-02 — Maximum torque reduction limit without confirmed instability

**Parent Safety Goal:** SG-003 — ASIL B (see [section header above](#fsrs-from-sg-003-asil-b))  
**Allocation:** SW  
**Category:** Prevention

**Requirement:**

If a torque reduction request is issued without an active confirmed instability condition (as a result of a software fault), the ESC shall limit the torque reduction to a maximum of [TBD %] of driver-demanded torque. The limiting function shall be implemented independently of the torque reduction request path.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Torque reduction magnitude limited to ≤ [TBD %] during spurious command | HIL fault injection — force torque reduction without instability condition | Commanded torque reduction ≤ [TBD %] in all injected scenarios |
| Limiting function operates independently of request path | Architecture review + code independence audit | Limiter confirmed in separate module with no shared state with request path |
| Limiter cannot be bypassed by single software fault | Fault injection — disable limiter path | Full unintended torque reduction not achievable with limiter disabled |

---

### FSR-003-03 — Unintended torque reduction detection and suppression

**Parent Safety Goal:** SG-003 — ASIL B (see [section header above](#fsrs-from-sg-003-asil-b))  
**Allocation:** SW  
**Category:** Detection + Reaction

**Requirement:**

The ESC shall monitor the torque reduction command output and compare it against the vehicle state. A torque reduction command issued when no instability condition is active shall be classified as unintended. The ESC shall suppress the command and transition to deactivated state within 100 ms of detection.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Unintended torque reduction correctly classified when no instability active | HIL — inject spurious torque reduction command | Unintended classification triggered; command suppressed |
| Suppression and deactivation completed within 100 ms FTTI | HIL timing measurement | Fault-to-safe-state (full torque restored) < 100 ms in all scenarios |
| ESC transitions to deactivated state on detection | HIL state monitoring | ESC deactivated after suppression; warning lamp active; DTC stored |

---

## FSRs from SG-004 (ASIL B)

**Safety Goal:** ESC shall not reduce ABS control authority during a combined braking and steering manoeuvre.

**Item:** Electronic Stability Control (ESC) — see [HARA §1 item boundary](hara-esc.md).

**Hazardous Events Addressed:** H-007 (ESC state machine conflict suspends ABS authority during combined braking/steering) — see [HARA §4](hara-esc.md).

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

---

### FSR-004-01 — ABS priority arbitration

**Parent Safety Goal:** SG-004 — ASIL B (see [section header above](#fsrs-from-sg-004-asil-b))  
**Allocation:** SW  
**Category:** Prevention

**Requirement:**

The ESC shall implement a wheel pressure arbitration function that gives ABS unconditional priority over ESC for individual wheel brake pressure commands. When ABS is active on any wheel, the ESC shall not reduce the ABS-commanded pressure on that wheel.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| ABS pressure command never reduced by ESC arbitration when ABS is active | Unit test — arbitration logic, all input combinations | Zero cases where ESC output reduces ABS-commanded pressure |
| ABS priority maintained across all concurrent ABS + ESC activation combinations | HIL — combined ABS + ESC activation | ABS wheel-slip control uninterrupted across all combined activation scenarios |
| Arbitration function fully covered | Unit test — state + branch coverage | 100% branch coverage of arbitration logic confirmed |

---

### FSR-004-02 — Detection of simultaneous ABS and ESC activation

**Parent Safety Goal:** SG-004 — ASIL B (see [section header above](#fsrs-from-sg-004-asil-b))  
**Allocation:** SW  
**Category:** Detection + Reaction

**Requirement:**

The ESC shall detect simultaneous activation of ABS and ESC functions within [TBD ms]. Upon detection, the ESC shall suspend all individual wheel braking interventions and yield full pressure authority to ABS. The ESC torque reduction function may remain active subject to FSR-003-01.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| Simultaneous ABS and ESC activation detected within [TBD ms] | HIL — simultaneous ABS + ESC trigger | Detection confirmed < [TBD ms] in all test scenarios |
| ESC individual wheel interventions suspended within 20 ms FTTI | HIL timing measurement | ESC wheel pressure commands cease < 20 ms after detection in all scenarios |
| ESC torque reduction function unaffected by suspension | HIL signal monitoring | Torque reduction channel remains active (subject to FSR-003-01) during ABS priority mode |

---

### FSR-004-03 — ABS authority restoration on ESC reactivation

**Parent Safety Goal:** SG-004 — ASIL B (see [section header above](#fsrs-from-sg-004-asil-b))  
**Allocation:** SW  
**Category:** Reaction

**Requirement:**

When ABS deactivates, the ESC may resume individual wheel interventions only after re-evaluating the current stability condition. ESC shall not resume wheel braking based on a stability condition that was assessed prior to the ABS event.

**Verification Criteria:**

| Criterion | Method | Pass Condition |
|-----------|--------|----------------|
| ESC does not resume wheel interventions based on pre-ABS stability assessment | HIL — ABS deactivation with stale stability condition active | Zero ESC wheel pressure commands until fresh stability re-evaluation completes |
| State reset confirmed on ABS deactivation | HIL state monitoring | ESC state machine confirmed reset to idle on ABS deactivation event |
| ESC re-evaluation completes before resuming interventions | HIL sequence timing | No ESC hydraulic commands observed until new stability assessment confirms need |

---

## FSR Summary

| FSR ID     | Parent SG | ASIL | Category | Summary | Allocation |
|------------|-----------|------|----------|---------|------------|
| FSR-001-01 | SG-001    | D    | Prevention + Detection | Wheel pressure plausibility monitoring (incl. hard-braking mode) | SW + HW |
| FSR-001-02 | SG-001    | D (B+B) | Detection | Independent monitoring channel — ASIL D decomposition B(D)+B(D) | SW + HW |
| FSR-001-03 | SG-001    | D    | Reaction | Deactivation within 50 ms FTTI — end-to-end including hydraulic release | SW + HW |
| FSR-001-04 | SG-001    | D    | Prevention | Power-on self-test of brake command path | SW |
| FSR-001-05 | SG-001    | D    | Reaction | Warning lamp within 50 ms FTTI + DTC in NVM | SW |
| FSR-001-06 | SG-001    | D    | Detection | Hydraulic actuator release confirmation (stuck-valve detection) | SW + HW |
| FSR-001-07 | SG-001    | D    | Reaction | Safe state persistence — no autonomous re-enable | SW |
| FSR-001-08 | SG-001    | D    | Reaction | Non-volatile retention of deactivated state across power cycle | SW |
| FSR-001-09 | SG-001    | D    | Prevention | Hard-braking false-detection inhibit (H-002 explicit coverage) | SW |
| FSR-002-01 | SG-002    | C    | Prevention | Continuous yaw rate deviation calculation | SW |
| FSR-002-02 | SG-002    | C    | Detection | Instability detection threshold and timing | SW |
| FSR-002-03 | SG-002    | C    | Detection | Sensor plausibility monitoring | SW |
| FSR-002-04 | SG-002    | C    | Reaction | Degraded mode notification | SW |
| FSR-003-01 | SG-003    | B    | Prevention | Torque reduction gating on confirmed instability | SW |
| FSR-003-02 | SG-003    | B    | Prevention | Maximum torque reduction limit without instability | SW |
| FSR-003-03 | SG-003    | B    | Detection + Reaction | Unintended torque reduction detection + suppression | SW |
| FSR-004-01 | SG-004    | B    | Prevention | ABS priority arbitration | SW |
| FSR-004-02 | SG-004    | B    | Detection + Reaction | Simultaneous ABS + ESC detection and suspension | SW |
| FSR-004-03 | SG-004    | B    | Reaction | ABS authority restoration on ESC reactivation | SW |

---

## Open Issues

| ID | Issue | Blocking FSR | Owner | Target Date |
|----|-------|-------------|-------|-------------|
| OI-01 | FTTI values for SG-001 and SG-002 require vehicle dynamics validation | FSR-001-03 | TBD | TBD |
| OI-02 | ASIL D decomposition for FSR-001-02 to be confirmed and DFA completed in system architecture phase | FSR-001-02 | Systems Safety | TBD |
| OI-03 | Minimum sensor set for FSR-002-03 degraded mode to be defined with systems team | FSR-002-03 | Systems Team | TBD |
| OI-04 | HSI specification for hydraulic valve release time needed from HW team — blocks SW reaction time budget in FSR-001-03 | FSR-001-03, FSR-001-06 | HW Team | TBD |
| OI-FSR-001-01a | Pressure plausibility threshold [TBD bar] — derive from vehicle dynamics simulation | FSR-001-01 | Vehicle Dynamics | TBD |
| OI-FSR-001-01b | Plausibility window duration [TBD ms] — derive from vehicle dynamics simulation | FSR-001-01 | Vehicle Dynamics | TBD |
| OI-FSR-001-05 | Confirm warning lamp activation is achievable within the portion of the 50 ms FTTI remaining after SW and hydraulic budgets are allocated | FSR-001-05 | Systems Safety | TBD |
| OI-FSR-001-06 | Pressure feedback sensor availability and accuracy to be confirmed in HSI; if unavailable, alternative actuator feedback mechanism to be agreed | FSR-001-06 | HW Team | TBD |
| OI-FSR-001-09 | Hard-braking detection thresholds [TBD m/s², TBD bar] — derive from vehicle dynamics simulation for H-002 scenarios | FSR-001-09 | Vehicle Dynamics | TBD |

---

## Traceability

| Safety Goal | Hazardous Events | FSRs Derived |
|-------------|-----------------|-------------|
| SG-001 (ASIL D) | H-001, H-002 | FSR-001-01, FSR-001-02, FSR-001-03, FSR-001-04, FSR-001-05, FSR-001-06, FSR-001-07, FSR-001-08, FSR-001-09 |
| SG-002 (ASIL C) | H-003, H-006 | FSR-002-01, FSR-002-02, FSR-002-03, FSR-002-04 |
| SG-003 (ASIL B) | H-004, H-005 | FSR-003-01, FSR-003-02, FSR-003-03 |
| SG-004 (ASIL B) | H-007 | FSR-004-01, FSR-004-02, FSR-004-03 |

---

## Next Steps

1. **HSI specification** — Obtain hydraulic valve release time from HW team to finalise SW reaction time budget (OI-04); confirm pressure feedback sensor availability (OI-FSR-001-06)
2. **ASIL D decomposition** — Complete DFA for FSR-001-02 B(D)+B(D) decomposition and baseline in system architecture document (OI-02)
3. **Vehicle dynamics simulation** — Derive all [TBD] threshold and timing values (OI-FSR-001-01a/b, OI-FSR-001-09)
4. **Technical Safety Requirements (TSRs)** — Refine FSRs into implementable TSRs with concrete threshold values once simulation results are available
5. **FMEA** — Map each FSR to failure modes in the ESC software DFMEA; FSR-001-06 maps to stuck-valve failure mode not yet in DFMEA
6. **Test specification** — Derive test cases for each FSR; FSR-001-02 requires fault injection at both main path and monitor path independently; FSR-001-08 requires NVM corruption injection

---

## Revision History

| Version | Date       | Author | Changes |
|---------|------------|--------|---------|
| 1.0     | 2026-04-06 | —      | Initial draft |
| 1.1     | 2026-04-08 | —      | SG-001 FSRs revised and extended: formal ASIL D decomposition in FSR-001-02; end-to-end FTTI budget in FSR-001-03; lamp timing bounded in FSR-001-05; added FSR-001-06 (stuck-valve detection), FSR-001-07 (safe state persistence), FSR-001-08 (NVM retention), FSR-001-09 (hard-braking inhibit for H-002); added Prevention/Detection/Reaction category field to all FSRs; open issues expanded per FSR |
| 1.2     | 2026-04-08 | —      | Reformatted all 19 FSRs and SG section headers to ISO 26262 skill template structure: each SG block now includes ASIL derivation table (S/E/C with rationale), FTTI with budget rationale, and full safe state bullet list; each FSR now includes Parent Safety Goal reference, Allocation, Category, Requirement as narrative paragraph, and structured Verification Criteria table (criterion / method / pass condition) |
