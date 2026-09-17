Transparent static-analysis scoring for software sustainability, grounded in the Green Software Foundation SCI specification.
green-software
sustainability
software-carbon-intensity
carbon-aware-computing
static-analysis
python
developer-tools
cloud-efficiency
https://sci.greensoftware.foundation/

# sustainability-score

A static-analysis tool that scores a code repository's **sustainability posture**
across five weighted pillars, grounded in the Green Software Foundation
[Software Carbon Intensity (SCI)](https://sci.greensoftware.foundation/)
specification. Think "code-quality scanner, but for carbon and cost efficiency."

It reads only what is in the repository (source, Dockerfiles, CI workflows,
Terraform, Kubernetes manifests, dependency files). It never runs the code and
never measures live infrastructure, so it is **honest about confidence**: every
finding is stamped with a data-quality tier, and the report says plainly that
the score is directional, not a measurement.

## Scope

This is an early **prototype / proof-of-concept**. It is deliberately narrow:
one working end-to-end path from a repo to a scored, dual-format report. It is
advisory only. It does not gate builds, and every recommendation that could
affect capacity or resiliency states the SLO trade-off explicitly.

**In scope:** static repo/PR-level scoring across the five pillars below.
**Out of scope:** runtime measurement, carbon accounting/reporting, live
telemetry, and any commercialization/SaaS concerns.

## The five pillars

| Pillar | Weight | SCI terms it moves |
|---|---:|---|
| Code / Algorithm Efficiency | 30% | E, R |
| Cloud Infrastructure Choices | 25% | E, I, M |
| Containerization | 15% | E, M |
| CI/CD Practices | 15% | E |
| SRE / Operations | 15% | E, R |

`SCI = ((E x I) + M) / R` — a static scan cannot measure E, I, M or R, but it
can assess the engineering choices that push each term up or down.

## Data-quality tiers

Every finding declares the highest tier its evidence supports:

1. **Static analysis only** — directional; inferred from source and config.
2. **Static + declared infra** — corroborated by IaC (region, instance types).
3. **Static + operational telemetry** — backed by observability data.
4. **Direct measurement** — energy/carbon measured; full SCI computable.

A static-only scan never claims above Tier 1 on its own; Tier 2 is reached only
when the repo declares its own infrastructure.

## Install

```bash
pip install -e .
```

## Usage

```bash
# Print a Markdown report to stdout
sustainability-score /path/to/repo

# Write both formats to files
sustainability-score /path/to/repo --json report.json --md report.md
```

Or from Python:

```python
from sustainability_score import scan, to_json, to_markdown
result = scan("/path/to/repo")
print(result.composite_score, result.grade)
```

## Example

Run against the bundled fixture (a deliberately wasteful sample service):

```bash
sustainability-score tests/fixtures/sample_service --md docs/sample-report.md
```

A rendered sample report lives at [`docs/sample-report.md`](docs/sample-report.md).

## How scoring works

Each applicable pillar starts at 100 and loses points per finding by severity
(high 20 / medium 10 / low 4 / info 0), floored at 0. The composite is the
weighted mean over **applicable** pillars only — a pillar with no relevant
artifacts (e.g. no Dockerfile) is marked *not applicable* and its weight is
renormalized away rather than counted as a free 100.

## Tests

```bash
python -m pytest
```

## License

MIT. See [LICENSE](LICENSE).

## Status & roadmap

This is a v0.1 prototype. Near-term direction: broaden per-language efficiency
checks, add a JSON schema for the output, wire an optional GitHub Action that
posts the advisory PR comment, and (with declared infra + telemetry) climb from
Tier 1 toward Tier 3 confidence. It is intended as the upstream/canonical
implementation, with a version proposed to the GSF reference-implementations
collection referencing this repo.
