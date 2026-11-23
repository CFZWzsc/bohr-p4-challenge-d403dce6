# Runtime contract

Bohrium Playground Period 4 submission — challenge `challenge-d403dce6` (track: `academic-writing`).

This repository holds the complete skill package for the challenge: the skill
definition, its executable entry point and all supporting scripts. No precomputed
answers, private data or credentials are included.

## Skill

- Name: `grant-proposal-alignment`
- Summary: Use when a case supplies grant-proposal materials (rationale, scientific question, objectives, research contents, technical route, innovation claims, preliminary work) and asks to draft or audit the proposal; builds an objective-content-indicator alignment matrix, flags unverifiable superlatives and uncited strong claims, separates completed from planned work, and writes /app/output/result.json preserving case_id and every output_contract field. Deterministic standard library only; no network.

## Contents

| File |
|------|
| `SKILL.md` |
| `scripts/run_skill.py` |
| `README.md` |

## How to run

```bash
python scripts/run_skill.py --case /input/case.json --output /app/output/result.json
```

The entry point is `SKILL.md`; input/output contracts, error handling and a minimal
usage example are documented there.

## Notes

- Pure Python standard library; no network access required.
- The result JSON keeps the input case id, the main results, per-item evidence,
  a run status and an uncertainty section.
