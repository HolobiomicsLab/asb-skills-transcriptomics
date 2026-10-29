---
name: pathway-activity-quantification-from-rnaseq
description: Use when you have a cpm-normalized gene-by-sample expression matrix with
  sample labels (e.g., CF vs healthy donors) and need to identify which gene sets
  show significantly shifted or over-dispersed activity between conditions.
when_to_use_negative:
- Data are not normalized — rRoma requires normalized input; the README warns 'rRoma
  works with normalized data' and cpm from edgeR was used for normalisation
- The goal is differential expression of individual genes rather than coordinated
  gene set/module activity quantification
- You need a complete statistical power assessment from a single small dataset — the
  authors state the CF case study 'would require us to include additional transcriptional
  dataset... and increase the statistical power of our analyses'
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3170
tools:
- name: rROMA
  role: Core R package that computes module activity via PC1 (L1 and Median Exp statistics)
    against random gene set nulls
  repo: github.com/sysbio-curie/rROMA
- name: irlba
  role: Efficiently computes only the first principal component (first column of V
    and first value of S) for each gene set
- name: edgeR
  role: Provides cpm normalization of the raw count matrix before running rRoma.R
- name: mice
  role: Imputes missing values in the global expression matrix via iterative low-rank
    PCA when needed
- name: DESeq2
  role: Alternative vst normalization option mentioned by the article
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:e9993a88462b8c0048d9d0afc6e0d60c0071aa940755721aa72d8f3d6b7d8c38
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - R
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pathway-activity-quantification-from-rnaseq

## Summary

Quantify gene set (module) activity from a normalized RNA-seq expression matrix using the rROMA R package, which computes PC1-based statistics (L1 overdispersion and Median Exp shift) against random gene set null distributions to detect shifted pathways, as demonstrated on the GSE176121 cystic fibrosis airway epithelial dataset with MSigDB Hallmark gene sets.

## When to use

Use when you have a cpm-normalized gene-by-sample expression matrix with sample labels (e.g., CF vs healthy donors) and need to identify which gene sets show significantly shifted or over-dispersed activity between conditions.

## When NOT to use

- Data are not normalized — rRoma requires normalized input; the README warns 'rRoma works with normalized data' and cpm from edgeR was used for normalisation
- The goal is differential expression of individual genes rather than coordinated gene set/module activity quantification
- You need a complete statistical power assessment from a single small dataset — the authors state the CF case study 'would require us to include additional transcriptional dataset... and increase the statistical power of our analyses'

## Inputs

- Normalized gene expression matrix (genes as rows, samples as columns; cpm from edgeR or vst from DESeq2), e.g. MatData
- Sample labels dataframe with sample_id and condition Type, e.g. sample_labels
- Gene set module list, e.g. 50 HALLMARK sets from MSigDB v6.2 via SelectFromMSIGdb or a GMT file

## Outputs

- rRoma.output$ModuleMatrix table with per-module PC1-based L1 and Median Exp statistics and their ppv p-values
- List of shifted pathways (ppv Median Exp <= 0.05) and over-dispersed non-shifted pathways (ppv L1 <= 0.05)
- Per-sample module activity scores for plotting (Plot.Genesets.Samples, Plot.Genesets.vs.Sampled)
- Gene weight table of top contributors per module (GetTopContrib), exportable as CSV

## How to apply

