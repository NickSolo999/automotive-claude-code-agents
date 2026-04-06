# Functional Safety Requirements — Electronic Stability Control (ESC)

**Standard:** ISO 26262-4  
**Date:** 2026-04-06  
**Status:** Draft  
**Version:** 1.0  
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

Allocation to SW, HW, or both is indicated. Final allocation is confirmed during
system architectural design (ISO 26262-4 §7).

---

## FSRs from SG-001 (ASIL D)
**Safety Goal:** ESC shall not apply unintended individual wheel brake pressure exceeding [TBD bar] during normal driving conditions.  
**Safe state:** Remove all ESC braking commands; deactivate ESC function within 50 ms.

---

### FSR-001-01 — Wheel brake pressure plausibility monitoring
| Field | Value |
|-------|-------|
| ASIL | **D** |
| Allocation | SW + HW |
| Text | The ESC system shall continuously compare the commanded brake pressure for each wheel against a plausibility envelope derived from driver brake demand, vehicle speed, lateral acceleration, and yaw rate. A commanded pressure that deviates from the plausibility envelope by more than [TBD bar] for longer than [TBD ms] shall be classified as an unintended intervention. |
| Safety mechanism | Plausibility cross-check (prevention + detection) |
| Verification | Unit test, back-to-back SIL, HIL fault injection |

---

### FSR-001-02 — Independent monitoring channel for brake command path
| Field | Value |
|-------|-------|
| ASIL | **D** |
| Allocation | SW (monitor path) + HW (separate monitor channel) |
| Text | The ESC shall implement a monitoring function that is independent of the main brake command calculation path. The monitoring function shall observe the output brake pressure commands and shall suppress transmission of any command that is classified as unintended per FSR-001-01. The monitoring function shall not share source code, calibration data, or hardware execution context with the main control path. |
| Safety mechanism | Redundant independent monitor (ASIL D decomposition candidate: main path ASIL B(D) + monitor ASIL B(D)) |
| Verification | Architecture review, code independence audit, HIL dual-channel fault injection |

---

### FSR-001-03 — Deactivation within FTTI on unintended pressure detection
| Field | Value |
|-------|-------|
| ASIL | **D** |
| Allocation | SW + HW |
| Text | Upon detection of an unintended wheel brake pressure command, the ESC shall set all wheel brake pressure commands to zero and transition to the deactivated state within 50 ms of fault detection. The hydraulic actuator shall physically release within [TBD ms] of the command being removed (to be confirmed via HSI). |
| Safety mechanism | Fault reaction with bounded response time |
| Verification | HIL timing measurement, worst-case execution time (WCET) analysis |

---

### FSR-001-04 — Power-on self-test of brake command path
| Field | Value |
|-------|-------|
| ASIL | **D** |
| Allocation | SW |
| Text | At every ECU power-on, the ESC shall execute a self-test of the brake pressure command path before enabling ESC function. The self-test shall verify sensor read-back, actuator enable logic, and monitoring channel operation. ESC shall not activate if any self-test step fails. |
| Safety mechanism | Start-up diagnostic |
| Verification | Unit test, HIL self-test injection |

---

### FSR-001-05 — ESC warning lamp activation on deactivation
| Field | Value |
|-------|-------|
| ASIL | **D** |
| Allocation | SW |
| Text | When ESC transitions to the deactivated state due to a detected fault, the ESC warning lamp shall be illuminated within [TBD ms] and a corresponding DTC shall be stored in non-volatile memory. |
| Safety mechanism | Driver notification |
| Verification | HIL test, UDS DTC read-back |

---

## FSRs from SG-002 (ASIL C)
**Safety Goal:** ESC shall detect vehicle instability and initiate corrective braking within [TBD ms] across all operating conditions.  
**Safe state:** Disable ESC; illuminate ESC warning lamp.

---

### FSR-002-01 — Continuous yaw rate deviation calculation
| Field | Value |
|-------|-------|
| ASIL | **C** |
| Allocation | SW |
| Text | The ESC shall continuously calculate the deviation between the measured yaw rate and the driver-intended yaw rate (derived from steering angle, vehicle speed, and vehicle model). The calculation shall execute at a minimum rate of [TBD Hz] and shall complete within the task WCET budget. |
| Safety mechanism | Core stability detection function (prevention) |
| Verification | SIL back-to-back vs reference model, HIL steady-state and dynamic scenarios |

---

