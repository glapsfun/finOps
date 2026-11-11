# finops

A plugin for Claude Code and Codex, also distributed as a standard Agent Skill, that answers cloud cost
questions on AWS, GCP, and Kubernetes with numbers that hold up.

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

See the [plugin README](plugins/finops/README.md) for what ships and how it stays safe.

## Installation

### Claude Code

```text
/plugin marketplace add glapsfun/finOps
/plugin install finops@finops
```

Or from a shell:

```bash
claude plugin marketplace add glapsfun/finOps
claude plugin install finops@finops
```

### Agent Skill

```bash
npx skills add glapsfun/finOps --skill finops --agent codex --global -y
```

Omit `--global` to install into the current project. The `/finops` command comes only with the Claude Code
plugin. Elsewhere, ask the question and the skill picks it up.

## What it will never do

- Buy commitments, resize anything, delete anything, enable billing exports, or change IAM, even if asked.
- Run a query that creates a job or a result object outside the budget you approved.
- Print or store credentials. It uses ambient credentials and named profiles only.
- Estimate GCP spend from SKU prices when there's no billing export.

## Repository layout

```text
.claude-plugin/marketplace.json   # Claude Code marketplace manifest
.agents/plugins/marketplace.json  # Codex marketplace manifest
plugins/finops/
├── .claude-plugin/plugin.json    # Claude Code plugin manifest
├── .codex-plugin/plugin.json     # Codex plugin manifest
├── commands/finops.md            # /finops slash command
└── skills/finops/
    ├── SKILL.md                  # the cost workflow, safety rules, ledger
    ├── agents/openai.yaml        # Codex interface
    ├── evals/evals.json          # skill evals
    ├── references/               # AWS and GCP billing, Kubernetes cost, assurance,
    │                             #   calculations, report, telemetry, troubleshooting
    ├── sql/                      # prepared billing queries
    └── scripts/                  # discovery, query runners, math, validation, report
scripts/                          # repo tooling and CI checks
tests/                            # script tests, SQL tests, synthetic fixtures
```

## Development

```bash
scripts/check.sh --all       # everything CI runs
scripts/validate.sh --fast   # structure, marketplace sync, manifest versions, JSON, YAML, shell syntax
scripts/lint.sh              # shellcheck, yamllint, markdownlint, actionlint
scripts/test.sh              # eval schema validation, unit tests, SQL tests
scripts/security.sh          # gitleaks secret scan
```

The SQL tests need DuckDB. Without it, a local run prints `SKIP` and proves nothing about the SQL, and on CI
a missing DuckDB is a failure.

## License

[MIT](LICENSE)
