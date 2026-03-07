# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Python-based GitHub profile README generator that creates 4 animated SVG banners from `config.yml` + live GitHub stats. SVGs are auto-regenerated every 12h via GitHub Action.

```
config.yml → generator/ → assets/generated/*.svg → README.md displays them
```

## Commands

```bash
# Generate SVGs (fetches live GitHub stats via REST/GraphQL)
python -m generator.main

# Generate with demo data (no API calls)
python -m generator.main --demo

# Interactive setup wizard (creates config.yml)
python -m generator.main init

# Run tests
pip install -r requirements-dev.txt
pytest

# Run specific test file or test
pytest tests/test_config.py -v
pytest tests/test_utils.py::TestFormatNumber::test_1234
```

## Architecture

**Pipeline:** `config.yml` → `main.py` (loads config, fetches stats) → `SVGBuilder` → 4 template renderers → `assets/generated/`

| File | Role |
|------|------|
| `generator/main.py` | CLI entry point, orchestrates load → fetch → render → write |
| `generator/config.py` | YAML validation, required fields, theme defaults |
| `generator/github_api.py` | Two-tier: GraphQL (with token) → REST fallback (public only) |
| `generator/svg_builder.py` | Thin orchestrator that passes config to templates |
| `generator/utils.py` | Colors (GitHub linguist), SVG icons, math (spirals, arcs), text helpers |
| `generator/templates/` | 4 SVG renderers (galaxy_header, stats_card, tech_stack, projects_constellation) |

**SVG outputs (all 850px wide):**
- `galaxy-header.svg` (280px) — Animated spiral galaxy with name, tagline, tech labels
- `stats-card.svg` (180px) — GitHub metrics (commits, stars, PRs, issues, repos)
- `tech-stack.svg` (dynamic height) — Language bar chart + rotating radar chart
- `projects-constellation.svg` (220px) — Featured project cards with connection lines

## Config Schema (`config.yml`)

Required: `username`, `profile.name`, `galaxy_arms` (1-3 arms with name, color, items).
Optional: `social`, `projects` (max 3), `theme` (9 hex overrides), `stats.metrics`, `languages.exclude`.

Arm colors must be one of: `synapse_cyan`, `dendrite_violet`, `axon_amber`.

## Key Patterns

**Deterministic rendering:** Star positions/sizes are seeded by `hashlib.md5(username + index)` — same config always produces identical SVG layout (only stats values change).

**API fallback chain:** GraphQL (accurate, needs token) → REST (public data only) → generation still succeeds with zeros. The GitHub Action provides `GITHUB_TOKEN` automatically.

**Theme resolution:** User overrides in `config.yml` merge with `DEFAULT_THEME` (9 deep-space colors). All templates reference theme dict keys — never hardcoded colors.

**Animation types mixed:** CSS `@keyframes` (twinkling, pulsing), SMIL `<animate>` (opacity), `<animateMotion>` (particles on paths), `<animateTransform>` (rotation). Staggered delays prevent sync flashing.

## GitHub Action

`.github/workflows/generate-profile.yml` — Triggers on: schedule (every 12h), push to `config.yml` or `generator/**`, manual dispatch. Generates SVGs with `GITHUB_TOKEN` and auto-commits to `assets/generated/` with `[skip ci]`.

## Git Setup

This repo lives inside a monorepo at `packages/imdouglasoliveira/` but has its own `.git`. Push via SSH alias:

```bash
git push origin main   # remote: git@github-imdouglas:imdouglasoliveira/imdouglasoliveira.git
```
