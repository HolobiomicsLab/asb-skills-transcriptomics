---
name: reference-table-metric-comparison
description: Use when you have recomputed clustering/integration metrics (ARI, kBET,
  ASW, ASW-beta) for a batch-effect benchmark and need to verify them against the
  paper's published reference table before trusting the reproduction.
when_to_use_negative:
- The input is a raw count matrix or cell embeddings with no recomputed ARI/kBET/ASW/ASW-beta
  values yet — run the evaluate workflow first.
- The benchmark uses a different dataset or cell-removal design than the Human Pancreas
  beta-cell-removal variant (Supplementary Table 4), making the Supplementary Table
  5 reference values inapplicable.
- Only one metric is available (e.g. ARI alone); the comparison relies on the joint
  pattern across ARI, kBET, ASW, and ASW-beta to detect overcorrection.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Method whose reproduced ARI/kBET/ASW/ASW-beta values are compared against
    Supplementary Table 5 reference values
  repo: github.com/hui2000ji/scETM
- name: Harmony
  role: Competing integration method whose metrics are tabulated in the same comparison
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method whose metrics are tabulated in the same comparison
  repo: https://github.com/brianhie/scanorama
- name: LIGER
  role: Aggressive-corrector baseline whose reproduced metrics (ARI 0.8476, kBET 0.5978)
    are checked against the reference table
  repo: https://github.com/welch-lab/liger
- name: scVI
  role: Competing integration method whose metrics are tabulated in the same comparison
  repo: https://github.com/YosefLab/scvi-tools
- name: scVAE-GM
  role: Competing integration method whose metrics are tabulated in the same comparison
  repo: https://github.com/scvae/scvae
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:ad64fc41b05ade00c75b8a84a0e2f12409a83f7b032e0db50f5c2090255f7203
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - scETM
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# reference-table-metric-comparison

## Summary

Validates a reproduced single-cell integration benchmark by comparing freshly computed metrics (ARI, kBET, ASW, ASW-beta) against the published reference values from Supplementary Table 5 of the scETM paper, exposing deviations that indicate pipeline drift.

## When to use

Use when you have recomputed clustering/integration metrics (ARI, kBET, ASW, ASW-beta) for a batch-effect benchmark and need to verify them against the paper's published reference table before trusting the reproduction.

## When NOT to use

- The input is a raw count matrix or cell embeddings with no recomputed ARI/kBET/ASW/ASW-beta values yet — run the evaluate workflow first.
- The benchmark uses a different dataset or cell-removal design than the Human Pancreas beta-cell-removal variant (Supplementary Table 4), making the Supplementary Table 5 reference values inapplicable.
- Only one metric is available (e.g. ARI alone); the comparison relies on the joint pattern across ARI, kBET, ASW, and ASW-beta to detect overcorrection.

## Inputs

- Computed metric table (per method: ARI, kBET, ASW, ASW-beta) from the evaluate workflow on the beta-cell-removed Human Pancreas AnnData
- Reference values from Supplementary Table 5 (e.g. scETM ARI 0.9298, kBET 0.1247, ASW 0.3525, ASW-beta 0.5370; LIGER ARI 0.8476, kBET 0.5978)
- Method names for the seven benchmarked integrators (scETM, scETM+adv, Harmony, Scanorama, Seurat, LIGER, scVI, scVAE-GM)

## Outputs

- benchmark_metrics.csv containing computed vs. reference metric values side by side for every method
- Per-method deviation assessment identifying which methods reproduce the overcorrection finding (LIGER, scETM+adv) and which reproduce the balance (scETM)

## How to apply

Recompute each integration metric on the same cell embeddings the benchmarked methods produced (e.g. scETM cell embeddings stored in obsm['delta'] on the beta-cell-removed Human Pancreas variant). Assemble the reference values from the paper's Supplementary Table 5 (scETM: ARI 0.9298, kBET 0.1247, ASW 0.3525, ASW-beta 0.5370; LIGER: ARI 0.8476, kBET 0.5978) alongside your computed values per method. Join the two tables by method name and metric into a single benchmark_metrics.csv so deltas are inspectable column-by-column. Treat a method as faithfully reproduced when its metric pattern matches the qualitative finding — scETM balancing cell-type discrimination (high ARI/ASW) against batch mixing (low kBET), while aggressive correctors like scETM+adv and LIGER compromise ASW-beta. Large deviations on one metric (especially ASW-beta, which is sensitive to the removed beta-cell structure) indicate a data-preparation or training-parameter mismatch rather than metric-code error, so re-check the beta-cell removal recipe (Supplementary Table 4) before tuning.

## Related tools

- **scETM** (Method whose reproduced ARI/kBET/ASW/ASW-beta values are compared against Supplementary Table 5 reference values) — github.com/hui2000ji/scETM
- **Harmony** (Competing integration method whose metrics are tabulated in the same comparison) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method whose metrics are tabulated in the same comparison) — https://github.com/brianhie/scanorama
- **LIGER** (Aggressive-corrector baseline whose reproduced metrics (ARI 0.8476, kBET 0.5978) are checked against the reference table) — https://github.com/welch-lab/liger
- **scVI** (Competing integration method whose metrics are tabulated in the same comparison) — https://github.com/YosefLab/scvi-tools
- **scVAE-GM** (Competing integration method whose metrics are tabulated in the same comparison) — https://github.com/scvae/scvae

## Examples

```
import pandas as pd; ref = {'method':['scETM','LIGER'],'ARI':[0.9298,0.8476],'kBET':[0.1247,0.5978],'ASW':[0.3525,None],'ASW_beta':[0.5370,None]}; comp = computed.merge(pd.DataFrame(ref), on='method', suffixes=('_computed','_reference')); comp.to_csv('benchmark_metrics.csv', index=False)
```

## Evaluation signals

- Computed scETM ARI falls near the reference 0.9298 (within reproduction tolerance) while kBET stays low (~0.1247)
- Computed scETM ASW-beta is close to 0.5370, confirming beta-cell structure is preserved rather than over-corrected
- LIGER's computed kBET is elevated (reference 0.5978) with reduced ARI (reference 0.8476), reproducing the overcorrection pattern
- benchmark_metrics.csv contains one row per method (scETM, scETM+adv, Harmony, Scanorama, Seurat, LIGER, scVI, scVAE-GM) with all four metrics populated
- Relative ordering of methods by each metric matches the Supplementary Table 5 ordering, even if absolute values shift slightly

## Limitations

- Reference values are dataset- and split-specific: they apply only to the Human Pancreas batch with beta cells removed from all batches except CelSeq2 (Supplementary Table 4)
- Small metric drifts can stem from stochastic training (scETM uses batched gradient descent with n_samplers=4); convergence issues arise if training is shorter than ~6k gradient updates on small datasets
- ASW-beta is the most sensitive indicator of overcorrection of the removed cell-type structure and may diverge even when ARI and kBET match

## Evidence

- [methods] Supplementary Table 5 reference values: [evidence span withheld — anchor status: not_found]
- [methods] LIGER overcorrection reference: [evidence span withheld — anchor status: not_found]
- [readme] Training recipe grounding: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [readme] Convergence requirement: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model"
- [methods] Evaluate workflow grounding: "evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")"
