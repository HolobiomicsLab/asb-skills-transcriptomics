---
name: gene-periodicity-magnitude-quantification
description: Use when you have Cyclum-inferred circular pseudo-time for cells in two
  or more conditions (e.
when_to_use_negative:
- You only need cell-cycle phase labels or pseudo-time itself, not gene-level periodicity
  scores — the Cyclum model fitting suffices without the evaluation step
- The data lack a circular/periodic structure (e.g., linear differentiation trajectories);
  Cyclum projects data onto a predefined circular manifold and is designed for periodic
  processes
- You intend to use marker-gene-based cell-cycle scoring (e.g., Seurat::CellCycleScoring,
  scanpy.tl.score_genes_cell_cycle) as the primary quantification — Cyclum explicitly
  relies on the circular manifold instead of marker genes
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that infers circular pseudo-time and provides cyclum.evaluation
    to quantify per-gene circular-pattern magnitude
  repo: github.com/KChen-lab/cyclum
- name: cyclum.tuning.CyclumAutoTune
  role: Automatically selects the proper number of linear components to find the 'most
    circular' manifold before evaluation
  repo: github.com/KChen-lab/cyclum
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:b2d8e21b56dc1a2d1e5d5214093776fc17ce8e0a26645a6c020b005d32c26d7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - cyclum.tuning.CyclumAutoTune
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-periodicity-magnitude-quantification

## Summary

Quantify each gene's circular-pattern magnitude (degree of match between its expression kinetics and an inferred periodic trajectory) with Cyclum's evaluation module, and compute condition-vs-condition differences in that magnitude to nominate genes associated with a periodic process such as the cell cycle.

## When to use

Use when you have Cyclum-inferred circular pseudo-time for cells in two or more conditions (e.g., nicotine-treated vs control hESC 10X Chromium data) and need to rank genes by how strongly their expression kinetics match the inferred periodicity or by the treated-minus-control difference in that match.

## When NOT to use

- You only need cell-cycle phase labels or pseudo-time itself, not gene-level periodicity scores — the Cyclum model fitting suffices without the evaluation step
- The data lack a circular/periodic structure (e.g., linear differentiation trajectories); Cyclum projects data onto a predefined circular manifold and is designed for periodic processes
- You intend to use marker-gene-based cell-cycle scoring (e.g., Seurat::CellCycleScoring, scanpy.tl.score_genes_cell_cycle) as the primary quantification — Cyclum explicitly relies on the circular manifold instead of marker genes

## Inputs

- Per-condition scRNA-seq count matrices (e.g., 10X Chromium hESC matrices for nicotine-treated and control samples, 12,280 cells x 33,694 genes) preprocessed as scaled (variance = 1), centered, log2(x+1)-transformed data
- Cyclum-fitted models / inferred circular pseudo-time for each condition

## Outputs

- Per-gene circular-pattern magnitude values for each condition
- Ranked table of treated-minus-control circular-pattern magnitude differences per gene (gene_circular_magnitude_ranking.csv) with top-ranked genes such as LDHA and ENO1

## How to apply

First preprocess each condition's count matrix with the log2(x+1) transformation, centered and scaled to variance 1, as recommended in the Cyclum FAQ. Fit the Cyclum sinusoidal autoencoder (cyclum.models) separately to each condition (optionally using cyclum.tuning.CyclumAutoTune to select the number of linear components) to infer circular pseudo-time. Then use cyclum.evaluation to compute, per gene, the circular-pattern magnitude — the degree of match between the gene's expression kinetics and the inferred periodicity — in each condition. Finally, compute the treated-minus-control difference per gene, sort in descending order, and inspect the top-ranked genes; in the hESC application this ranking nominated nicotine targets LDHA and ENO1 (Supplementary Fig. 8b, Supplementary Note 2).

## Related tools

- **Cyclum** (Sinusoidal autoencoder that infers circular pseudo-time and provides cyclum.evaluation to quantify per-gene circular-pattern magnitude) — github.com/KChen-lab/cyclum
- **cyclum.tuning.CyclumAutoTune** (Automatically selects the proper number of linear components to find the 'most circular' manifold before evaluation) — github.com/KChen-lab/cyclum

## Examples

```
from cyclum.evaluation import evaluate; mags_treated = evaluate(model_treated.pseudotime, log2_treated); ranking = (mags_treated - mags_control).sort_values(ascending=False); ranking.to_csv('gene_circular_magnitude_ranking.csv')
```

## Evaluation signals

- Known periodic-process-associated genes (in the hESC nicotine application, LDHA and ENO1) appear among the top-ranked genes by treated-vs-control circular-pattern magnitude difference, reproducing Supplementary Fig. 8b
- Input matrices were log2(x+1)-transformed, centered, and scaled to variance 1 before model fitting
- The ranked difference table is sorted in descending order of treated-minus-control magnitude and exported to gene_circular_magnitude_ranking.csv
- CyclumAutoTune was run (or a suitable number of linear components chosen) so that the inferred manifold is circular rather than degenerate
- Scales to the full dataset: Cyclum can analyze 12,280 cells x 33,694 genes in ~40 min on a laptop with a GTX 960M GPU

## Limitations

- Ranked gene lists depend on separately fitting the model per condition; differences in pseudo-time inference between conditions can confound the magnitude comparison
- No version numbers for Python/Keras/TensorFlow are stated in the article; README notes pip installs latest dependencies differing from tested versions (e.g., tensorflow 1.14.0 tested, keras 2.2.4)
- Runtime details (40 min for 12,280 cells x 33,694 genes) and the nicotine nomination analysis are reported in Supplementary Note 2 rather than the main text

## Evidence

- [intro] Circular-pattern magnitude definition: "Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity."
- [methods] Preprocessing recipe: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [readme] AutoTune role: [evidence span withheld — anchor status: external_doc]
- [discussion] Scaling capability: "Cyclum can analyze an scRNA-seq dataset consisting of 12,280 cells 33,694 genes in 40 min on a laptop computer with a GTX 960M GPU and 2 GB graphic memory"
