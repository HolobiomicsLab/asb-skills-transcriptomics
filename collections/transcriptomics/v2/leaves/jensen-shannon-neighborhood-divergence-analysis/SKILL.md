---
name: jensen-shannon-neighborhood-divergence-analysis
description: 'Use when benchmarking a spatial reconstruction method for single-cell
  data: a predicted neighbor structure exists for query cells with known truth locations,
  and you need a local, neighborhood-composition quality metric beyond raw neighbor
  hits.'
when_to_use_negative:
- You lack cell-type annotations for the query cells, so neighborhood cell-type distributions
  cannot be formed.
- You only need global reconstruction fidelity — Spearman's rank correlation of global
  distances is the appropriate metric, not neighborhood JSD.
- The test data has no truth locations, so no reference neighborhood composition exists
  to compare against.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellContrast
  role: Provides the eval module that computes Jensen-Shannon distance of neighborhood
    cell-type distributions, nearest-neighbor hits, and Spearman's rank correlation
    for benchmarking spatial reconstruction
  repo: https://github.com/HKU-BAL/CellContrast
- name: SageNet
  role: Comparator method whose reconstruction is also scored by JSD (0.369 in the
    breast benchmark)
- name: NovoSpaRc
  role: Comparator method whose reconstruction is also scored by JSD (0.393 in the
    breast benchmark)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:bcdabd5317219aa750269e4b9f7a0080dd185fe87725d4db29bb9cb5a904d990
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellContrast
  - NovoSpaRc
  - SageNet
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# jensen-shannon-neighborhood-divergence-analysis

## Summary

Evaluation skill that scores spatial reconstruction quality by comparing cell-type composition of k-nearest-neighborhoods between predicted and true spatial layouts using Jensen-Shannon distance, as implemented in CellContrast's eval module.

## When to use

Use when benchmarking a spatial reconstruction method for single-cell data: a predicted neighbor structure exists for query cells with known truth locations, and you need a local, neighborhood-composition quality metric beyond raw neighbor hits.

## When NOT to use

- You lack cell-type annotations for the query cells, so neighborhood cell-type distributions cannot be formed.
- You only need global reconstruction fidelity — Spearman's rank correlation of global distances is the appropriate metric, not neighborhood JSD.
- The test data has no truth locations, so no reference neighborhood composition exists to compare against.

## Inputs

- Reference ST AnnData (.h5ad) with truth locations in .obs[['x','y']]
- Testing/query AnnData (.h5ad) with truth locations for evaluation
- Trained CellContrast model folder (e.g. cellContrast_models/ containing epoch_3000.pt)
- Parameter JSON file used during training (e.g. parameters/parameters_spot.)
- Cell-type annotations for query cells to compose neighborhood distributions

## Outputs

- results.csv with JSD, nearest-neighbor hits, and Spearman's rank correlation per testing sample

## How to apply

Run CellContrast's eval module, which takes a reference ST h5ad, a testing h5ad with truth locations in .obs[['x','y']], the trained model folder, and the parameter file used at training. For each query cell, gather its k nearest predicted neighbors (JSD is computed at k=20 in the benchmark) and compare the cell-type distribution of that neighborhood against the distribution around the true location, computing the Jensen-Shannon distance. Lower JSD indicates neighborhoods whose cell-type composition better matches the spatial ground truth; in the 10X Visium human breast benchmark (S06 train, S08 test), CellContrast achieved a mean average JSD of about 0.306 versus SageNet (0.369) and NovoSpaRc (0.393). Report results alongside nearest-neighbor hits (k=20..200) and Spearman's rank correlation of global distances in results.csv, since JSD is a local metric that complements these global and raw-hit measures.

## Related tools

- **CellContrast** (Provides the eval module that computes Jensen-Shannon distance of neighborhood cell-type distributions, nearest-neighbor hits, and Spearman's rank correlation for benchmarking spatial reconstruction) — https://github.com/HKU-BAL/CellContrast
- **SageNet** (Comparator method whose reconstruction is also scored by JSD (0.369 in the breast benchmark))
- **NovoSpaRc** (Comparator method whose reconstruction is also scored by JSD (0.393 in the breast benchmark))

## Examples

```
python cellContrast.py eval --ref_data_path ref_ST.h5ad --query_data_path query_ST.h5ad --model_folder cellContrast_models/ --parameter_file parameters/parameters_spot. --save_path results.csv
```

## Evaluation signals

- results.csv is produced and contains JSD, neighbor hit, and Spearman's rank correlation for each testing sample.
- Mean average JSD at k=20 for the 10X Visium breast S06-train/S08-test benchmark is approximately 0.306 for CellContrast.
- JSD should be lower than competitor baselines: SageNet at 0.369 and NovoSpaRc at 0.393.
- JSD is reported together with neighbor hits (k=20..200, ~58.556 true neighbors at k=200) so local and raw-hit metrics agree qualitatively.
- The same parameter file must be passed to eval as was used in training, ensuring consistent neighborhood sizes and model settings.

## Limitations

- JSD is a local neighborhood metric and does not capture global reconstruction fidelity; use it jointly with Spearman's rank correlation.
- Requires ground-truth locations in the test sample, limiting use to datasets with known spatial truth.
- Tissues with a high level of repeating elements could pose a challenge for spatial reconstruction, degrading JSD alongside other metrics.
- Large gaps between ST and SC samples lead to a performance drop in reconstruction across all methods, which will inflate JSD.

## Evidence

- [readme] three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation: "The peformance of benchmarking can be evaluated with the following command, and three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation."
- [results] average neighborhood JSD of 0.306 versus SageNet 0.369 and NovoSpaRc 0.393: [evidence span withheld — anchor status: not_found]
- [discussion] tissues with repeating elements challenge spatial reconstruction: "tissues with a high level of repeating elements could pose a challenge for the spatial reconstruction of SC cells"
- [readme] truth locations stored in AnnData obs x,y: "CellContrast model was trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']])`"
