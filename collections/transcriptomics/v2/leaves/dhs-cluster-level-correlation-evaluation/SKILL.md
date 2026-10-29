---
name: dhs-cluster-level-correlation-evaluation
description: Use when you have BIRD-predicted DH plus measured DNase-seq signal for
  the same cell types and need to quantify prediction accuracy at pathway/DHS-cluster
  level (rC, fraction of clusters with rC>0.5/0.75) rather than per-locus.
when_to_use_negative:
- You need locus-level (per-DHS) prediction accuracy rather than pathway/cluster-level
  aggregation — use BIRD's locus-level model (-l option) and per-locus correlation
  instead
- No measured DNase-seq signal exists for the test cell type, since rC requires matching
  predicted against measured DH for the same genomic clusters
- The input is raw RNA-seq reads (FASTQ/BAM) rather than a gene expression FPKM matrix
  — expression must first be quantified (e.g., Tophat/Cufflinks or HISAT2/StringTie)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BIRD_predict
  role: Generates the predicted genome-wide DH values (log2(x+1) matrix) from the
    FPKM expression matrix that are correlated against measured DNase-seq per DHS
    cluster
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD
  role: Prediction framework whose locus-level (BIRD) vs pathway-level (BIR(X-bar,
    Y-bar)) outputs are compared via rC statistics
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD-data
  role: Supplies the preprocessed DNase-seq (chromatin accessibility) and gene expression
    training/test data used as the measured reference for rC computation
  repo: https://github.com/WeiqiangZhou/BIRD-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:cf3921f02596613d4f84e0578ebe3f4756205420d39e49651dea166b532373ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD-data
  - BIRD_predict
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# dhs-cluster-level-correlation-evaluation

## Summary

Evaluates gene-expression-based prediction of DNase I hypersensitivity (DH) at pathway level: measured and predicted DH vectors are matched across DHSs aggregated into co-activated clusters (1000/2000/5000 resolution), and per-cluster correlation coefficients (rC) between predicted and measured DH are computed to test whether pathway-level prediction beats locus-level BIRD.

## When to use

Use when you have BIRD-predicted DH plus measured DNase-seq signal for the same cell types and need to quantify prediction accuracy at pathway/DHS-cluster level (rC, fraction of clusters with rC>0.5/0.75) rather than per-locus.

## When NOT to use

- You need locus-level (per-DHS) prediction accuracy rather than pathway/cluster-level aggregation — use BIRD's locus-level model (-l option) and per-locus correlation instead
- No measured DNase-seq signal exists for the test cell type, since rC requires matching predicted against measured DH for the same genomic clusters
- The input is raw RNA-seq reads (FASTQ/BAM) rather than a gene expression FPKM matrix — expression must first be quantified (e.g., Tophat/Cufflinks or HISAT2/StringTie)

## Inputs

- Gene expression FPKM data matrix with gene ensembl ids in the first column and per-sample expression columns (17 test cell types in the study)
- Prebuilt BIRD prediction model file (e.g., human_hg38_model.bin)
- Measured DNase I hypersensitivity (DNase-seq) signal per cell type from the BIRD-data repository
- DHS cluster partition of co-activated regulatory element pathways at 1000, 2000, and 5000 cluster resolutions

## Outputs

- Predicted DH data matrix in log scale (log2(x+1) transformed) per test sample (optionally WIG format via -w/-wt)
- Per-cluster correlation coefficients (rC) between predicted and measured DH at 1000, 2000, and 5000 cluster resolutions
- Summary results (rc_evaluation_summary.csv): mean rC per model/clustering resolution and fraction of clusters exceeding rC = 0.5 and rC = 0.75

## How to apply

Load the gene expression FPKM data matrices for the test cell types (17 test cell types in the study), with gene ensembl ids in the first column, from the BIRD-data repository. Run BIRD_predict with a prebuilt model (e.g., human_hg38_model.bin) to generate predicted DH values per sample; the program handles ensembl id matching (version suffixes ignored by default, -e for exact match) and robust quantile normalization, and outputs predictions in log scale (log2(x+1)). Match each predicted DH vector to the measured DNase I hypersensitivity signal of the corresponding test cell type at the same genomic clusters, evaluated at 1000, 2000, and 5000 DHS-cluster resolutions. Compute per-cluster correlation coefficients (rC) between predicted and measured DH across all clusters, then aggregate into mean rC per model/clustering resolution and the fraction of clusters exceeding rC = 0.5 and rC = 0.75. Compare the pathway-level model BIR(X-bar, Y-bar) against locus-level BIRD at each resolution to test whether clustering co-activated DHSs improves cross-cell-type prediction accuracy.

## Related tools

- **BIRD_predict** (Generates the predicted genome-wide DH values (log2(x+1) matrix) from the FPKM expression matrix that are correlated against measured DNase-seq per DHS cluster) — https://github.com/WeiqiangZhou/BIRD
- **BIRD** (Prediction framework whose locus-level (BIRD) vs pathway-level (BIR(X-bar, Y-bar)) outputs are compared via rC statistics) — https://github.com/WeiqiangZhou/BIRD
- **BIRD-data** (Supplies the preprocessed DNase-seq (chromatin accessibility) and gene expression training/test data used as the measured reference for rC computation) — https://github.com/WeiqiangZhou/BIRD-data

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Mean rC for the pathway-level model BIR(X-bar, Y-bar) at 1000 clusters reproduces the reported 0.71 versus 0.50 for locus-level BIRD
- Approximately 84% and 55% of clusters exceed rC = 0.5 and rC = 0.75 respectively at 1000 clusters, with similar agreement at 2000 and 5000 clusters
- Pathway-level rC summary statistics at all three resolutions (1000, 2000, 5000) consistently exceed the locus-level BIRD baseline
- Predicted DH matrix is on the log2(x+1) scale and input file matching succeeded (input_file.match.txt produced, ensembl version suffixes ignored)
- Results are written to rc_evaluation_summary.csv with per-cluster and summary (mean rC, threshold fractions) fields

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample — evaluation is restricted to the fixed DHS cluster set
- Genomic information in the current version of BIRD is based on human genome hg19 (with hg38 models also released); measured DH must be on the matching reference genome
- Prediction performance depends on robust quantile normalization; large numbers of tied values in input data (e.g., single-cell RNA-seq) can affect results without the updated normalization

## Evidence

- [other] cross-cell-type DH variation of regulatory element pathways is predicted with substantially higher accuracy than that of individual loci: [evidence span withheld — anchor status: not_found]
- [readme] BIRD_predict output is log-transformed predicted DH: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [readme] BIRD_predict handles gene id matching internally: "BIRD will first match the ensembl id of the genes used for building the prediction model with the ids in the input data matrix. The matched data matrix will be outputted as input_file.match.txt"
- [readme] robust quantile normalization is applied to input expression data: "The quantile normalization function is updated to a more robust version (signfigantly boost prediction performance when there is a large number of tied values in the input data"
- [discussion] BIRD does not discover new DHS locations: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
