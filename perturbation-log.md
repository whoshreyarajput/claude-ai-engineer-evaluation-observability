# Perturbation and Resilience Log

## 1. Insurance Policy Extraction Pipeline
**Change I made:** Copied `data/policies/` to `/tmp/perturbed/`, deleted the premium paragraph ("The total premium for the renewal term ... monthly installments") from `POL-2025-003.txt` (`sed -i '/The total premium/,/in monthly installments/d'`); then copied only the modified `POL-2025-003.txt` into `/tmp/p1/` so the run covered just that policy. Saved as `evidence/01-policy/POL-2025-003-perturbed.txt`.
**Command:** `.venv/bin/policy-extractor pipeline /tmp/p1 --routing-out /tmp/p1_routing.json --spot-check-pct 0.2 --seed 42` (output: `evidence/01-policy/perturbation-live.txt`)
**Observed:** 1 API call, log line `validation_failed`, `decisions_written: 0`, `escalations: 1`, pattern `premium_amount_absent` (category `missing_source`) for POL-2025-003.
**Baseline contrast:** Unperturbed, POL-2025-003 was `human_review` (`reviewer_disagreement=['coverage_limit']`) and the run wrote 9 decisions. The perturbed run escalated with no routing decision and no retry.

## 2. Mortgage Document Extraction System
**Change I made:** Raised `DEFAULT_TOLERANCE_USD` in `mortgage_extractor/config.py` from 1.00 to 2000.00 (backed up, restored afterwards).
**Command:** `.venv/bin/mortgage-extract fixtures/documents/income_sum_mismatch.txt --mode replay --verbose` (output: `evidence/02-mortgage/tolerance-perturbation.txt`)
**Observed:** exit code 0, `"consistent": true`, no discrepancies.
**Baseline contrast:** At tolerance 1.00 (`income-mismatch.txt`) the same fixture gave `"consistent": false`, calculated 9642.17 vs stated 10892.17, delta -1250.0, exit code 1. The -1250.00 gap is unchanged; only the threshold moved, so a loose tolerance hides a real error.

## 3. Supply Chain Risk Investigation
**Change:** Ran with `--simulate-timeout` so the logistics source fails.
**Output:** `evidence/03-supply-chain/meridian-timeout.txt`
**Observed:** "Sources unavailable: logistics unavailable (timeout)"; `late_shipment_count` moved to Incomplete ("missing source: timeout reading logistics"); the run completed.
**Baseline contrast:** In `meridian-offline.txt`, `on_time_delivery_rate` was Contested (95.0 percent supplier_audit vs 78.0 percent logistics, ESCALATE) and `late_shipment_count` was 11.0 shipments from logistics. With logistics down, `on_time_delivery_rate` became "single source only" and `late_shipment_count` became Incomplete.
**Limitation:** The timeout is simulated.

## 4. Overall Learning

The experiments showed that reliable automation requires more than successful execution or well-formed output. Each system must validate the meaning of its results and preserve uncertainty.

- **System 1:** Missing source evidence caused immediate escalation rather than unnecessary retries or fabricated values.
- **System 2:** Increasing the tolerance changed the validation outcome without changing the underlying $1,250 discrepancy. This demonstrated how poorly chosen thresholds can hide real errors.
- **System 3:** A source timeout produced an incomplete result while allowing the remaining sources to continue processing.

Across all three systems, deliberate changes produced observable behavior rather than silent failures. In my own work, I would apply the same principle to document extraction and invoice processing: validate extracted values, retain source provenance, monitor failures, and escalate cases that cannot be resolved safely.
