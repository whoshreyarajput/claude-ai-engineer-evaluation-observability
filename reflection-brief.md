# Reflection Brief — Evaluation and Observability Capstone

**Name:** Ayush
**Date:** 2026-11-12

## 0. Environment
| Field | Value |
|---|---|
| OS & version | Udacity Linux Workspace (`evidence/environment.txt`) |
| Python version | Python 3.13.0 |
| Date run | Wed Nov 11 2026 (perturbation runs 2026-11-12) |
| Ran any system live? (which) | System 1 (policy pipeline via claude.vocareum.com, `pipeline.txt`); Systems 2 and 3 in replay/offline |

## 1. Validated, routed pipeline
| Evidence | Value |
|---|---|
| Passing test count | 45 passed, 3 skipped (`01-policy/tests.txt`) |
| Routing output file | `evidence/01-policy/routing_decisions.json` |
| auto_approve / human_review / spot_check counts | 0 / 8 / 1 (1 escalation) |

**1a. Retry boundary.** With the premium removed from POL-2025-003, `perturbation-live.txt` shows one `POST` (1 API call), then `validation_failed`, and its summary JSON records `"escalations": 1`, `premium_amount_absent`, `missing_source`. The data is not in the document, so retrying would pay for identical failures and could tempt the model to invent a value; escalating immediately is cheaper and surfaces the real problem.

**1b. Reading the router.** POL-2025-001 (`routing_decisions.json`): confidences 0.90-0.99 and `fields_below_threshold: []`, `integration_failures: []`, but `reason: reviewer_disagreement=['coverage_limit', 'deductible']`. The reviewer signal sent it to a human. Trusting confidence alone would have auto-approved a policy where the two independent reviewers disagreed on coverage limit and deductible.

**1c. Where the aggregate lies.** `calibration.txt`: `umbrella exclusions n=2 conf=0.93 acc=0.00 brier=0.865`, against `OVERALL brier=0.291`. A single overall number averages the umbrella exclusions failure with the accurate auto premium cell (conf 0.95, acc 1.00); slicing by policy_type × field shows exactly where confidence is unearned.

## 2. Schema-enforced two-pass extraction
| Evidence | Value |
|---|---|
| Passing test count | 25 passed (`02-mortgage/tests.txt`) |
| Document run | `fixtures/documents/appraisal_informal_sqft.txt` (`02-mortgage/appraisal.txt`) |
| Classified type | `classify: model=claude-haiku-4-5-20251001 in=2169 out=108 type=appraisal` |

**2a. Two guarantees.** `income-mismatch.txt`: `"consistent": false`, `calculated: 9642.17`, `stated: 10892.17`, `delta: -1250.0`. Tool use guarantees the output is valid JSON matching the schema; the validator checks the arithmetic meaning of the values. Tool use cannot catch a well-formed but wrong number (e.g. a misread stated total). The validator cannot catch an error that keeps the sums consistent, or a field that is absent. Perturbation: raising tolerance to 2000.00 made the same -1250.00 gap pass (`tolerance-perturbation.txt`, `consistent: true`).

**2b. Refusing to fabricate.** `missing-bonus.txt` lines 15-16: `"bonus_monthly": null`, `"bonus_ytd": null`. `models.py` declares these as `float | None = None`, so null is valid, and rule 1 of `_CATEGORICAL_CRITERIA` in `prompts.py` says "Return null for any field not explicitly stated in the document. Do not infer, default, or fabricate." `Income.calculated_monthly_total` returns `None` rather than `0.0`, so absence is not treated as zero.

**2c. Normalization.** Source line 37 of `appraisal_informal_sqft.txt`: "approximately 2,400 sq ft" → `"gross_living_area_sqft": 2400` (`appraisal.txt`). Normalizing at extraction time (rules in `NORMALIZATION_RULES`) means every downstream consumer and the validator receive typed numbers, not strings each must parse differently.

## 3. Multi-source synthesis
| Evidence | Value |
|---|---|
| Passing test count | 34 passed (`03-supply-chain/tests.txt`) |
| Briefing file | `evidence/03-supply-chain/meridian-offline.txt` |
| Section the conflict landed in | Contested (`on_time_delivery_rate`, ESCALATE) |

**3a. Annotate, don't arbitrate.** `meridian-offline.txt`: "95.0 percent — supplier_audit (as of 2026-04-10)" vs "78.0 percent — logistics (as of 2026-04-05)". A reader who sees both knows the supplier's self-audit is far more optimistic than logistics data and can investigate; an average (~86.5%) would match neither source and hide the dispute.

**3b. Source goes dark.** `meridian-timeout.txt` line 5: "Sources unavailable: logistics unavailable (timeout)"; `late_shipment_count` is under Incomplete ("missing source: timeout reading logistics"). Unreachable is recorded as missing/incomplete, not as zero or "nothing to report"; the other sources keep processing, so the run finishes with a partial briefing. Baseline had `late_shipment_count` = 11.0 shipments.

**3c. Dates as a guardrail.** Industry news is dated: the recall "(industry_news, 2026-02-19)" and the port strike "(industry_news, 2026-03-17)", and the contested metric values carry "as of" dates (2026-04-10 vs 2026-04-05). Requiring the date lets the reader see whether two claims are simultaneous or simply measured at different times, so a time difference is not read as a contradiction.

## 4. Synthesis
**4a. One principle.** Across all three systems, reliable automation requires more than valid output: it must validate meaning, preserve uncertainty and source provenance, and escalate when evidence is insufficient. The clearest moment: System 2, `income-mismatch.txt`: valid JSON from the model, yet the validator reported `delta: -1250.0`. A design trusting well-formed output would have shipped the wrong income total.

**4b. Confidence ≠ correctness.** System 1: `calibration.txt` shows `umbrella exclusions conf=0.93 acc=0.00`, and POL-2025-001 had confidences 0.90-0.99 yet reviewers disagreed. High stated confidence coincided with wrong or disputed answers.

**4c. Apply it.** Workflow: extracting line items and totals from supplier invoice PDFs. I would use validated retry with escalation first: validate that line items sum to the stated total, retry only when the failure is fixable, and escalate missing-source cases immediately (as in 1a). I would instrument the validation-failure rate per field, retry counts, escalation count by reason, and calibration (confidence vs accuracy) per vendor × field, alerting when a slice drifts.
