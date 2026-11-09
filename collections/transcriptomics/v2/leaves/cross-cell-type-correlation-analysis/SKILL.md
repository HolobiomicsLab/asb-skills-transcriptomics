---
name: cross-cell-type-correlation-analysis
description: Use when you have DH profiles (predicted or imputed) for the same set
  of cell types from two methods and need genome-wide cross-locus and cross-cell-type
  agreement metrics, plus differential-DH comparison for similar cell type pairs.
when_to_use_negative:
- You only have DH profiles from a single method and no second method or ground truth
  to compare against
- Cell types have no shared genomic loci or are on incompatible genome builds without
  coordinate alignment
- You need to discover new DHS locations — BIRD prediction will not identify new DH
  sites and this comparison framework evaluates agreement only at fixed loci
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: BIRD
  role: Generates gene-expression-based DH predictions for each test cell type to
    be compared against imputed profiles
  repo: github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Runs prediction with human_hg38_model.bin on FPKM expression matrices, producing
    log2(x+1)-scaled DH matrices bounded at 14
  repo: github.com/WeiqiangZhou/BIRD
- name: R
  role: Used to align profiles on shared loci and compute cross-locus, cross-cell-type,
    and differential-DH agreement metrics
  repo: https://www.r-project.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:c797d2e1fd0276ca3c28482db53f0bfafc2b59d72a20582a4a986e6e65dd140d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cross-cell-type-correlation-analysis

## Summary

Compares predicted/imputed DNase I hypersensitivity (DH) profiles across multiple cell types by computing genome-wide cross-locus and cross-cell-type agreement (e.g., correlation) between methods such as BIRD (gene expression only) and ChromImpute (multiple functional genomic data types), plus differential-DH comparison for highly similar cell type pairs.

## When to use

Use when you have DH profiles (predicted or imputed) for the same set of cell types from two methods and need genome-wide cross-locus and cross-cell-type agreement metrics, plus differential-DH comparison for similar cell type pairs.

## When NOT to use

- You only have DH profiles from a single method and no second method or ground truth to compare against
- Cell types have no shared genomic loci or are on incompatible genome builds without coordinate alignment
- You need to discover new DHS locations — BIRD prediction will not identify new DH sites and this comparison framework evaluates agreement only at fixed loci

## Inputs

- BIRD-predicted DH matrices (log2(x+1) scale, upper bound 14) for each test cell type
- ChromImpute-imputed DH tracks for the same cell types
- FPKM gene expression data matrices with gene ensembl ids as the first column
- Shared genomic locus coordinates (hg19/hg38)

## Outputs

- Per-cell-type cross-locus agreement metrics (genome-wide correlation) for all 10 shared test cell types
- Per-locus cross-cell-type agreement metrics
- Differential-DH agreement per highly similar cell type pair under both methods
- Aggregated comparison table (e.g., bird_vs_chromimpute_comparison.csv)

## How to apply

Obtain predicted DH matrices for each test cell type (e.g., BIRD_predict with human_hg38_model.bin on FPKM expression matrices, output in log2(x+1) scale bounded at the default upper value of 14) and the comparison imputed DH tracks (e.g., ChromImpute). Align both profiles on shared genomic loci using hg19/hg38 coordinates in R. Compute genome-wide correlation/agreement per cell type for cross-locus comparison (within a cell type across loci) and cross-cell-type comparison (within a locus across cell types), and summarize all metrics in a results table. For differential analysis, identify highly similar cell type pairs, compute differential DH between pair members under both methods, and compare the methods' differential predictions — the paper found BIRD and ChromImpute comparable overall, with BIRD substantially better for differential DH between highly similar cell type pairs.

## Related tools

- **BIRD** (Generates gene-expression-based DH predictions for each test cell type to be compared against imputed profiles) — github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Runs prediction with human_hg38_model.bin on FPKM expression matrices, producing log2(x+1)-scaled DH matrices bounded at 14) — github.com/WeiqiangZhou/BIRD
- **R** (Used to align profiles on shared loci and compute cross-locus, cross-cell-type, and differential-DH agreement metrics) — https://www.r-project.org

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Comparison table reports cross-locus agreement for all 10 shared test cell types
- Comparison table reports cross-cell-type agreement for all 10 shared test cell types
- BIRD and ChromImpute show comparable performance with neither consistently outperforming the other
- Differential-DH comparison demonstrates BIRD's advantage for highly similar cell type pairs
- BIRD predictions are in log2(x+1) scale and bounded at the default upper value of 14

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample, so comparisons are restricted to loci in the training locus set
- Neither method consistently outperforms the other in overall prediction, so conclusions depend on the analysis target (overall vs differential)
- Current BIRD genomic information is based on human genome hg19 (with hg38 models available); comparisons require coordinate alignment across builds

## Evidence

- [results] BIRD vs ChromImpute comparable performance: [evidence span withheld — anchor status: not_found]
- [results] BIRD differential DH advantage: [evidence span withheld — anchor status: not_found]
- [methods] BIRD_predict invocation: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] bounded predictions and default upper value: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
- [readme] output in log2(x+1) scale: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [discussion] BIRD will not discover new DHS locations: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
