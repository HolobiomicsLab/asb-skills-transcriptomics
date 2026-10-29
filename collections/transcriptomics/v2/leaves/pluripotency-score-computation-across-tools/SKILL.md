---
name: pluripotency-score-computation-across-tools
description: Use when comparing runtime and memory footprint of pluripotency/potency
  scoring tools (ORIGINS activity, LandSCENT SR, CytoTRACE) on scRNA-seq count matrices
  from breast, colon, hematopoietic, or lung datasets.
when_to_use_negative:
- The goal is biological accuracy of pluripotency ranking rather than computational
  efficiency — the Table 1 benchmark only measures runtime and RAM, not score agreement
  with known biology
- The input is not a scRNA-seq expression matrix (e.g., bulk RNA-seq or single-cell
  ATAC-seq) — ORIGINS' activity() expects a gene-by-cell expression matrix
- You need PPINs for biological processes other than differentiation — the extended
  ORIGINS2 package, not ORIGINS, provides 18 process-specific PPINs and custom PPIN
  construction
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS
  role: Computes the differentiation activity score (and activity HVF variant) whose
    runtime and RAM usage are benchmarked; installed from GitHub via remotes
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Competing SR-based pluripotency scorer benchmarked for runtime and extra RAM
    (e.g., 6.4 Gb extra RAM on the breast dataset)
- name: CytoTRACE
  role: Competing potency scorer benchmarked for runtime and extra RAM (e.g., 5.8
    Gb extra RAM on the breast dataset)
- name: remotes
  role: R package used to install ORIGINS from its GitHub repository
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:9945c5c6ce0702bb3d304bed7c07fc443e6aad7bf5f67ba0217c3e7c08ee59e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - remotes
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pluripotency-score-computation-across-tools

## Summary

Reproduce the computational benchmark from Senra et al. (2022) comparing ORIGINS' differentiation activity score against LandSCENT (SR) and CytoTRACE across four GEO scRNA-seq datasets, recording elapsed runtime and extra RAM per tool per dataset (Table 1).

## When to use

Use when comparing runtime and memory footprint of pluripotency/potency scoring tools (ORIGINS activity, LandSCENT SR, CytoTRACE) on scRNA-seq count matrices from breast, colon, hematopoietic, or lung datasets.

## When NOT to use

- The goal is biological accuracy of pluripotency ranking rather than computational efficiency — the Table 1 benchmark only measures runtime and RAM, not score agreement with known biology
- The input is not a scRNA-seq expression matrix (e.g., bulk RNA-seq or single-cell ATAC-seq) — ORIGINS' activity() expects a gene-by-cell expression matrix
- You need PPINs for biological processes other than differentiation — the extended ORIGINS2 package, not ORIGINS, provides 18 process-specific PPINs and custom PPIN construction

## Inputs

- GEO scRNA-seq count matrices: GSE113197 (breast), GSE125970 (colon), GSE117498 (hematopoietic bone marrow), GSE161383 (lung); genes as rows, cells as columns
- differentiation_edges PPIN data frame bundled within the ORIGINS package
- Installed R packages: ORIGINS, LandSCENT, CytoTRACE

## Outputs

- runtime_memory_benchmark.csv with 16 tool-by-dataset rows containing elapsed runtime and extra RAM columns
- Per-cell differentiation activity vectors from ORIGINS activity and activity HVF
- Per-cell SR scores (LandSCENT) and CytoTRACE scores for comparison

## How to apply

Download the four GEO datasets (GSE113197 breast, GSE125970 colon, GSE117498 hematopoietic, GSE161383 lung) and load each as an expression_matrix with genes as rows and cells as columns. Install ORIGINS via remotes::install_github("danielasenraoka/ORIGINS") alongside the published LandSCENT and CytoTRACE packages in R. For each dataset, run SR (LandSCENT), CytoTRACE, activity(expression_matrix, differentiation_edges), and activity HVF, wrapping each call in system.time() to capture elapsed time and using gc()/memory profiling to record extra RAM allocated. Assemble a 16-row tool-by-dataset table with runtime and RAM columns, using the reported reference values (breast: 3.43 h LandSCENT / 28.27 s CytoTRACE-adjacent runs / 3.00 min activity; extra RAM 6.4 Gb LandSCENT, 5.8 Gb CytoTRACE, 26.6 Kb for the ORIGINS activity vector). Validate by checking runtime_memory_benchmark.csv exists with all 16 rows populated and measured ORIGINS activity values within the same order of magnitude as reported.

## Related tools

- **ORIGINS** (Computes the differentiation activity score (and activity HVF variant) whose runtime and RAM usage are benchmarked; installed from GitHub via remotes) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Competing SR-based pluripotency scorer benchmarked for runtime and extra RAM (e.g., 6.4 Gb extra RAM on the breast dataset))
- **CytoTRACE** (Competing potency scorer benchmarked for runtime and extra RAM (e.g., 5.8 Gb extra RAM on the breast dataset))
- **remotes** (R package used to install ORIGINS from its GitHub repository)

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- runtime_memory_benchmark.csv exists and contains exactly 16 rows (4 tools x 4 datasets) with both runtime and RAM columns populated
- Measured ORIGINS activity values are within the same order of magnitude as the reported reference values
- On the breast dataset, extra RAM matches the reported pattern: LandSCENT ~6.4 Gb, CytoTRACE ~5.8 Gb, ORIGINS activity vector only ~26.6 Kb
- Runtime ordering is consistent with the paper: ORIGINS activity completes in minutes (e.g., ~3 min on breast) while LandSCENT takes hours (~3.43 h)
- Each expression_matrix is oriented with genes as rows and cells as columns before calling activity()

## Limitations

- Benchmark covers only four healthy human datasets (breast, colon, hematopoietic, lung); results may not transfer to disease or other tissue contexts
- The differentiation PPIN (differentiation_edges) is specific to the differentiation process; ORIGINS cannot score other biological processes — use ORIGINS2 for that
- RAM savings are partly structural: ORIGINS stores only a per-cell activity vector (~26.6 Kb), so the comparison favors it on memory rather than on score quality

## Evidence

- [supplementary] ORIGINS required far less RAM than competitors: LandSCENT needed an extra 6.4 Gb and CytoTRACE 5.8 Gb for the breast dataset, while ORIGINS only needed 26.6 Kb to store the activity vector.: [evidence span withheld — anchor status: not_found]
- [abstract] We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets"
- [methods] The remotes package is needed, if you don't have it type: install.packages("remotes"): "The remotes package is needed, if you don't have it type: install.packages("remotes")"
- [readme] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [methods] remotes::install_github("danielasenraoka/ORIGINS"): "remotes::install_github("danielasenraoka/ORIGINS")"
