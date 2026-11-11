# Reflection Brief — Evaluation and Observability

## 1. Insurance Policy Extraction Pipeline

The insurance pipeline demonstrated the importance of combining extraction confidence, validation, and human review.

The live run processed **nine decisions** and identified missing source evidence and a premium consistency issue.

An important observation was that high extraction confidence does not automatically mean a decision should be approved. Reviewer disagreement can independently justify human review.

The calibration output showed an overall Brier score of **0.291**. However, the sample was small, so this should be treated as illustrative rather than representative of production performance.

**Verification:**
- Tests: 45 passed, 3 skipped.
- MyPy: No issues in 11 source files.
- Ruff: All checks passed.
- Live pipeline: Routing decisions generated successfully.

---

## 2. Mortgage Document Extraction System

The mortgage system demonstrated the importance of mathematical consistency checks and preserving missing values.

The income mismatch scenario calculated monthly income as **$9,642.17**, while the document stated **$10,892.17**. The validator detected a **$1,250 discrepancy** and marked the result inconsistent.

The missing-bonus scenario preserved missing values as `null` instead of interpreting them as zero. This prevents incomplete documentation from being mistaken for a confirmed financial value.

**Verification:**
- Tests: 25 passed.
- MyPy: No issues in 11 source files.
- Ruff: All checks passed.
- Replay scenarios: Completed successfully.

---

## 3. Supply Chain Risk Investigation

The supply-chain coordinator demonstrated resilience when one source became unavailable.

During the normal investigation, the system identified:
- Corroborated findings.
- Conflicting delivery-rate evidence.
- Ambiguous supplier identity.
- A metric with no available source.

During the simulated logistics timeout, the coordinator continued processing other sources. It preserved partial results and marked late shipment count as incomplete.

This showed why source availability, evidence provenance, and uncertainty should be represented separately. A failed source should not be interpreted as an empty result or cause the entire investigation to fail.

**Verification:**
- Tests: 34 passed.
- MyPy: No issues in 8 source files.
- Ruff: All checks passed.
- Normal offline and simulated-timeout investigations: Completed successfully.

---

## 4. Overall Learnings

1. Confidence alone is insufficient for automated decisions; validation and human review are also important.
2. Missing values should remain distinguishable from actual zero values.
3. Conflicting evidence should be preserved and surfaced instead of silently resolved.
4. Partial results are valuable when a source becomes unavailable.
5. Provenance helps identify where a claim came from.
6. Automated tests, type checking, and linting provide complementary forms of verification.

---

## 5. Limitations

- The insurance calibration sample was small and should not be treated as production-level statistical validation.
- The mortgage mismatch and missing-bonus scenarios used existing replay fixtures rather than manually modified documents.
- The supply-chain timeout was explicitly simulated rather than caused by a real external outage.

These limitations are important when interpreting the evidence and assessing how the systems might behave in production.
