# FinOps plugin

Answers cloud cost questions on AWS, GCP, and Kubernetes. It never changes anything.

```text
/finops why did our AWS bill go up in October?
Reconcile our October GCP invoice with the BigQuery billing export.
Where can we save money on the search node pool, and is it safe?
Run the monthly cost report again, and check whether last month's savings landed.
Compare Kubecost's namespace costs with what we actually pay for the cluster.
```

It detects which billing data exists, asks once for a query budget, reconciles to the invoice, and saves
a report under `docs/finops/`. Every number in the report is read from a saved query result, calculated
by a script from such numbers, or marked as provided by a person, and a script proves this before the
report is written. That guards against mistakes. It can't prove a query asked the right question, or that
a cited cause is the true cause.

- It needs `python3` 3.9 or newer. Without it, the skill stops after discovery.
- GCP spend needs a billing export in BigQuery, and without one the report says so rather than estimating.
- Reports hold real spend figures, so add `docs/finops/` to your `.gitignore`. The skill warns if it isn't.
- Forecasting isn't included.

It hasn't been tried on real accounts yet. Its tests use stubbed CLIs and synthetic data, with its SQL run
in DuckDB, which proves query logic but not provider schemas. Treat the first live run as a trial.

## Installation

```text
/plugin install finops --marketplace glapsfun/finOps
```

This works in one step on Claude Code 2.1.275 or later. On older versions, run
`/plugin marketplace add glapsfun/finOps`, then `/plugin install finops@finops`.

## What's inside

| Component | Purpose |
| :--- | :--- |
| `commands/finops.md` | The `/finops <question>` entry point (Claude Code) |
| `skills/finops/SKILL.md` | The workflow: scope, discovery, budget approval, evidence, assurance, math, and report |
| `skills/finops/references/` | AWS and GCP billing, Kubernetes cost, assurance, calculations, report fields, telemetry, troubleshooting, and which facts are confirmed |
| `skills/finops/sql/` | Prepared billing queries for CUR 2.0, legacy CUR, and the GCP export, exercised on fixtures in DuckDB |
| `skills/finops/scripts/` | `finops-discovery.sh` (no billable calls), `finops_aws.py`, `finops_gcp.py`, `finops_k8s.py`, and `finops_capture.py` (evidence), `finops_calc.py` (decimal math), `finops_record.py` and `finops_guard.py` (run record, read-only checks, budget), `finops_compare.py` (last report, past savings), `finops_validate.py` and `finops_report.py` (proof, then report) |

## Safety model

Runs are advisory. They never buy commitments, resize anything, enable billing exports, or change IAM, and
their queries run only under a budget you approve. Tags, labels, resource names, and query output are treated
as data, never as instructions. Spend figures stay in the run directory and the conversation.

## Degraded environments

Missing tooling or data becomes a gap in the report, not a guess. The
[skill](skills/finops/SKILL.md#degraded-environments) lists what happens for each one.
