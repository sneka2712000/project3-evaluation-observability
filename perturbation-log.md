# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):**
- **Command I ran:**
- **What I predicted:**
- **What actually happened (paste the key output line):**
- **How this differs from the unperturbed run:**

---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):**
- **Command I ran:**
- **What I predicted:**
- **What actually happened (paste the key output line):**
- **How this differs from the unperturbed run:**

---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):**
- **Command I ran:**
- **What I predicted:**
- **What actually happened (paste the key output line):**
- **How this differs from the unperturbed run:**

## System 2 — Mortgage Extraction

**Deliberate change:** Changed the recorded extraction's `stated_monthly_total` from `10892.17` to `9642.17`, matching the sum of the extracted income components.

**Backup:** Created `/tmp/mortgage_response_backup.json` before the change and restored the original recorded response afterward.

**Unperturbed result:**
- Calculated total: `9642.17`
- Stated total: `10892.17`
- `consistent: false`
- Discrepancy: `delta = -1250.0`

**Perturbed result:**
- Calculated total: `9642.17`
- Stated total: `9642.17`
- `consistent: true`
- `discrepancies: []`

**Contrast:** The unperturbed recorded extraction exposed a $1,250 discrepancy between the stated total and the sum of the line items. After deliberately changing the stated total to equal the calculated sum, the independent consistency validator no longer reported a discrepancy. The original recorded response was restored after the experiment.


## System 3 — Multi-Source Supply Chain Synthesis

**Deliberate change:** Enabled the `--simulate-timeout` configuration to make the logistics source unavailable during the investigation.

**Unperturbed command:**
```text
.venv/bin/python -m supply_chain_risk meridian --offline --supplier-name "Meridian Components"
**Unperturbed result:** The normal investigation used the logistics source and preserved the `on_time_delivery_rate` conflict under **Contested**, with `95.0 percent` from `supplier_audit` and `78.0 percent` from `logistics`.

**Perturbed command:**
```text
.venv/bin/python -m supply_chain_risk meridian --offline --simulate-timeout --supplier-name "Meridian Components"
```text
**Perturbed result:** With `--simulate-timeout`, the run still completed and reported `Sources unavailable: logistics unavailable (timeout)`. The `late_shipment_count` metric was marked **Incomplete** with `missing source: timeout reading logistics`.

**Contrast:** Without the timeout, logistics contributed evidence to the synthesis. With the timeout, the unavailable source was explicitly annotated instead of being treated as evidence that nothing was reported, while the investigation continued using the remaining sources.

