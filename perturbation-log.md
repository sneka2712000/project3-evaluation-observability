# Perturbation Log

This log records one deliberate input or configuration change for each system. Each experiment includes the change, prediction, command, actual result, and contrast with the unperturbed behavior.

---

## System 1 — Validated, Routed Insurance Policy Pipeline

### Deliberate Change

Changed the recorded test response for policy `POL-2025-009` by changing the required `endorsements` field from:

`None`

to:

`[{"name": "Schedule A"}]`

This supplied a previously missing required source field.

### Prediction

I predicted that the missing-source escalation would no longer occur because the required `endorsements` field would be present and valid. The extraction should therefore complete without a retry.

### Unperturbed Command

`.venv/bin/pytest tests/test_us01_retry.py::test_ac_01_04_missing_source_halts_immediately -v`

### Unperturbed Result

`1 passed in 0.08s`

The missing `endorsements` source produced `RetryFutileEscalation`. The recorded client made one client call.

### Perturbed Result

After changing `endorsements` to `[{"name": "Schedule A"}]`, the extraction returned `PolicyExtraction` with the endorsement present, `retry_count=0`, and `final_attempt_index=0`.

### Contrast

Without the required source field, the system stopped with `RetryFutileEscalation`. After supplying the missing field, extraction succeeded without retrying. This demonstrates the system's validation boundary for missing required information.

---

## System 2 — Schema-Enforced Mortgage Extraction

### Deliberate Change

Changed the recorded extraction's `stated_monthly_total` from `10892.17` to `9642.17`, matching the calculated sum of the extracted income components.

A backup of the original recorded response was created before the change and the original response was restored afterward.

### Prediction

I predicted that the independent arithmetic consistency validator would stop reporting a discrepancy because the stated total would equal the calculated total.

### Unperturbed Result

The original recorded response produced:

- Calculated total: `9642.17`
- Stated total: `10892.17`
- `consistent: false`
- Discrepancy: `delta = -1250.0`

### Perturbed Result

After changing the stated total:

- Calculated total: `9642.17`
- Stated total: `9642.17`
- `consistent: true`
- `discrepancies: []`

### Contrast

The unperturbed extraction exposed a $1,250 discrepancy between the stated total and the sum of the extracted line items. After the deliberate change made the stated total equal the calculated sum, the independent validator reported the result as consistent. The original recorded response was restored after the experiment.

---

## System 3 — Multi-Source Supply Chain Synthesis

### Deliberate Change

Enabled the `--simulate-timeout` configuration to make the logistics source unavailable during the investigation.

### Prediction

I predicted that the investigation would continue instead of aborting, while information dependent on the unavailable logistics source would be explicitly marked as incomplete.

### Unperturbed Command

`.venv/bin/python -m supply_chain_risk meridian --offline --supplier-name "Meridian Components"`

### Unperturbed Result

The normal investigation used the logistics source and preserved an `on_time_delivery_rate` conflict under **Contested**:

- `95.0 percent` from `supplier_audit`
- `78.0 percent` from `logistics`

### Perturbed Command

`.venv/bin/python -m supply_chain_risk meridian --offline --simulate-timeout --supplier-name "Meridian Components"`

### Perturbed Result

The run completed and reported:

`Sources unavailable: logistics unavailable (timeout)`

The `late_shipment_count` metric was marked **Incomplete** with:

`missing source: timeout reading logistics`

### Contrast

Without the timeout, the logistics source contributed evidence to the synthesis and exposed a conflicting metric. With the timeout, the unavailable source was explicitly annotated and dependent information was marked incomplete, while the investigation continued using the remaining sources. This demonstrates graceful degradation under partial source failure.