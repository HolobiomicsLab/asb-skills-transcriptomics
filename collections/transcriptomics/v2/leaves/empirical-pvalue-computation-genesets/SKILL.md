---
name: empirical-pvalue-computation-genesets
description: Use when quantifying significance of coordinated gene-set expression
  in a normalized expression matrix, and needing to classify gene sets as shifted
  (Median Exp) or overdispersed (L1) against random-set nulls.
when_to_use_negative:
- Input is raw (unnormalized) count data — rRoma requires normalized expression (e.g.
  edgeR cpm or DESeq2 vst) before PC1 and null-distribution statistics are meaningful
- The goal is differential expression per gene rather than gene-set/module-level activity
  quantification — this skill only tests coordinated gene-set behavior
- Expression data contain unimputed missing values and mice imputation is not desired
  — README recommends using datasets for which missing values have already been imputed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: rROMA
  role: Implements the SVD-based PC1 computation, random gene-set null generation,
    empirical ppv p-value and BH correction (rRoma.R, ModuleMatrix output)
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Computes only the first principal component (first column of V and first value
    of S) for the L1 and Median Exp statistics
- name: mice
  role: Imputes missing values in the global expression matrix via iterative PCA before
    significance testing
- name: R
  role: Runtime environment for the rRoma package and its workflow functions (SelectFromMSIGdb,
    rRoma.R)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:c861fc588c23e2bd242d24af729824b1e2be34ebd4658fe4671aaffe61161b9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - R
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Empirical p-value computation for gene-set significance (shifted vs overdispersed)

## Summary

rROMA assesses gene-set activity significance empirically: for each module it computes the L1 statistic (% variance explained by PC1) and Median Exp (median of sample projections on PC1) via SVD-based PCA, then compares them against null distributions built from randomly generated gene sets of matched size.

## When to use

Use when quantifying significance of coordinated gene-set expression in a normalized expression matrix, and needing to classify gene sets as shifted (Median Exp) or overdispersed (L1) against random-set nulls.

## When NOT to use

- Input is raw (unnormalized) count data — rRoma requires normalized expression (e.g. edgeR cpm or DESeq2 vst) before PC1 and null-distribution statistics are meaningful
- The goal is differential expression per gene rather than gene-set/module-level activity quantification — this skill only tests coordinated gene-set behavior
- Expression data contain unimputed missing values and mice imputation is not desired — README recommends using datasets for which missing values have already been imputed

## Inputs

- Normalized gene expression matrix with row names = gene names and column names = sample names (e.g. edgeR cpm or DESeq2 vst normalized counts, MatData)
- Module/gene-set list loaded from a GMT file or MSigDB (e.g. AllHall from SelectFromMSIGdb("HALLMARK"))
- Sample labels (e.g. sample_labels with CF vs non-CF type)
- A fixed random seed for reproducible null sampling

## Outputs

- ModuleMatrix table with per-module L1, Median Exp, empirical ppv p-values (ppv L1, ppv Median Exp) and BH-adjusted q values (q L1, q Median Exp)
- Binary classification per module: shifted (ppv Median Exp <= 0.05) or overdispersed (ppv L1 <= 0.05)

## How to apply

First center the global expression matrix (genes as rows, samples as columns) so every gene has mean zero, as required before rROMA's PCA interpretation; impute missing values via the mice package's iterative PCA method if needed. For each module (e.g. the 50 MSigDB v6.2 Hallmark sets loaded with SelectFromMSIGdb("HALLMARK")), extract compatible genes and compute PC1 via SVD using the R irlba package (PCADims = 2), deriving the L1 statistic and Median Exp statistic. Generate null distributions by sampling nSamples = 100 random gene sets matched in size, sharing background sets across modules whose sizes differ by less than the ApproxSamples percentage (representative sizes are uniformly distributed on a log scale between the minimum and maximum module size). Compute empirical ppv p-values — ppv L1 for overdispersion and ppv Median Exp for shift — by comparing each module's observed statistics to its null, then apply Benjamini-Hochberg correction to obtain q L1 and q Median Exp. Classify a module as shifted if 'ppv Median Exp' <= 0.05 and as overdispersed if 'ppv L1' <= 0.05, assembling all statistics into the ModuleMatrix table.

## Related tools

- **rROMA** (Implements the SVD-based PC1 computation, random gene-set null generation, empirical ppv p-value and BH correction (rRoma.R, ModuleMatrix output)) — https://github.com/sysbio-curie/rROMA
- **irlba** (Computes only the first principal component (first column of V and first value of S) for the L1 and Median Exp statistics)
- **mice** (Imputes missing values in the global expression matrix via iterative PCA before significance testing)
- **R** (Runtime environment for the rRoma package and its workflow functions (SelectFromMSIGdb, rRoma.R))

## Examples

```
set.seed(69); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Each module's ppv L1 and ppv Median Exp values lie in [0,1] and are derived from comparison against nSamples = 100 random gene sets matched in size
- BH-corrected q L1 and q Median Exp are present alongside raw ppv values in the ModuleMatrix
- On MatData with the 50 MSigDB v6.2 Hallmark sets, the run reproduces 3 significant pathways at p < 0.05: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION
- Random gene sets are matched to module sizes, with background sets shared across modules whose sizes differ by less than the ApproxSamples percentage (log-spaced grid of representative sizes)
- Fixing the seed before rRoma.R yields reproducible ppv p-values across runs

## Limitations

- Default values of hyperparameters such as ApproxSample are not reported in the article; users must inspect package defaults
- No software version stated for R, irlba, or mice packages, so exact reproducibility of ppv values may vary
- The CF case study was stated by the authors to lack statistical power and would require additional transcriptional datasets and testing of several gene set reference databases
- README recommends using datasets for which missing values have already been imputed rather than relying on mice imputation within rRoma

## Evidence

- [discussion] uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution.: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1"
- [discussion] rROMA constructs null distributions for a representative list of gene set sizes. These are selected to be uniformly distributed in the logarithmic scale: "rROMA constructs null distributions for a representative list of gene set sizes. These are selected to be uniformly distributed in the logarithmic scale"
- [discussion] out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION.: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [intro] estimate the statistical significance of the distribution of samples along the first component for a gene set in two ways: it distinguishes between shifted and over-dispersed gene sets: "estimate the statistical significance of the distribution of samples along the first component for a gene set in two ways"
- [readme] rRoma requires a gene expression matrix, with column names indicating samples and row names indicating gene names.: "rRoma requires a gene expression matrix, with column names indicating samples and row names indicating gene names."