Start with normalized data (cpm from edgeR is what the article used; vst from DESeq2 also acceptable), genes as rows and samples as columns, and load sample labels to define the grouping (e.g., CF vs non-CF from sample_labels$Type). Retrieve the module list with SelectFromMSIGdb("HALLMARK") (MSigDB v6.2, 50 gene sets), stripping the 'HALLMARK_' prefix from names for plotting, or load a custom GMT with ReadGMTFile. Run rRoma.R(MatData, AllHall) with a fixed random seed; the algorithm centers the global matrix, imputes missing values with mice via low-rank PCA, and computes PC1 using the irlba package, comparing observed L1 and Median Exp statistics to random gene set null distributions (nSamples = 100 by default, built on a log-spaced grid of gene set sizes via ApproxSample). From rRoma.output$ModuleMatrix, call modules shifted when 'ppv Median Exp' <= 0.05 and over-dispersed-but-not-shifted when 'ppv L1' <= 0.05 with 'ppv Median Exp' > 0.05, ignoring q values for small sample and module counts. Visualize sample scores with Plot.Genesets.Samples (GroupInfo = Type, cluster_cols = TRUE) and Plot.Genesets.vs.Sampled (Plot = 'PC1Median'), and rank top-contributing genes with GetTopContrib (nGenes = 0.1, OrderType = 'Abs', Mode = 'Wei', Plot = TRUE).

## Related tools

- **rROMA** (Core R package that computes module activity via PC1 (L1 and Median Exp statistics) against random gene set nulls) — github.com/sysbio-curie/rROMA
- **irlba** (Efficiently computes only the first principal component (first column of V and first value of S) for each gene set)
- **edgeR** (Provides cpm normalization of the raw count matrix before running rRoma.R)
- **mice** (Imputes missing values in the global expression matrix via iterative low-rank PCA when needed)
- **DESeq2** (Alternative vst normalization option mentioned by the article)

## Examples

```
AllHall <- SelectFromMSIGdb("HALLMARK"); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- On GSE176121 CF vs healthy with 50 Hallmark sets, the shifted list reproduces FATTY_ACID_METABOLISM and APICAL_SURFACE (3 shifted genesets selected)
- The over-dispersed non-shifted list contains exactly COAGULATION (1 geneset selected)
- The gene-weight table lists GSN (weight ~15.6) as the top contributor of the COAGULATION module
- Shifted modules satisfy 'ppv Median Exp' <= 0.05; over-dispersed modules satisfy 'ppv L1' <= 0.05 with 'ppv Median Exp' > 0.05
- ModuleMatrix is non-empty for all 50 tested Hallmark gene sets with both ppv L1 and ppv Median Exp columns populated

## Limitations

- Authors state the CF case study lacks statistical power and would require additional transcriptional datasets and testing of several gene set reference databases
- Default values of hyperparameters (extreme-weight percentage, outlier z-score threshold, number of PCs, distance threshold, ApproxSample) are not reported
- rRoma depends on Bioconductor-only packages (scater, biomaRt); mice is needed only if the dataset contains missing values, and the authors recommend using data with missing values already imputed
- Hallmark was chosen for explanatory purpose; a more complete view requires combining multiple databases
- Q-values should be ignored in small-sample, small-module-count settings in favor of raw ppv thresholds

## Evidence

- [readme] Warning: rRoma works with normalized data. Here cpm from edgeR was used for normalisation: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [readme] SelectFromMSIGdb HALLMARK from MSigDB v6.2: "AllHall <- SelectFromMSIGdb("HALLMARK")"
- [intro] rROMA distinguishes shifted and over-dispersed gene sets via PC1 significance: "estimate the statistical significance of the distribution of samples along the first component for a gene set in two ways: it distinguishes between shifted and over-dispersed gene sets"
- [discussion] 3 of 50 hallmark pathways significantly active in CF: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [discussion] Missing values imputed via mice and low-rank approximation: "they can be imputed using an approximation of the data matrix with missing values by a full matrix of lower rank."
- [discussion] rROMA uses irlba to compute only the first principal component: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component, by computing only the first column of V and the first value of S."
- [discussion] Null distributions built on log-spaced grid of gene set sizes: "rROMA constructs null distributions for a representative list of gene set sizes. These are selected to be uniformly distributed in the logarithmic scale"
- [readme] Run rRoma.R on MatData and AllHall with a fixed seed: "rRoma.output <- rRoma.R(MatData, AllHall)"