### FSR-002-02 — Instability detection threshold and timing
| Field | Value |
|-------|-------|
| ASIL | **C** |
| Allocation | SW |
| Text | When the yaw rate deviation exceeds [TBD deg/s] for longer than [TBD ms], or the lateral acceleration exceeds [TBD m/s²], the ESC shall classify the vehicle as unstable and shall initiate corrective wheel braking within [TBD ms] of classification. Thresholds shall be calibratable per vehicle variant. |
| Safety mechanism | Detection + bounded response time |
| Verification | HIL dynamic manoeuvres (step steer, sine sweep, icy curve simulation) |

---

### FSR-002-03 — Sensor plausibility monitoring
| Field | Value |
|-------|-------|
| ASIL | **C** |
| Allocation | SW |
| Text | The ESC shall continuously monitor the plausibility of yaw rate sensor, lateral accelerometer, steering angle sensor, and all four wheel speed sensors. Plausibility checks shall include: range validation, rate-of-change validation, and cross-sensor consistency checks. A sensor classified as implausible shall be excluded from the stability calculation; if the minimum sensor set for reliable detection is unavailable, ESC shall deactivate. |
| Safety mechanism | Input plausibility monitoring |
| Verification | HIL sensor fault injection per sensor channel |

---

### FSR-002-04 — Degraded mode notification
| Field | Value |
|-------|-------|
| ASIL | **C** |
| Allocation | SW |
| Text | If ESC function is degraded or deactivated due to a sensor fault, the ESC warning lamp shall be activated and a DTC shall be stored. The driver shall be informed that ESC is unavailable within [TBD ms] of the degradation condition being confirmed. |
| Safety mechanism | Driver notification |
| Verification | HIL fault injection + lamp output measurement |

---

## FSRs from SG-003 (ASIL B)
**Safety Goal:** ESC shall not command engine torque reduction greater than [TBD %] without a confirmed instability event.  
**Safe state:** Restore full torque demand; deactivate ESC within 100 ms.

---

### FSR-003-01 — Torque reduction gating on confirmed instability
| Field | Value |
|-------|-------|
| ASIL | **B** |
| Allocation | SW |
| Text | The ESC shall only issue an engine torque reduction request when a vehicle instability condition has been confirmed per FSR-002-02. Torque reduction requests shall be inhibited during all other vehicle states. |
| Safety mechanism | Logical gate — prevention of unintended command |
| Verification | Unit test (state machine coverage), SIL scenario testing |

---

### FSR-003-02 — Maximum torque reduction limit without confirmed instability
| Field | Value |
|-------|-------|
| ASIL | **B** |
| Allocation | SW |
| Text | If a torque reduction request is issued without an active confirmed instability condition (as a result of a software fault), the ESC shall limit the torque reduction to a maximum of [TBD %] of driver-demanded torque. The limiting function shall be implemented independently of the torque reduction request path. |
| Safety mechanism | Output value limitation |
| Verification | HIL fault injection — force torque reduction command without instability condition |

---

### FSR-003-03 — Unintended torque reduction detection and suppression
| Field | Value |
|-------|-------|
| ASIL | **B** |
| Allocation | SW |
| Text | The ESC shall monitor the torque reduction command output and compare it against the vehicle state. A torque reduction command issued when no instability condition is active shall be classified as unintended. The ESC shall suppress the command and transition to deactivated state within 100 ms of detection. |
| Safety mechanism | Command plausibility monitor + fault reaction |
| Verification | HIL injection of spurious torque reduction command, timing measurement |

---

## FSRs from SG-004 (ASIL B)
**Safety Goal:** ESC shall not reduce ABS control authority during a combined braking and steering manoeuvre.  
**Safe state:** ABS retains full priority; ESC suspends all interventions within 20 ms.

---

### FSR-004-01 — ABS priority arbitration
| Field | Value |
|-------|-------|
| ASIL | **B** |
| Allocation | SW |
| Text | The ESC shall implement a wheel pressure arbitration function that gives ABS unconditional priority over ESC for individual wheel brake pressure commands. When ABS is active on any wheel, the ESC shall not reduce the ABS-commanded pressure on that wheel. |
| Safety mechanism | Arbitration with defined priority order |
| Verification | Unit test (arbitration logic), HIL combined ABS+ESC activation scenario |

---

