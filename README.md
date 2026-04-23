# Instruction Hierarchy and Conflict Resolution in Multi-Layer Context Stacks

> MLCS-Bench: how a small instruction-tuned language model resolves conflicts between instructions at the system, developer, tool-schema, retrieved-document, and user layers, and whether wrapping each layer in a priority-tagged element (`<layer name=... priority=K>...</layer>`) measurably improves adherence to the highest-priority layer.

## Paper
- **PDF**: [tex/main.pdf](tex/main.pdf)
- **Source**: [tex/main.tex](tex/main.tex) (compile with `tectonic -X compile tex/main.tex`)
- **Peer review**: [review.md](review.md) (auto-review, sealed PDF)

## Primary result

Across three seeds on MLCS-Bench (200 conflict scenarios + 20 non-conflict controls per seed, Qwen2.5-0.5B-Instruct, greedy decoding):

- Explicit layer tagging raises **layer-adherence rate from 0.613 to 0.732**, a mean absolute improvement of **+11.8 ± 3.2 percentage points** over three seeds.
- Helpfulness on non-conflict controls is preserved (0.817 → 0.883).
- Per-layer attack success is sharply asymmetric: tagging cuts developer-layer ASR from 0.307 to 0.093 and retrieved-document ASR from 0.600 to 0.267, but raises user-layer ASR from 0.487 to 0.627.
- Retrieved-document injection is the single most effective attack vector under plain concatenation, at 0.600 success.

## How to reproduce

```bash
# Modal serverless GPU (recommended)
modal run experiments/01-mlcs-bench-unlabeled-vs-tagged/experiment.py

# Or locally (no GPU required; ~15 minutes per seed on CPU)
python experiments/01-mlcs-bench-unlabeled-vs-tagged/experiment.py
```

Per-seed metric files land in `experiments/01-mlcs-bench-unlabeled-vs-tagged/metrics_seed{0,1,2}.jsonl`, and a reconstructable `summary.json` aggregates the three seeds.

## Figures

| Figure | Target section | Description |
| --- | --- | --- |
| [fig-hierarchy.png](figures/fig-hierarchy.png) | method | The five context layers considered in this work, ordered top (highest priority) to bottom (lowest). When layers conflict... |
| [fig-prompt-formats.png](figures/fig-prompt-formats.png) | method | The same five-layer input rendered as (a) UNLABELED, plain concatenation with uppercase name prefixes, and (b) TAGGED, e... |
| [fig-main-bars.png](figures/fig-main-bars.png) | results | Explicit layer tagging raises layer-adherence rate on conflict scenarios from 0.613 to 0.732 and control helpfulness fro... |
| [fig-asr-per-layer.png](figures/fig-asr-per-layer.png) | results | Tagging reduces attack success rate for developer, tool-schema and retrieved-document injections, but raises it for user... |
| [fig-per-seed-delta.png](figures/fig-per-seed-delta.png) | results | Per-seed delta in layer-adherence (TAGGED minus UNLABELED) is +15.0, +13.0, and +7.5 percentage points for seeds 0, 1, 2... |


## Recommended venues

- **EMNLP 2026 Findings** (https://2026.emnlp.org/): Strong.
- **COLM 2027** (https://colmweb.org/): Excellent scope match.
- **NeurIPS 2026 SafeGenAI / Red Teaming workshop** (https://neurips.cc/Conferences/2026/CallForWorkshops): Excellent safe fallback.
- **TMLR** (https://www.jmlr.org/tmlr/): Good.
- **JAIR** (https://www.jair.org/): Moderate.


## Authors

- Vikash Chandra Mishra (`vikash@vizuara.com`)

## Provenance

Session id: `20260423-064720-k3n1`. See [log.md](log.md) and [state.json](state.json) for the full pipeline audit trail.
