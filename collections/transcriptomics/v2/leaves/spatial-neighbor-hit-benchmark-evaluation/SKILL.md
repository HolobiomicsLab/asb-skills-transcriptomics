---
name: spatial-neighbor-hit-benchmark-evaluation
description: Use when benchmarking SC-to-ST spatial reconstruction methods (CellContrast,
  SageNet, CeLEry, CytoSpace, NovoSpaRc, Tangram) on data with truth cell locations,
  e.g. SeqFISH mouse gastrulation sections in AnnData .h5ad with .obs[['x','y']].
when_to_use_negative:
- Query data lacks ground-truth cell locations in .obs[['x','y']] — neighbor hit,
  JSD, and Spearman metrics all require truth coordinates to score against
- The goal is producing spatially reconstructed SC coordinates for downstream use
  (co-localization, CCC) rather than benchmarking — use CellContrast `inference`/`reconstruct`
  instead
- Evaluation targets sequencing-based ST spots rather than single-cell-resolution
  imaging data — switch to parameters_spot. defaults, not the single-cell parameter
  file
tools:
- name: CellContrast
  role: Provides the `eval` module that computes neighbor hit, JSD, and Spearman's
    rank correlation against a reference ST with truth locations
  repo: https://github.com/HKU-BAL/CellContrast
- name: Scipy
  role: jensenshannon function used to compute JSD between predicted and true cell-type
    proportions of k=20 nearest neighbors
- name: SageNet
  role: Baseline method benchmarked with the same three metrics; trained on seqFISH
    spatial partitions for 20 epochs with GLASSO regularization
- name: CeLEry
  role: Supervised deep-learning baseline benchmarked with the same three metrics
    under default parameters
- name: CytoSpace
  role: Baseline run with '-single-cell' and '-noss 10000'; evaluated via Euclidean
    distances of predicted coordinates
- name: NovoSpaRc
  role: Baseline run with num_neighbors_s/t=5, alpha_linear=0.8, epsilon=5e-3; evaluated
    on the same three metrics
- name: Tangram
  role: Baseline run with all genes and default parameters; evaluated on the same
    three metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:d166ac18500a4b3014b50cc8ba03b69b4e034de54cbe069508cd2f6101d8d099
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
  - Scipy
  - Tangram
  - scran
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# spatial-neighbor-hit-benchmark-evaluation

## Summary

Benchmark spatial reconstruction methods for single-cell RNA-seq by scoring predicted cell neighborhoods against ground-truth coordinates using three metrics: average nearest neighbor hit (k=20 to 200), Jensen-Shannon distance (JSD) of cell-type proportions, and Spearman's rank correlation of pairwise distances. This is the CellContrast paper's evaluation protocol for judging local and global spatial reconstruction quality.

## When to use

Use when benchmarking SC-to-ST spatial reconstruction methods (CellContrast, SageNet, CeLEry, CytoSpace, NovoSpaRc, Tangram) on data with truth cell locations, e.g. SeqFISH mouse gastrulation sections in AnnData .h5ad with .obs[['x','y']].

## When NOT to use

- Query data lacks ground-truth cell locations in .obs[['x','y']] — neighbor hit, JSD, and Spearman metrics all require truth coordinates to score against
- The goal is producing spatially reconstructed SC coordinates for downstream use (co-localization, CCC) rather than benchmarking — use CellContrast `inference`/`reconstruct` instead
- Evaluation targets sequencing-based ST spots rather than single-cell-resolution imaging data — switch to parameters_spot. defaults, not the single-cell parameter file

## Inputs

- Reference ST AnnData .h5ad with truth cell locations in .obs[['x','y']] (e.g. SeqFISH embryo 1 L1, 10,150 cells, 351 genes, log-normalized with scran)
- Query/test AnnData .h5ad with truth locations for scoring (e.g. embryo 1 L2, embryo 2 L2, embryo 3 L2)
- Trained model checkpoint folder (e.g. cellContrast_models/epoch_3000.pt) or predicted outputs from baseline methods
- Parameter JSON matching training (e.g. parameters/parameters_singleCell.)

## Outputs

- results.csv with neighbor hit, JSD, and Spearman's rank correlation for each testing sample and each method
- Per-method median average hit ratio over k=20..200 for cross-method comparison tables

## How to apply