### FSR-004-02 — Detection of simultaneous ABS and ESC activation
| Field | Value |
|-------|-------|
| ASIL | **B** |
| Allocation | SW |
| Text | The ESC shall detect simultaneous activation of ABS and ESC functions within [TBD ms]. Upon detection, the ESC shall suspend all individual wheel braking interventions and yield full pressure authority to ABS. The ESC torque reduction function may remain active subject to FSR-003-01. |
| Safety mechanism | State detection + conditional deactivation |
| Verification | HIL: trigger ABS at full ESC activation, verify suspension within 20 ms |

---

### FSR-004-03 — ABS authority restoration on ESC reactivation
| Field | Value |
|-------|-------|
| ASIL | **B** |
| Allocation | SW |
| Text | When ABS deactivates, the ESC may resume individual wheel interventions only after re-evaluating the current stability condition. ESC shall not resume wheel braking based on a stability condition that was assessed prior to the ABS event. |
| Safety mechanism | State reset on mode transition |
| Verification | HIL: ABS activation/deactivation cycle during active ESC, verify clean state re-entry |

---

## FSR Summary

| FSR ID     | Parent SG | ASIL | Summary                                              | Allocation |
|------------|-----------|------|------------------------------------------------------|------------|
| FSR-001-01 | SG-001    | D    | Wheel pressure plausibility monitoring               | SW + HW    |
| FSR-001-02 | SG-001    | D    | Independent monitoring channel for brake commands    | SW + HW    |
| FSR-001-03 | SG-001    | D    | Deactivation within FTTI on unintended pressure      | SW + HW    |
| FSR-001-04 | SG-001    | D    | Power-on self-test of brake command path             | SW         |
| FSR-001-05 | SG-001    | D    | Warning lamp + DTC on deactivation                   | SW         |
| FSR-002-01 | SG-002    | C    | Continuous yaw rate deviation calculation            | SW         |
| FSR-002-02 | SG-002    | C    | Instability detection threshold and timing           | SW         |
| FSR-002-03 | SG-002    | C    | Sensor plausibility monitoring                       | SW         |
| FSR-002-04 | SG-002    | C    | Degraded mode notification                           | SW         |
| FSR-003-01 | SG-003    | B    | Torque reduction gating on confirmed instability     | SW         |
| FSR-003-02 | SG-003    | B    | Maximum torque reduction limit without instability   | SW         |
| FSR-003-03 | SG-003    | B    | Unintended torque reduction detection + suppression  | SW         |
| FSR-004-01 | SG-004    | B    | ABS priority arbitration                             | SW         |
| FSR-004-02 | SG-004    | B    | Simultaneous ABS + ESC detection and suspension      | SW         |
| FSR-004-03 | SG-004    | B    | ABS authority restoration on ESC reactivation        | SW         |

---

## Open Issues

| ID    | Issue                                                                              | Owner | Target Date |
|-------|------------------------------------------------------------------------------------|-------|-------------|
| OI-01 | All [TBD] threshold and timing values require vehicle dynamics simulation           | TBD   | TBD         |
| OI-02 | ASIL D decomposition for FSR-001-02 to be confirmed in system architecture phase   | TBD   | TBD         |
| OI-03 | Minimum sensor set for FSR-002-03 degraded mode to be defined with systems team    | TBD   | TBD         |
| OI-04 | HSI specification for hydraulic release time (FSR-001-03) needed from HW team      | TBD   | TBD         |

---

## Traceability

| Safety Goal | FSRs Derived |
|-------------|--------------|
| SG-001 (ASIL D) | FSR-001-01, FSR-001-02, FSR-001-03, FSR-001-04, FSR-001-05 |
| SG-002 (ASIL C) | FSR-002-01, FSR-002-02, FSR-002-03, FSR-002-04 |
| SG-003 (ASIL B) | FSR-003-01, FSR-003-02, FSR-003-03 |
| SG-004 (ASIL B) | FSR-004-01, FSR-004-02, FSR-004-03 |

---

## Next Steps

1. **System architecture** — Allocate FSRs to ECU hardware/software elements; confirm ASIL D decomposition for FSR-001-02
2. **HSI specification** — Define sensor accuracy and actuator timing requirements that constrain FSR-002-01/02/03 and FSR-001-03
3. **Technical Safety Requirements (TSRs)** — Refine FSRs into implementable TSRs with concrete threshold values once simulation results are available
4. **FMEA** — Map each FSR to failure modes in the ESC software DFMEA
5. **Test specification** — Derive test cases for each FSR; FSR-001-02 requires fault injection at both main path and monitor path independently

---

## Revision History

| Version | Date       | Author | Changes       |
|---------|------------|--------|---------------|
| 1.0     | 2026-04-06 | —      | Initial draft |
