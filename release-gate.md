# Release Gate

## Required checks

1. Unit tests pass (`make test-unit`).
2. Integration smoke passes (`make test-integration-smoke`).
3. Shell syntax check passes (`bash -n`).
4. ShellCheck warnings do not increase over agreed baseline.
5. Optional full integration suite passes (`make test-integration-full`).
6. Optional perf regression check (`make test-release-gate`) stays within threshold.

## Commands

```bash
make test-unit
make test-integration-smoke
make test-integration-full
make test-release-gate
```

## CI mapping

`tests.yml` runs:

- `shellcheck` on push/PR.
- `unit-fast` on push/PR.
- `integration-smoke` on push/PR (after unit-fast).
- `integration-full` on schedule or explicit workflow dispatch.

Local runner modes are available through `tests/run_tests.sh`:

- `--unit`
- `--smoke`
- `--integration`
- `--all`

## Troubleshooting

- If integration smoke fails, inspect artifacts from `report/` and `.incremental/history/latest/`.
- If perf gate fails, compare `tests/bench/baseline_metrics.json` with `.log/perf_summary.json` and update baseline only with justified changes.
