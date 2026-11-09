---
name: mae-pcc-correction-accuracy-evaluation
description: Use when you have simulated technical dropouts (e.g., UMIs of marker
  genes subsampled to 50/100/200 counts) plus un-subsampled ground-truth expression,
  and you need to quantify and compare how well correction methods (e.g., SpatialScope
  vs Tangram) recover the true spatial expression profiles.
when_to_use_negative:
- There is no ground-truth (un-subsampled) expression reference — MAE and PCC cannot
  be computed without true values
- The goal is deconvolving cell-type proportions per spot (use RCTD/Cell2location/CARD)
  rather than evaluating gene-expression recovery accuracy
- Data are already measured at single-cell resolution with a predefined panel (e.g.,
  MERFISH); no dropout-correction benchmark on seq-based ST data is needed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Primary dropout-correction method whose posterior-mean expression is scored
    by MAE/PCC against ground truth
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Baseline imputation method compared against SpatialScope in the MAE/PCC benchmark
- name: RCTD (spacexr)
  role: Inspired the singlet/doublet classification that produces the 10,975-cell
    spot decomposition used as the evaluation substrate
  repo: https://github.com/dmcable/spacexr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:11c4da876aab86657f9537cd47da5aafe0c0cad68929fe9f0f0ade3de16b9ee6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - RCTD
  - RCTD (spacexr)
  - SpatialScope
  - Tangram
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mae-pcc-correction-accuracy-evaluation

## Summary

Benchmarks dropout-correction / gene-expression-imputation performance by computing mean absolute error (MAE) and Pearson's correlation (PCC) of corrected versus ground-truth expression across simulated subsampling settings, as done to compare SpatialScope against Tangram on Slide-seq V2 cerebellum data.

## When to use

Use when you have simulated technical dropouts (e.g., UMIs of marker genes subsampled to 50/100/200 counts) plus un-subsampled ground-truth expression, and you need to quantify and compare how well correction methods (e.g., SpatialScope vs Tangram) recover the true spatial expression profiles.

## When NOT to use

- There is no ground-truth (un-subsampled) expression reference — MAE and PCC cannot be computed without true values
- The goal is deconvolving cell-type proportions per spot (use RCTD/Cell2location/CARD) rather than evaluating gene-expression recovery accuracy
- Data are already measured at single-cell resolution with a predefined panel (e.g., MERFISH); no dropout-correction benchmark on seq-based ST data is needed

## Inputs

- Ground-truth spatial expression matrix from un-subsampled Slide-seq V2 cerebellum data (20,141 genes, 8,952 spots)
- Subsampled (dropout-simulated) expression matrices with marker-gene UMIs reduced to 50, 100, and 200 counts
- Corrected/denoised expression matrices output by SpatialScope and Tangram on each subsampled dataset
- List of the 22 high-capture marker genes selected for subsampling

## Outputs

- MAE and PCC values per method per subsampling level (50/100/200 counts)
- Comparison table of SpatialScope vs Tangram correction accuracy (reproducing Fig. 5b–d)
- Posterior-mean dropout-corrected single-cell expression covering 10,975 cells across 8,952 spots

## How to apply

First simulate dropout by subsampling UMIs of a fixed panel of high-capture marker genes (the paper used 22 markers on Slide-seq V2 cerebellum data) to controlled total-count levels (50, 100, and 200 counts), keeping the un-subsampled data as ground truth. Re-run each correction method — SpatialScope with its conditional score-based generative model and annealed Langevin dynamics posterior sampling, and the baseline Tangram — on each subsampled dataset. For every method and subsampling level, compute MAE and PCC between the corrected posterior-mean expression and the ground-truth expression over the evaluated cells/spots (10,975 cells across 8,952 spots in the paper). Assemble the per-setting comparison table and judge correctness by whether the top-performing method attains the lowest MAE and highest PCC in all subsampling settings, reproducing the Fig. 5b–d pattern.

## Related tools

- **SpatialScope** (Primary dropout-correction method whose posterior-mean expression is scored by MAE/PCC against ground truth) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Baseline imputation method compared against SpatialScope in the MAE/PCC benchmark)
- **RCTD (spacexr)** (Inspired the singlet/doublet classification that produces the 10,975-cell spot decomposition used as the evaluation substrate) — https://github.com/dmcable/spacexr

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml
```

## Evaluation signals

- SpatialScope attains the lowest MAE and highest PCC versus Tangram in all three subsampling settings (50, 100, 200 counts), reproducing Fig. 5b–d
- Corrected output covers exactly 10,975 cells across 8,952 spots
- MAE decreases and PCC increases monotonically as subsampling level rises from 50 to 200 counts
- MAE and PCC are computed cell-wise between corrected and ground-truth expression over the same 22-marker gene panel

## Limitations

- Dropouts are simulated by UMI subsampling, which may not capture real technical dropout mechanisms
- Only 22 high-capture marker genes were subsampled, so accuracy conclusions are marker-panel specific
- Training the deep generative models is time-consuming (hours or days), limiting rapid re-benchmarking

## Evidence

- [results] On the simulated dropout experiment, SpatialScope achieved the best dropout correction performance in all subsampling settings, as measured by MAE and PCC, outperforming Tangram.: [evidence span withheld — anchor status: not_found]
- [methods] Filtering rule applied before benchmarking.: "We filtered out genes that have zero counts across all spots and filtered out spots with total UMI counts less than 100"
- [methods] Posterior sampling used to obtain the corrected expression being scored.: [evidence span withheld — anchor status: not_found]
- [methods] Subsampling levels applied to marker genes to simulate dropouts.: [evidence span withheld — anchor status: not_found]
- [readme] Baseline tool installation for comparison runs.: [evidence span withheld — anchor status: external_doc]
