---
name: perturbation-response-error-comparison
description: Use when you must quantitatively compare perturbation-effect estimation
  methods (e.
when_to_use_negative:
- Data are real scCRISPR-seq experiments without known ground-truth cellular responses
  — error-against-truth comparison is impossible; use real-data pathway/gene-ranking
  evaluation instead.
- You only need to run scDecouple on one dataset without a competing baseline — this
  skill is specifically a method-vs-method benchmark, not a single-analysis pipeline.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: scDecouple
  role: Estimates cellular response from Y and Z via Gaussian mixture modeling and
    EM (run_scDecouple), the method under evaluation against the conventional baseline
  repo: https://github.com/MengQiuchen/scDecouple
- name: Seurat
  role: Clustering of simulated control-group cells to implement the conventional
    cluster-annotation-and-mapping perturbation effect estimation strategy
- name: tidyverse
  role: R data handling and aggregation of per-setting error metrics
- name: devtools
  role: Installs the developmental scDecouple package from GitHub (devtools::install_github)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:7af92d94c23d291c2866084569618757c01c972089b127a52e8586efcd368529
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - devtools
  - diptest
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# perturbation-response-error-comparison

## Summary

Benchmark strategy for comparing scDecouple's maximum-likelihood cellular-response estimates against the conventional cluster-and-map approach in simulated scCRISPR-seq data, quantifying estimation error relative to known ground-truth responses across varying covariance angles, cluster distances, and gRNA infection efficiencies.

## When to use

Use when you must quantitatively compare perturbation-effect estimation methods (e.g. scDecouple vs clustering-and-mapping) on synthetic cell-by-gene count matrices with known ground-truth cellular responses, sweeping simulation parameters such as covariance matrix angle, cluster distance, and infection efficiency.

## When NOT to use

- Data are real scCRISPR-seq experiments without known ground-truth cellular responses — error-against-truth comparison is impossible; use real-data pathway/gene-ranking evaluation instead.
- You only need to run scDecouple on one dataset without a competing baseline — this skill is specifically a method-vs-method benchmark, not a single-analysis pipeline.

## Inputs

- Synthetic perturbation-group cell-by-gene count matrix Y
- Synthetic control-group cell-by-gene count matrix Z
- Ground-truth simulation values of cellular response for each setting
- Simulation parameter grid: covariance matrix angles, inter-cluster distances, gRNA infection efficiencies, cluster counts (2 or 4), number of gRNAs (e.g. 10)

## Outputs

- scdecouple_vs_conventional_errors.csv with per-setting estimation errors for both methods
- Aggregated error comparison showing scDecouple achieves lower and more stable errors than the conventional clustering-and-mapping strategy

## How to apply

Generate synthetic perturbation-group (Y) and control-group (Z) cell-by-gene count matrices: a two-cluster scenario sweeping covariance matrix angles, inter-cluster distances, and gRNA infection efficiencies (100 runs per setting), plus a four-cluster scenario with 10 gRNAs (50 runs per setting). Run scDecouple on each dataset via run_scDecouple(Y, Z) with default parameters (using_rank=TRUE, top.sdev=30, top.multimodal=3, degenes.num=1500, seed_=0). Implement the conventional strategy by clustering the simulated cells (e.g. Seurat clustering on the control group), annotating clusters, and mapping perturbation effects through cluster centers. For every setting and both methods, compute the estimation error of the inferred cellular response against the known ground-truth simulation values. Aggregate errors per setting and write the per-method error metrics to scdecouple_vs_conventional_errors.csv, then verify scDecouple shows lower and less variable errors than the conventional approach.

## Related tools

- **scDecouple** (Estimates cellular response from Y and Z via Gaussian mixture modeling and EM (run_scDecouple), the method under evaluation against the conventional baseline) — https://github.com/MengQiuchen/scDecouple
- **Seurat** (Clustering of simulated control-group cells to implement the conventional cluster-annotation-and-mapping perturbation effect estimation strategy)
- **tidyverse** (R data handling and aggregation of per-setting error metrics)
- **devtools** (Installs the developmental scDecouple package from GitHub (devtools::install_github))

## Examples

```
res <- run_scDecouple(Y, Z)
```

## Evaluation signals

- scdecouple_vs_conventional_errors.csv exists and contains an error entry for every simulated setting (all covariance angles, cluster distances, infection efficiencies, and the four-cluster/10-gRNA scenario).
- Across 100 runs per two-cluster setting and 50 runs per four-cluster setting, scDecouple errors are lower in mean than the conventional strategy's errors.
- scDecouple error distributions are less variable (more stable) across replicate runs than the conventional clustering-and-mapping errors, qualitatively matching the reported Figure 3D-G finding.
- Both methods were run on identical simulated Y and Z matrices per setting, so errors are directly comparable against the same ground truth.
- Default scDecouple parameters (using_rank=TRUE, top.sdev=30, top.multimodal=3, degenes.num=1500, seed_=0) were used consistently across all simulation settings.

## Limitations

- scDecouple assumes gene expression in perturbed cells follows a Gaussian mixture, which may not hold in intricate systems or disease tissues such as tumors.
- scDecouple assumes the same cell clusters exist in NT and perturbation groups, which fails when rare cell types are completely absent from infection.
- The assumption of similar cellular responses across cell clusters limits scDecouple to cells with moderate heterogeneity.
- Simulation conclusions are specific to two- and four-cluster scenarios with the swept parameter ranges; behavior outside these regimes is not established.

## Evidence

- [results] scDecouple consistently showed more stable estimation results and lower errors than the conventional clustering-and-mapping strategy.: [evidence span withheld — anchor status: not_found]
- [methods] run_scDecouple(Y, Z) with default parameters: [evidence span withheld — anchor status: not_found]
- [readme] install the developmental version from GitHub: "Users can install the developmental version from [GitHub](https://github.com/MengQiuchen/scDecouple)"
- [abstract] scDecouple decouples true cellular response from infected proportion bias: "We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias."
