# accounting/

Per-job accounting for Project1: how we estimate energy, emissions and cost for each job, the per-job log that records the result, and the dashboard that displays it.

**Owner:** accounting, backend and dashboard role.
**Status:** week 1 (Mon Oct 26 to Sun Nov 1, 2026). Everything here is a draft until the interfaces are agreed with the AWS person. Dates are assumptions until the graduation deadline is confirmed.

## What this folder is for

1. **Accounting spec.** The emissions and cost formulas, written as a short document.
2. **Per-job log schema.** One record per job. This is one of the three team interfaces (job spec, scheduler decision, metrics log), so field names and units must be exact and stable. The scheduler/evaluation person reads this log.
3. **Dashboard.** A simple static page (hosted on S3 by the AWS person) showing the per-job log, a carbon-versus-cost chart and savings against the baseline.

## How emissions are estimated

```
compute emissions = modeled energy x PUE x actual grid carbon intensity during the run
```

- Energy is **modeled** from utilization and published power coefficients, not measured. It is always labeled "modeled".
- Intensity is the **actual** value during the run, never the forecast. The forecast is logged separately (`forecast_intensity_at_decision`) for analysis only.
- Cross-region data movement has a **separate transfer-carbon term** with a best-case and a worst-case value.
- Caribou's Lambda power numbers are **not** reused. They do not transfer to EC2.

### Open placeholders (must be verified from a primary source before use)

| Item | Status | Source |
|---|---|---|
| PUE value | OPEN | not yet recorded |
| EC2 power coefficients (per chosen instance type) | OPEN | not yet recorded |
| Transfer energy / carbon best and worst values | OPEN | not yet recorded |
| Instance types (2 to 3 small CPU types in all regions) | to agree with the spot person | n/a |
| Regions (proposed: us-east-1, us-west-1, eu-central-1, eu-north-1) | proposed, not confirmed | n/a |

No value goes into the spec or the code until the source has been opened and recorded here.

## Per-job log (draft)

Conventions, proposed:

- Timestamps in UTC, ISO 8601 with `Z` suffix.
- Energy kWh, intensity gCO2e/kWh, emissions gCO2e, money USD, durations minutes.
- JSON Lines as the canonical format, with the same field names in any CSV export.
- Required fields are never null. Nullable fields state when they may be null.

Fields from the v2 brief:

`job_id`, `type`, `release_time`, `deadline`, `region`, `purchase_option`, `start_time`, `end_time`, `energy_kwh` (modeled), `forecast_intensity_at_decision`, `actual_intensity_during_run`, `emissions_g`, `spot_price`, `cost_usd`, `transfer_cost_usd`, `interruptions`, `delay_min`, `deadline_met`, `decision_reason`, `transfer_emissions_g`, `immediate_start_emissions_g` (counterfactual, computed after the run from traces), `carbon_trap` (true if the run emitted more than starting immediately would have).

Proposed additions, pending agreement with the AWS and scheduler people:

`schema_version`, `run_id`, `trial`, `policy`, `source` (`simulator` or `aws`), `instance_type`, `status`, `pue_used`, `energy_method` (constant `"modeled"`), `on_demand_price`.

Open schema decisions:

- Store transfer emissions as best and worst fields, or one headline value.
- Exact definition of `delay_min` (must be fixed because it feeds the p95 delay metric).
- Whether `spot_price` is a time-weighted average over the run.
- Basis of `carbon_trap` (compute plus transfer, and whether the counterfactual assumes no interruption).

## Claims policy (applies to everything in this folder)

- We say **"estimated emissions of the workloads we schedule"**. We never claim reduced grid or datacenter emissions.
- No savings number is quoted before we measure it.
- Modeled energy is never presented as measured.
- Nothing is cited that we have not opened. Unknowns are marked as placeholders.
- Other papers' figures are loose background only, never our expected results.

## Dependencies

| Who | What we need or provide |
|---|---|
| Spot person | Agreed instance types now; observed interruption data later |
| AWS person | Agree the log format together; they host the static dashboard |
| Data person | Carbon intensity numbers the formula multiplies by |
| Scheduler person | Reads the log for evaluation, so names and units must stay stable |

## Timeline for this role

| Week | Work |
|---|---|
| 1 | Sources (PUE, EC2 coefficients), log schema, emissions formula document, dashboard stack and mock-up with fake data |
| 3 | Simulator writes the real per-job log |
| 7 | Per-job accounting on real AWS runs |
| 8 | Real dashboard |

## Planned contents

```
accounting/
  README.md              this file
  SOURCES.md             verified PUE and coefficient sources (to create)
  accounting_spec.md     formulas incl. transfer term, best and worst case (to create)
  schema/                per-job log schema, versioned (to create)
  dashboard/             static page and mock-up with fake data (to create)
```

Files are added as they are written; nothing above exists yet except this README.
