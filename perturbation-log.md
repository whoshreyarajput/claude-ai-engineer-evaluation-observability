# Perturbation and Resilience Log

## 1. Insurance Policy Extraction Pipeline

**Scenario:** Missing required source evidence.

**Method:** Ran the existing test `test_ac_01_04_missing_source_halts_immediately`.

**Result:** 1 test passed.

**Observation:**
- The missing-source safeguard was verified.
- The live pipeline identified missing endorsements for `POL-2025-009`.
- A premium consistency issue was identified for `POL-2025-010`.

**Limitation:** This was an existing test scenario, not a manually modified policy document.

---

## 2. Mortgage Document Extraction System

**Scenario:** Monthly income inconsistency.

**Method:** Executed the existing `income_sum_mismatch.txt` replay fixture.

**Observed Values:**

| Metric | Value |
|---|---:|
| Calculated monthly income | $9,642.17 |
| Stated monthly income | $10,892.17 |
| Difference | -$1,250.00 |
| Validation result | Inconsistent |

**Observation:**
- The validator detected the income discrepancy.
- The missing-bonus scenario preserved missing values as `null` rather than treating them as zero.

**Limitation:** These were existing replay fixtures, not manually modified inputs.

---

## 3. Supply Chain Risk Investigation

**Scenario:** Logistics source becomes unavailable.

**Method:** Used the CLI's `--simulate-timeout` option.

**Result:** The investigation completed and generated a partial briefing.

**Observations:**
- The logistics timeout did not abort the investigation.
- Findings from other sources remained available.
- Late shipment count was marked incomplete.
- The report identified the unavailable logistics source.

**Limitation:** The timeout was simulated, not caused by a real external outage.

---

## 4. Overall Learning

The three projects demonstrate the importance of validation, missing-data handling, evidence preservation, and system resilience.

The insurance and mortgage scenarios used existing tests or replay fixtures, while the supply-chain scenario used an explicit runtime timeout simulation.