---
name: jensen-shannon-cell-type-distribution-comparison
description: Use when evaluating spatial reconstruction of single-cell data (e.g.,
  CellContrast on SeqFISH mouse gastrulation sections) and you need to score whether
  reconstructed neighbors reproduce the reference local cell-type composition.
when_to_use_negative:
- When the goal is global spatial alignment rather than local neighborhood composition
  — use SC-ST mapping plus Spearman's rank correlation of pairwise distances instead
- When truth cell-type annotations or truth coordinates (.obs[['x','y']]) are unavailable
  for the test section, since JSD cannot be computed without a reference composition
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: CellContrast
  role: Provides the eval module (python cellContrast.py eval) that computes the Jensen-Shannon
    distance metric alongside neighbor hit and Spearman's rank correlation
  repo: https://github.com/HKU-BAL/CellContrast
- name: Scipy
  role: Supplies the jensenshannon function used to compute the Jensen-Shannon distance
    between predicted and true cell-type proportion distributions
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:d9e4505388c0dc176acefa055dbec4ecf9a6f7e57c4cf5bfd158a8052030cfa9
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# jensen-shannon-cell-type-distribution-comparison

## Summary

Benchmark metric for spatial reconstruction methods that compares predicted versus true cell-type proportions among the k=20 nearest neighbors of each cell using the Jensen-Shannon distance (Scipy jensenshannon), quantifying how well local cell-type composition is preserved.

## When to use

Use when evaluating spatial reconstruction of single-cell data (e.g., CellContrast on SeqFISH mouse gastrulation sections) and you need to score whether reconstructed neighbors reproduce the reference local cell-type composition.

## When NOT to use

- When the goal is global spatial alignment rather than local neighborhood composition — use SC-ST mapping plus Spearman's rank correlation of pairwise distances instead
- When truth cell-type annotations or truth coordinates (.obs[['x','y']]) are unavailable for the test section, since JSD cannot be computed without a reference composition

## Inputs

- Predicted nearest-neighbor lists for query cells (from SC-SC similarities or predicted coordinates)
- Truth cell coordinates for query/test sections in AnnData .obs[['x','y']] (e.g., SeqFISH embryo 1 L2, 2 L2, 3 L2 h5ad files)
- Cell-type annotations for query cells to build proportion vectors

## Outputs

- Per-sample Jensen-Shannon distance values written to results.csv (e.g., ~0.237 for CellContrast on embryo 1 L2)
- Per-cell JSD between predicted and true cell-type proportion distributions among k=20 nearest neighbors

## How to apply

After a spatial method (e.g., CellContrast trained with infoNCE loss for 3,000 epochs) has assigned cell neighbors, derive the k=20 nearest neighbors for each query cell both from the predicted spatial relationships and from the truth coordinates stored in .obs[['x','y']]. For each cell, build the cell-type proportion vectors over its predicted and true 20-neighbor sets, then compute the Jensen-Shannon distance between the two distributions with Scipy's jensenshannon function. Average the per-cell JSD over the test sample (e.g., SeqFISH embryo 1 L2) and report it alongside neighbor hit and Spearman's rank correlation of pairwise distances. On the cross-platform benchmark (Stereo-seq E9.5 reference, SeqFISH E8.5 queries), CellContrast's mean JSD is approximately 0.305; on the within-platform SeqFISH embryo 1 L2 benchmark, approximately 0.237 versus 0.289 for SageNet and 0.324 for CytoSpace. The eval module of cellContrast.py computes this metric automatically and writes it to results.csv.

## Related tools

- **CellContrast** (Provides the eval module (python cellContrast.py eval) that computes the Jensen-Shannon distance metric alongside neighbor hit and Spearman's rank correlation) — https://github.com/HKU-BAL/CellContrast
- **Scipy** (Supplies the jensenshannon function used to compute the Jensen-Shannon distance between predicted and true cell-type proportion distributions)

## Examples

```
python cellContrast.py eval --ref_data_path ref_ST.h5ad --query_data_path query_ST.h5ad --model_folder cellContrast_models/ --parameter_file parameters/parameters_spot. --save_path results.csv
```

## Evaluation signals

- CellContrast on SeqFISH embryo 1 L2 should yield mean JSD near 0.237 (vs 0.289 SageNet, 0.324 CytoSpace)
- Cross-platform run (Stereo-seq E9.5 spot reference, SeqFISH E8.5 queries) should show mean JSD near 0.305
- results.csv must contain JSD values for each of the three test sections for every method
- JSD must be bounded in [0, 1] since Scipy jensenshannon returns base-2 Jensen-Shannon distance

## Limitations

- The metric only assesses local neighborhood composition (k=20), not global reconstruction fidelity; large gaps between ST reference and SC query degrade global reconstruction across all methods
- Computation requires ground-truth coordinates and cell-type labels, so it applies only to benchmark datasets with truth locations

## Evidence

- [readme] The peformance of benchmarking can be evaluated with the following command, and three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation.: "The peformance of benchmarking can be evaluated with the following command, and three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation."
- [methods] JSD between predicted and true cell-type proportions of the k=20 nearest neighbors computed with the scipy jensenshannon function: [evidence span withheld — anchor status: not_found]
- [results] average JSD near 0.237 on embryo 1 L2 (vs 0.289 SageNet, 0.324 CytoSpace): [evidence span withheld — anchor status: not_found]