Run the CellContrast `eval` module with a reference ST h5ad (with truth locations in .obs[['x','y']]), a query h5ad with truth locations, the trained model folder, and the same parameter file used at training (e.g. parameters/parameters_singleCell. for imaging-based ST). Compute neighbor hits from SC-SC similarities (or Euclidean distances of predicted coordinates for coordinate-output methods) for k = 20, 40, ..., 200; compute JSD between predicted and true cell-type proportions of the k=20 nearest neighbors using scipy's jensenshannon function; and compute Spearman's rank correlation between predicted and true pairwise distances per cell. Derive global positions by SC-ST mapping to the reference section (e.g. embryo 1 L1) and evaluate each method on the held-out test sections (embryo 1 L2, embryo 2 L2, embryo 3 L2). Write per-sample neighbor hit, JSD, and Spearman values for all methods to results.csv. Rationale: cosine similarity of the encoder's learnt representations (projector discarded) indicates shorter true spatial distances, so neighbor-hit recovery directly probes the method's core claim.

## Related tools

- **CellContrast** (Provides the `eval` module that computes neighbor hit, JSD, and Spearman's rank correlation against a reference ST with truth locations) — https://github.com/HKU-BAL/CellContrast
- **Scipy** (jensenshannon function used to compute JSD between predicted and true cell-type proportions of k=20 nearest neighbors)
- **SageNet** (Baseline method benchmarked with the same three metrics; trained on seqFISH spatial partitions for 20 epochs with GLASSO regularization)
- **CeLEry** (Supervised deep-learning baseline benchmarked with the same three metrics under default parameters)
- **CytoSpace** (Baseline run with '-single-cell' and '-noss 10000'; evaluated via Euclidean distances of predicted coordinates)
- **NovoSpaRc** (Baseline run with num_neighbors_s/t=5, alpha_linear=0.8, epsilon=5e-3; evaluated on the same three metrics)
- **Tangram** (Baseline run with all genes and default parameters; evaluated on the same three metrics)

## Examples

```
python cellContrast.py eval --ref_data_path ref_ST.h5ad --query_data_path query_ST.h5ad --model_folder cellContrast_models --parameter_file parameters/parameters_singleCell. --save_path results.csv
```

## Evaluation signals

- Median average hit ratio (k=20..200) for CellContrast on the SeqFISH embryo 1 L1 reference should be near 47.32%, versus 36.22% for SageNet and 14.27% for CeLEry
- Average JSD on embryo 1 L2 should be near 0.237 for CellContrast (vs 0.289 SageNet, 0.324 CytoSpace)
- Spearman's rank correlation on embryo 1 L2 should be near 0.874 for CellContrast
- results.csv must contain per-sample neighbor hit, JSD, and Spearman values for all six methods across the three test sections (embryo 1 L2, 2 L2, 3 L2)
- Neighbor hits are computed at k = 20, 40, ..., 200 and JSD uses the scipy jensenshannon function on cell-type proportions of the k=20 nearest neighbors

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, causing overlap and loss of predicted SC-SC distances
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods; forcing SC assignment to resemble ST in this case can introduce more bias
- Using predicted neighbors based on SC-SC distances may result in a loss of similarity to the reference spatial distributions, though it yields more accurate cell neighbor identification
- Tissues with a high level of repeating elements could pose a challenge for spatial reconstruction and thus for meaningful benchmarking

## Evidence

- [readme] three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation: "The peformance of benchmarking can be evaluated with the following command, and three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation."
- [readme] Output file: result.csv with neighbor hit, JSD, spearman's rank correlation for each testing sample.: "## Output file: result.csv with neighbor hit, JSD, spearman's rank correlation for each testing sample."
- [discussion] The resulting cell-cell distances are estimated through cosine similarities of learnt representations and therefore identifying the potential cell neighbors.: "The resulting cell-cell distances are estimated through cosine similarities of learnt representations and therefore identifying the potential cell neighbors."
- [discussion] CellContrast consistently outperformed existing methods in local spatial reconstruction, such as SageNet,14 CeLEry,15 CytoSpace,13 NovoSpaRc,11 and Tangram,12: "CellContrast consistently outperformed existing methods in local spatial reconstruction, such as SageNet,14 CeLEry,15 CytoSpace,13 NovoSpaRc,11 and Tangram,12"
- [discussion] the approach of SC-ST mapping also introduces a potential "cell clumping" issue: "the approach of SC-ST mapping also introduces a potential "cell clumping" issue, where multiple neighbors are projected into the same reference location"
