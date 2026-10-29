---
name: hydra-neuron-subtype-cluster-evaluation
description: Use when you need to quantify whether merging visually defined single-cell
  clusters (e.g., five Hydra neuron ectodermal subtypes, or CAR-T clusters 3+4+9+11
  under the scDEED-optimized t-SNE) preserves or reduces cluster purity as measured
  by ROGUE.
when_to_use_negative:
- Input cluster labels come from a non-optimized t-SNE or UMAP embedding, not the
  scDEED-optimized t-SNE the CAR-T combinations were defined under
- Data is raw counts without cluster assignments in a Seurat object — ROGUE purity
  scoring requires pre-defined cluster labels
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Reads Seurat object cluster labels for the Hydra neuron ectodermal subtypes
    and CAR-T clusters; scDEED uses Seurat objects and the optimized t-SNE is run
    through it
  repo: https://satijalab.org/seurat/
- name: Rogue
  role: R package whose CalculateRogue function computes cluster purity for each Hydra
    subtype, the unified ectodermal neuron set, and each CAR-T cluster combination
- name: R
  role: Analysis environment; packages installed via devtools::install_github and
    analysis scripted in R
- name: scDEED
  role: Provided the optimized t-SNE hyperparameters under which the CAR-T cluster
    combinations were defined
  repo: github.com/JSB-UCLA/scDEED
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:02abb70e62c456a29ac7ff4f03be520ccbf508628ee25dcc9fcbbf8d40d8e379
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rogue
  - Seurat
  - scDEED
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hydra-neuron-subtype-cluster-evaluation

## Summary

Recompute ROGUE cluster-purity scores for the five Hydra neuron ectodermal subtypes (individually and merged) and for CAR-T cluster combinations defined under the scDEED-optimized t-SNE, using Seurat cluster labels and the R Rogue package, to test whether unifying subtypes meaningfully reduces purity.

## When to use

Use when you need to quantify whether merging visually defined single-cell clusters (e.g., five Hydra neuron ectodermal subtypes, or CAR-T clusters 3+4+9+11 under the scDEED-optimized t-SNE) preserves or reduces cluster purity as measured by ROGUE.

## When NOT to use

- Input cluster labels come from a non-optimized t-SNE or UMAP embedding, not the scDEED-optimized t-SNE the CAR-T combinations were defined under
- Data is raw counts without cluster assignments in a Seurat object — ROGUE purity scoring requires pre-defined cluster labels

## Inputs

- Seurat object with Hydra dataset cluster assignments (five neuron ectodermal subtypes)
- Seurat object with CAR-T cell cluster assignments under the scDEED-optimized t-SNE
- input_counts (Hydra dataset, 10000 randomly selected cells) as a Seurat object starting point

## Outputs

- rogue_scores.csv with per-subtype ROGUE values (Hydra: 0.710/0.784/0.714/0.793/0.839), unified-cluster ROGUE (0.744), average (0.768) and sd (0.055), and CAR-T combination ROGUE values (0.887/0.885/0.886 vs 0.832 for merged clusters 3+4+9+11)

## How to apply

Load the published Hydra dataset cluster assignments (five neuron ectodermal subtypes) and the CAR-T cell cluster assignments defined under the scDEED-optimized t-SNE into R, reading Seurat object cluster labels via Seurat. Install and load the R package Rogue, then run CalculateRogue on each of the five Hydra neuron ectodermal subtype clusters separately, and again on the unified ectodermal neuron set; compute the average and standard deviation of the five per-subtype values. Separately run CalculateRogue on each CAR-T cluster combination under the optimized t-SNE, including the merged combination of clusters 3+4+9+11. Compile all recomputed scores into rogue_scores.csv and compare against the reported values to validate reproduction.

## Related tools

- **Seurat** (Reads Seurat object cluster labels for the Hydra neuron ectodermal subtypes and CAR-T clusters; scDEED uses Seurat objects and the optimized t-SNE is run through it) — https://satijalab.org/seurat/
- **Rogue** (R package whose CalculateRogue function computes cluster purity for each Hydra subtype, the unified ectodermal neuron set, and each CAR-T cluster combination)
- **R** (Analysis environment; packages installed via devtools::install_github and analysis scripted in R)
- **scDEED** (Provided the optimized t-SNE hyperparameters under which the CAR-T cluster combinations were defined) — github.com/JSB-UCLA/scDEED

## Examples

```
library(Seurat); load("input_counts.rda"); data = CreateSeuratObject(input_counts); data = NormalizeData(data)
```

## Evaluation signals

- Hydra per-subtype ROGUE values reproduce 0.710/0.784/0.714/0.793/0.839 (average 0.768, sd 0.055)
- Unified Hydra ectodermal neuron ROGUE reproduces 0.744, showing unification did not meaningfully reduce purity (0.744 vs 0.768 average)
- CAR-T combination ROGUE values reproduce 0.887/0.885/0.886, with the merged clusters 3+4+9+11 yielding 0.832
- rogue_scores.csv contains all recomputed values and matches the reported values on diff/comparison

## Limitations

- The task validates that unifying the five Hydra neuron ectodermal subtypes did NOT meaningfully reduce purity — do not interpret the small ROGUE drop (0.744 vs 0.768) as evidence of misclassification on its own
- Seurat version differences (5.2.1 vs 5.1.0) reportedly affect installation; avoid updating all packages when installing scDEED dependencies
- CAR-T combination scores are only valid under the scDEED-optimized t-SNE; other hyperparameter settings yield different cluster definitions

## Evidence

- [other] Install and load the R package Rogue and run CalculateRogue on each of the five Hydra neuron ectodermal subtype clusters separately to obtain per-subtype ROGUE values.: [evidence span withheld — anchor status: not_found]
- [other] the recomputed scores should reproduce the reported values — Hydra per-subtype ROGUE of 0.710/0.784/0.714/0.793/0.839 (average 0.768, sd 0.055) versus unified 0.744: [evidence span withheld — anchor status: not_found]
- [readme] scDEED uses Seurat objects. This input_counts contains 10000 randomly selected cells from the Hydra dataset.: "scDEED uses Seurat objects. This input_counts contains 10000 randomly selected cells from the Hydra dataset."
- [readme] You can install scDEED from GitHub using devtools. It should install in 1 minute, but may take longer if you need to update dependencies.: "You can install scDEED from GitHub using devtools. It should install in 1 minute, but may take longer if you need to update dependencies."
