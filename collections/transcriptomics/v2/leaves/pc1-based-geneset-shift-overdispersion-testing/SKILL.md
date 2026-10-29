---
name: pc1-based-geneset-shift-overdispersion-testing
description: Use when you have a normalized gene expression matrix (samples as columns,
  genes as rows) plus a module/GMT list (e.g. MSigDB HALLMARK) and need sample-level
  module activity scores with statistical shift/overdispersion significance.
when_to_use_negative:
- Input data is raw, unnormalized counts and cannot be normalized first — rRoma explicitly
  works only with normalized data
- Datasets contain missing values that have not been imputed — the README recommends
  using datasets with missing values already imputed rather than relying on the mice
  fallback
- The goal is differential expression per gene rather than coordinated gene set (module)
  activity estimation
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: rROMA
  role: Core R package computing PC1 module activity and shift/overdispersion significance
    tests (rRoma.R)
  repo: github.com/sysbio-curie/rROMA
- name: irlba
  role: Efficiently computes only the first principal component (first column of V,
    first value of S) for each gene set
- name: mice
  role: Iterative PCA-based imputation of missing values in the global expression
    matrix (FillNAMethod)
- name: edgeR
  role: cpm normalization of RNA-seq counts before running rROMA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:395bb4af146f9193cb07582c6f0d02dd69f3408d07240d632c1fef264db7a30c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# pc1-based-geneset-shift-overdispersion-testing

## Summary

Quantify gene set activity by computing PC1 over a gene set's expression and test whether the gene set is 'shifted' (median of sample projections) or 'over-dispersed' (L1 variance) versus random gene set nulls, using the rROMA R package.

## When to use

Use when you have a normalized gene expression matrix (samples as columns, genes as rows) plus a module/GMT list (e.g. MSigDB HALLMARK) and need sample-level module activity scores with statistical shift/overdispersion significance.

## When NOT to use

- Input data is raw, unnormalized counts and cannot be normalized first — rRoma explicitly works only with normalized data
- Datasets contain missing values that have not been imputed — the README recommends using datasets with missing values already imputed rather than relying on the mice fallback
- The goal is differential expression per gene rather than coordinated gene set (module) activity estimation

## Inputs

- Normalized gene expression matrix (genes as rows, samples as columns; e.g. cpm-normalized MatData)
- Sample labels with group identifiers (e.g. sample_labels with Type: CF vs non-CF)
- Module/gene set list from a GMT file or MSigDB (e.g. Msigdb.v6.2.symbols.all, 50 HALLMARK sets)

## Outputs

- rRoma.output$ModuleMatrix with PC1-based L1 overdispersion and Median Exp shift ppv statistics per module
- Sample-level module activity scores (PC1 projections) for shifted and overdispersed modules
- Gene-weight/top-contributor table per module (via GetTopContrib, saveable as CSV)

## How to apply

Normalize raw RNA-seq counts with cpm from edgeR (or vst from DESeq2), since rRoma requires normalized data and imputes missing values with mice. Load the module list via SelectFromMSIGdb("HALLMARK") or ReadGMTFile, optionally stripping prefixes like 'HALLMARK_' for plotting, and run rRoma.R(MatData, modules) with a fixed random seed. rROMA centers the data, computes the first principal component (via the irlba package) of each gene set, and builds null distributions by random gene set sampling (nSamples = 100, log-spaced ApproxSample grid) to compute ppv statistics for L1 (overdispersion) and Median Exp (shift). Classify modules as shifted when 'ppv Median Exp' <= 0.05, and as overdispersed-but-not-shifted when 'ppv L1' <= 0.05 with 'ppv Median Exp' > 0.05, ignoring q-values for small sample/module counts. Visualize sample scores with Plot.Genesets.Samples (GroupInfo = sample type) and rank top-contributing genes with GetTopContrib (nGenes = 0.1, OrderType = 'Abs', Mode = 'Wei').

## Related tools

- **rROMA** (Core R package computing PC1 module activity and shift/overdispersion significance tests (rRoma.R)) — github.com/sysbio-curie/rROMA
- **irlba** (Efficiently computes only the first principal component (first column of V, first value of S) for each gene set)
- **mice** (Iterative PCA-based imputation of missing values in the global expression matrix (FillNAMethod))
- **edgeR** (cpm normalization of RNA-seq counts before running rROMA)

## Examples

```
library(rRoma); AllHall <- SelectFromMSIGdb("HALLMARK"); set.seed(69); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- ModuleMatrix is non-empty and each tested module has both 'ppv L1' and 'ppv Median Exp' columns
- Shifted module selection (ppv Median Exp <= 0.05) on the GSE176121 CF vs healthy data reproduces FATTY_ACID_METABOLISM and APICAL_SURFACE (3 shifted genesets selected)
- Overdispersed-but-not-shifted selection (ppv L1 <= 0.05, ppv Median Exp > 0.05) contains exactly COAGULATION (1 geneset selected)
- Gene-weight table from GetTopContrib lists GSN (weight ~15.6) as the top COAGULATION contributor
- Random seed is fixed before rRoma.R so the random gene set null (nSamples = 100) is reproducible

## Limitations

- Authors state the CF case study lacks statistical power and would require additional transcriptional datasets to increase power
- Default values of hyperparameters (extreme-weight percentage, outlier z-score threshold, number of PCs, distance threshold, ApproxSample) are not reported
- PC1 sign ambiguity requires choosing an orientation method (gene signs or extreme-weight/expression multiplication)
- Single-database analyses (e.g. HALLMARK only) give an incomplete view; the README recommends combining multiple databases

## Evidence

- [intro] estimate the statistical significance of the distribution of samples along the first component for a gene set in two ways: it distinguishes between shifted and over-dispersed gene sets: "estimate the statistical significance of the distribution of samples along the first component for a gene set in two ways: it distinguishes between shifted and over-dispersed gene sets"
- [discussion] uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution"
- [methods] Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [discussion] out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION.: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [readme] rRoma uses the `mice` package to fill missing values. This package needs to be installed if analysed datasets contain missing values. However, we recommend using on datasets for which missing values have already been imputed.: "rRoma uses the `mice` package to fill missing values. This package needs to be installed if analysed datasets contain missing values. However, we recommend using on datasets for which missing values"
