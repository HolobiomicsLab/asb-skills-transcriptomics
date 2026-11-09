---
name: spearman-rank-correlation-of-spatial-distances
description: Use when evaluating global spatial reconstruction quality of SC-to-ST
  mapped single-cell data — i.e., checking whether predicted pairwise cell-cell distances
  preserve the rank ordering of true coordinates in a query h5ad with truth locations
  in .obs[['x','y']].
when_to_use_negative:
- You need local neighbor-level accuracy (use the average nearest neighbor hit over
  k = 20..200 or Jensen-Shannon distance of cell-type proportions instead)
- The query SC data has no truth coordinates for computing true pairwise distances,
  so the rank correlation cannot be validated
- The query data has a large gap from the ST reference, where forced SC-ST mapping
  can introduce bias — consider the MDS/de novo path, accepting that it loses the
  reference's cell distribution pre-knowledge
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellContrast
  role: Provides the eval module that projects query cells onto ST reference locations
    and computes Spearman's rank correlation of predicted vs. true pairwise distances
    as a benchmark metric
  repo: https://github.com/HKU-BAL/CellContrast
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:faae6f8103e96091ea46f36fab387f532e5796fa7ec5f9fd7bc7f6597b5a0249
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellContrast
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spearman-rank-correlation-of-spatial-distances

## Summary

Computes Spearman's rank correlation between predicted and true pairwise SC-SC distances after SC-ST spatial mapping, serving as the global spatial reconstruction metric in the CellContrast benchmark alongside nearest neighbor hit and Jensen-Shannon distance.

## When to use

Use when evaluating global spatial reconstruction quality of SC-to-ST mapped single-cell data — i.e., checking whether predicted pairwise cell-cell distances preserve the rank ordering of true coordinates in a query h5ad with truth locations in .obs[['x','y']].

## When NOT to use

- You need local neighbor-level accuracy (use the average nearest neighbor hit over k = 20..200 or Jensen-Shannon distance of cell-type proportions instead)
- The query SC data has no truth coordinates for computing true pairwise distances, so the rank correlation cannot be validated
- The query data has a large gap from the ST reference, where forced SC-ST mapping can introduce bias — consider the MDS/de novo path, accepting that it loses the reference's cell distribution pre-knowledge

## Inputs

- Reference ST AnnData h5ad (e.g., Stereo-seq E9.5_E2S1 with truth locations in .obs[['x','y']])
- Query/testing AnnData h5ad with truth coordinates in .obs[['x','y']] (e.g., SeqFISH E8.5 embryo L2 samples)
- Trained CellContrast model folder (cellContrast_models/ containing epoch_3000.pt)
- Parameter JSON file matching the training phase (e.g., parameters/parameters_spot.)

## Outputs

- results.csv with Spearman's rank correlation, nearest neighbor hit, and Jensen-Shannon distance per testing sample

## How to apply

Run the CellContrast eval module (python cellContrast.py eval) with a trained model folder, the reference ST h5ad, and the query h5ad containing truth x/y coordinates; the module projects query cells onto ST reference locations and reports Spearman's rank correlation between predicted and true pairwise distances in results.csv. Spearman's rho is a rank-based (nonparametric) measure, so it captures monotone agreement of the full distance matrix rather than exact Euclidean agreement, making it robust to scale differences between reference and query coordinate systems. Interpret it as the 'global reconstruction' complement to the local average nearest neighbor hit (over k = 20, 40, ..., 200) and the Jensen-Shannon distance of cell-type proportions among the nearest 20 neighbors (computed with Scipy's jensenshannon function). Note that after SC-ST mapping, 'cell clumping' can collapse multiple cells to the same reference location, distorting predicted SC-SC distances — the de novo (MDS) mapping path, which relies only on SC-SC estimated cosine-similarity distances, is more robust when there is a large gap between SC and ST but loses the reference's cell distribution prior. In the cross-platform benchmark (Stereo-seq E9.5_E2S1 spot reference, 5,266 spots and 348 overlapped genes, tested on SeqFISH E8.5 embryo samples), CellContrast achieved a mean Spearman's rho of approximately 0.283, which serves as the expected validation value.

## Related tools

- **CellContrast** (Provides the eval module that projects query cells onto ST reference locations and computes Spearman's rank correlation of predicted vs. true pairwise distances as a benchmark metric) — https://github.com/HKU-BAL/CellContrast

## Examples

```
python cellContrast.py eval --ref_data_path train_ST.h5ad --query_data_path query_ST.h5ad --model_folder cellContrast_models/ --parameter_file parameters/parameters_spot. --save_path results.csv
```

## Evaluation signals

- results.csv contains a spearman's rank correlation column for each testing sample (SeqFISH E8.5 embryo 1, 2, and 3)
- Mean Spearman's rho across the three SeqFISH test datasets is approximately 0.283 in the cross-platform benchmark (Table 2)
- Rank correlation is reported jointly with the other two required metrics (nearest neighbor hit and Jensen-Shannon distance ≈ 0.305 mean JSD; ≈ 55.388 mean average hit at k=200)
- The parameter file passed to eval matches the one used at training (e.g., parameters_spot for spot-based Stereo-seq reference)
- rho is a rank correlation in [-1, 1]; values near 0 or negative indicate the mapping failed to preserve global distance ordering

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, causing overlap and loss of predicted SC-SC distances and degrading Spearman's rho
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods; forcing SC assignment to resemble the ST could introduce more bias
- Tissues with a high level of repeating elements could pose a challenge for spatial reconstruction of SC cells

## Evidence

- [readme] three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation.: "The peformance of benchmarking can be evaluated with the following command, and three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation."
- [discussion] the approach of SC-ST mapping also introduces a potential "cell clumping" issue: "the approach of SC-ST mapping also introduces a potential "cell clumping" issue, where multiple neighbors are projected into the same reference location"
- [discussion] large gaps between them can lead to a performance drop in global reconstruction across all methods: "large gaps between them can lead to a performance drop in global reconstruction across all methods"
