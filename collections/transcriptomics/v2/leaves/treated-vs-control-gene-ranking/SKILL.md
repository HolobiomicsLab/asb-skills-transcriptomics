---
name: treated-vs-control-gene-ranking
description: Use when you have Cyclum-fitted pseudo-time for matched treated and control
  single-cell datasets and want to nominate target genes by differential circular
  expression patterns.
when_to_use_negative:
- Data are not single-cell expression matrices with an expected periodic (cell-cycle-like)
  process — Cyclum relies on circular manifolds rather than marker genes
- Only one condition is available; the skill requires fitting separate treated and
  control models to compute a per-gene difference
- The goal is cell-cycle effect removal or pseudotime refinement rather than differential
  circular-pattern gene nomination
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Cyclum
  role: Sinusoidal autoencoder fitted per condition to infer circular pseudo-time;
    cyclum.evaluation quantifies per-gene circular-pattern magnitude
  repo: github.com/KChen-lab/cyclum
- name: cyclum.tuning.CyclumAutoTune
  role: Automatically selects the proper number of linear components to find the 'most
    circular' manifold before magnitude evaluation
  repo: github.com/KChen-lab/cyclum
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:fe02933d0b043137cf105b852c19188472ee8486051613e3d6b7241f59dbac55
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - cyclum.tuning.CyclumAutoTune
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# treated-vs-control-gene-ranking

## Summary

Ranks genes by the treated-minus-control difference in circular-pattern magnitude inferred by Cyclum, to nominate genes (e.g., LDHA, ENO1) whose periodic expression kinetics change between drug-treated and control scRNA-seq samples.

## When to use

Use when you have Cyclum-fitted pseudo-time for matched treated and control single-cell datasets and want to nominate target genes by differential circular expression patterns.

## When NOT to use

- Data are not single-cell expression matrices with an expected periodic (cell-cycle-like) process — Cyclum relies on circular manifolds rather than marker genes
- Only one condition is available; the skill requires fitting separate treated and control models to compute a per-gene difference
- The goal is cell-cycle effect removal or pseudotime refinement rather than differential circular-pattern gene nomination

## Inputs

- Nicotine-treated and control 10X Chromium scRNA-seq count matrices (e.g., hESC, 12,280 cells x 33,694 genes)
- Per-condition Cyclum models fitted via cyclum.models after log2(x+1), centering (mean), and scaling to variance 1

## Outputs

- Per-gene circular-pattern magnitude values for treated and control conditions (from cyclum.evaluation)
- Sorted treated-vs-control difference table, e.g., gene_circular_magnitude_ranking.csv, with nominated targets at top

## How to apply

Load the nicotine-treated and control 10X Chromium hESC count matrices (12,280 cells) and transform each with log2(x+1), centering and scaling to variance 1 as recommended in the Cyclum FAQ. Fit the Cyclum sinusoidal autoencoder (cyclum.models, optionally tuned with cyclum.tuning.CyclumAutoTune for the number of linear components) separately to the treated and control matrices to infer circular pseudo-time. Using cyclum.evaluation, quantify each gene's circular-pattern magnitude — the degree of match between its expression kinetics and the inferred periodicity — in each condition. Compute the treated-minus-control difference per gene, sort genes in descending order, and write the ranked table (e.g., gene_circular_magnitude_ranking.csv). Rationale: Cyclum identifies genes associated with the periodic process based on degrees of match between gene-expression kinetics and the inferred periodicity, so differential magnitude directly flags condition-specific circular behavior.

## Related tools

- **Cyclum** (Sinusoidal autoencoder fitted per condition to infer circular pseudo-time; cyclum.evaluation quantifies per-gene circular-pattern magnitude) — github.com/KChen-lab/cyclum
- **cyclum.tuning.CyclumAutoTune** (Automatically selects the proper number of linear components to find the 'most circular' manifold before magnitude evaluation) — github.com/KChen-lab/cyclum

## Examples

```
from cyclum.models import AutoEncoder; m = AutoEncoder(x.shape[1], linear_dimes=5); pseudotime = m.train(x, epochs=300).infer_pseudotime(x)
```

## Evaluation signals

- LDHA and ENO1 appear among the top-ranked genes by treated-vs-control circular-pattern magnitude difference, reproducing Supplementary Fig. 8b
- Both conditions were preprocessed with log2(x+1), centered and scaled to variance 1 before fitting
- A separate Cyclum model was fitted for each condition, and each gene received a magnitude value in both conditions
- The ranking is sorted in descending order of treated-minus-control magnitude difference and written to gene_circular_magnitude_ranking.csv

## Limitations

- Cyclum is a model-based approach fitting predefined circular manifolds; it assumes a genuine circular trajectory in the data
- Nomination depends on stable per-condition pseudo-time inference; noisy fits in either condition distort magnitude differences
- Runtime scales with data size, though Cyclum can analyze 12,280 cells x 33,694 genes in about 40 min on a GTX 960M GPU

## Evidence

- [intro] Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity.: "Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity."
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (log2(x+1)) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum can analyze an scRNA-seq dataset consisting of 12,280 cells 33,694 genes in 40 min on a laptop computer with a GTX 960M GPU and 2 GB graphic memory: "Cyclum can analyze an scRNA-seq dataset consisting of 12,280 cells 33,694 genes in 40 min on a laptop computer with a GTX 960M GPU and 2 GB graphic memory"
- [readme] we also implemented class cyclum.tuning.CyclumAutoTune, which automatically select the proper number of linear components to help find the "most circular" manifold.: [evidence span withheld — anchor status: external_doc]
