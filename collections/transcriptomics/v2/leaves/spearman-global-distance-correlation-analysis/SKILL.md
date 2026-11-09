---
name: spearman-global-distance-correlation-analysis
description: Use when benchmarking SC-to-ST spatial reconstruction methods (e.g.,
  CellContrast vs SageNet, CeLEry, CytoSpace, NovoSpaRc, Tangram) on test sections
  with truth locations, to quantify global distance-structure recovery alongside neighbor-hit
  and JSD metrics.
when_to_use_negative:
- When the goal is only local neighborhood accuracy (nearest-neighbor hit or JSD of
  cell-type proportions), not global distance-structure fidelity
- When the query section lacks truth locations in .obs[['x','y']], since the correlation
  requires true pairwise distances as reference
- When the method outputs only SC-ST spot assignments without SC-SC distances or coordinates
  usable to derive pairwise distances
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellContrast
  role: eval module computes Spearman's rank correlation alongside neighbor hit and
    JSD for benchmarked spatial-reconstruction methods
  repo: https://github.com/HKU-BAL/CellContrast
- name: SciPy
  role: provides the rank-correlation statistic (and jensenshannon) used to score
    predicted vs true pairwise distances
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:d55f0baf1282faae1fb097a5e5b5f15906fcd8be71f468947c05d12f77c1b746
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - CytoSpace
  - NovoSpaRc
  - SageNet
  - SciPy
  - Scipy
  - Tangram
  - scran
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Spearman global distance correlation analysis

## Summary

Benchmark metric that scores a spatial-reconstruction method by computing Spearman's rank correlation between predicted and true pairwise cell-cell distances for each query cell, capturing fidelity of global spatial structure rather than only local neighbor accuracy.

## When to use

Use when benchmarking SC-to-ST spatial reconstruction methods (e.g., CellContrast vs SageNet, CeLEry, CytoSpace, NovoSpaRc, Tangram) on test sections with truth locations, to quantify global distance-structure recovery alongside neighbor-hit and JSD metrics.

## When NOT to use

- When the goal is only local neighborhood accuracy (nearest-neighbor hit or JSD of cell-type proportions), not global distance-structure fidelity
- When the query section lacks truth locations in .obs[['x','y']], since the correlation requires true pairwise distances as reference
- When the method outputs only SC-ST spot assignments without SC-SC distances or coordinates usable to derive pairwise distances

## Inputs

- AnnData (.h5ad) query SC/ST section with truth locations in .obs[['x','y']]
- Predicted SC-SC cell-cell distances (cosine similarities of encoder representations) or predicted 2D coordinates per method
- Trained CellContrast model checkpoint (epoch_3000.pt) and matching parameters/parameters_singleCell.
- Reference ST h5ad with truth cell locations for SC-ST mapping

## Outputs

- Per-sample Spearman's rank correlation between predicted and true pairwise distances
- results.csv rows combining neighbor hit, JSD, and Spearman's rho per method and test dataset

## How to apply

For each test section with truth locations in .obs[['x','y']] (e.g., SeqFISH embryo 1 L2, embryo 2 L2, embryo 3 L2), obtain each method's SC-SC cell-cell distance estimates — cosine similarities of the learnt encoder representations for CellContrast, or Euclidean distances of predicted coordinates for coordinate-output methods. Compute per-cell Spearman's rank correlation between the predicted and true pairwise distance vectors using scipy's rank-correlation implementation. Average or report per-sample values across the k-neighbor-based predictions, then write them with neighbor hit and JSD (scipy jensenshannon, k=20 neighbors) to results.csv via the CellContrast eval module. Interpret higher Spearman's rho as better global reconstruction; the CellContrast paper reports rho near 0.874 on embryo 1 L2 as the reproduced expectation.

## Related tools

- **CellContrast** (eval module computes Spearman's rank correlation alongside neighbor hit and JSD for benchmarked spatial-reconstruction methods) — https://github.com/HKU-BAL/CellContrast
- **SciPy** (provides the rank-correlation statistic (and jensenshannon) used to score predicted vs true pairwise distances)

## Examples

```
python cellContrast.py eval --ref_data_path ref_ST.h5ad --query_data_path query_ST.h5ad --model_folder cellContrast_models/ --parameter_file parameters/parameters_singleCell. --save_path results.csv
```

## Evaluation signals

- Reproduced Spearman's rho on SeqFISH embryo 1 L2 should be near 0.874 for CellContrast
- results.csv must contain one Spearman's rho value per method (six methods) per test dataset (three test sections)
- Correlations must be computed between predicted and TRUE pairwise distances per cell, not between predicted coordinates and true coordinates directly
- Distance inputs must be consistent per method: cosine similarities of representations for similarity-based methods, Euclidean distances of predicted coordinates for coordinate-output methods

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, distorting predicted SC-SC distances
- Using predicted neighbors based on SC-SC distances may result in a loss of similarity to the reference spatial distributions
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods

## Evidence

- [intro] encoder representation reuse: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
- [discussion] distance estimation via cosine similarity: "The resulting cell-cell distances are estimated through cosine similarities of learnt representations and therefore identifying the potential cell neighbors."
- [discussion] coordinate mapping to reference: "The spatially related representations of SC cells can be further mapped to location coordinates using ST reference or MDS algorithm."
- [readme] three-metric benchmarking incl. Spearman: "The peformance of benchmarking can be evaluated with the following command, and three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation."
- [readme] eval output schema: "## Output file: result.csv with neighbor hit, JSD, spearman's rank correlation for each testing sample."
