---
name: imputation-quality-evaluation
description: Use when raw scRNA-seq count data with ERCC spike-in transcripts of known
  concentrations are available and you need to verify that scImpute imputation recovers
  true spike-in expression rather than introducing bias.
when_to_use_negative:
- The dataset contains no ERCC spike-in transcripts with known concentrations, so
  imputation accuracy cannot be measured against ground truth
- The input is already normalized or previously imputed — scImpute is designed to
  be applied to the raw count matrix before downstream analyses
- Comparing imputation methods that intentionally alter all expression levels (e.g.
  MAGIC, SAVER), since bias-free recovery cannot be assessed by the same unchanged-value
  expectation
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scImpute
  role: Imputation method whose output quality is evaluated; run via its scimpute()
    function on raw counts with drop_thre = 0.5 before comparing ERCC correlations
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:49a572182a65415e35c773fe82763c6b69048b319c6014c10bd82a3df1e39a15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# imputation-quality-evaluation

## Summary

Quantifies how well an scRNA-seq imputation method (scImpute) recovers true expression masked by dropouts by correlating ERCC spike-in read counts against known concentrations before and after imputation, on the GSE60361 mouse somatosensory cortex dataset (3005 cells, 57 ERCC transcripts).

## When to use

Use when raw scRNA-seq count data with ERCC spike-in transcripts of known concentrations are available and you need to verify that scImpute imputation recovers true spike-in expression rather than introducing bias.

## When NOT to use

- The dataset contains no ERCC spike-in transcripts with known concentrations, so imputation accuracy cannot be measured against ground truth
- The input is already normalized or previously imputed — scImpute is designed to be applied to the raw count matrix before downstream analyses
- Comparing imputation methods that intentionally alter all expression levels (e.g. MAGIC, SAVER), since bias-free recovery cannot be assessed by the same unchanged-value expectation

## Inputs

- Raw scRNA-seq count matrix (rows = genes, columns = cells), e.g. GSE60361 mouse cortex with 3005 cells
- ERCC spike-in transcript concentration table (57 ERCC transcripts)

## Outputs

- scimpute_count.csv — imputed count matrix in out_dir with the same gene-by-cell dimensions as the raw input
- Per-cell Pearson correlation values between ERCC counts and known concentrations (raw vs imputed)
- Summary of median and minimum ERCC correlations pre- vs post-imputation

## How to apply

Load the raw count matrix (genes as rows, cells as columns, e.g. GSE60361 with 3005 cells and 57 ERCC transcripts) plus the ERCC spike-in concentration table into R. Run the `scimpute` function with type = "count", drop_thre = 0.5, Kcluster set to the known number of cell subpopulations, labeled = FALSE, and ncores for parallel computation; scImpute only imputes values with dropout probability larger than drop_thre. Read the resulting scimpute_count.csv from out_dir. Compute per-cell Pearson correlations between the 57 ERCC observed counts and their known concentrations for both the raw and imputed matrices, then summarize the median and minimum correlations. Judge success by reproduction of the reported improvement: median correlation rising from 0.92 to 0.95 and minimum from 0.81 to 0.89, with the imputed matrix retaining the raw gene-by-cell dimensions.

## Related tools

- **scImpute** (Imputation method whose output quality is evaluated; run via its scimpute() function on raw counts with drop_thre = 0.5 before comparing ERCC correlations) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = "raw_count.csv", infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- scimpute_count.csv exists in out_dir and has the same gene-by-cell dimensions as the raw input matrix
- Median per-cell ERCC count-vs-concentration Pearson correlation increases from 0.92 (raw) to 0.95 (imputed)
- Minimum per-cell ERCC correlation increases from 0.81 (raw) to 0.89 (imputed)
- Only values with dropout probability larger than drop_thre are changed; non-dropout values remain unaltered (scImpute introduces no new biases to the rest of the data)
- Kcluster matches the known number of cell subpopulations in the dataset

## Limitations

- The choice of K (number of cell populations) is left to the user without a definitive selection procedure; it can be chosen based on clustering of the raw data and the desired resolution
- Imputation efficiency degrades when dropout rates in raw data are severely high (droplet-based technologies); improving this is a stated future direction
- scImpute detects outlier cells and excludes them from imputation
- Applying scImpute on the whole-genome count matrix is recommended; a gene filtering step is acceptable but most genes should be present to ensure robust identification of dropouts
- No software version for scImpute is specified in the article

## Evidence

- [abstract] scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data.: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [methods] Only the values that have dropout probability larger than drop_thre are imputed by scImpute.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [methods] We highly recommend using parallel computing with scImpute, which will significantly reduce the computation time.: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
- [readme] Its only input is the raw count matrix with rows representing genes and columns representing cells.: "Its only input is the raw count matrix with rows representing genes and columns representing cells."
- [discussion] It can be chosen based on the clustering result of the raw data and the resolution level desired by the users: "It can be chosen based on the clustering result of the raw data and the resolution level desired by the users"
- [intro] Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
