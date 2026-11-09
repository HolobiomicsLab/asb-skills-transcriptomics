---
name: bulk-rnaseq-pathway-activity-scoring
description: Use when a bulk RNA-seq normalized expression matrix (e.g. edgeR cpm)
  needs gene-set/module activity scores, and you must choose or benchmark among rROMA,
  GSVA, PLAGE, ssGSEA and z-score for differential pathway activity testing.
when_to_use_negative:
- The expression matrix is not normalized — rRoma explicitly requires normalized data
  (cpm/vst), so raw counts must first be normalized
- The dataset contains unimputed missing values — the authors recommend using datasets
  for which missing values have already been imputed rather than relying on the mice
  fallback
- The goal is a comprehensive view of biological systems with a single small gene-set
  collection — the README recommends combining multiple databases rather than relying
  on Hallmark alone
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0605
tools:
- name: rROMA
  role: 'Primary method: computes gene-set activity via PC1 (SVD/irlba) with shift/over-dispersion
    significance testing and outlier gene filtering'
  repo: github.com/sysbio-curie/rROMA
- name: GSVA
  role: Baseline sample-wise pathway scoring method used for benchmark comparison
    (also provides PLAGE and ssGSEA algorithms)
- name: edgeR
  role: 'Normalization: cpm transformation of RNA-seq counts before scoring'
- name: DESeq2
  role: Alternative normalization via vst, mentioned as acceptable for rRoma input
- name: mice
  role: Iterative PCA-based imputation of missing values in the global expression
    matrix
- name: irlba
  role: Efficient computation of the first principal component (first column of V,
    first value of S)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:8f4e236718d0daf127152f15576c400b777340611c5356b84cd81cb31e3e2027
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - GSVA
  - R
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bulk-rnaseq-pathway-activity-scoring

## Summary

Benchmarks rROMA against sample-wise pathway quantification methods (GSVA, PLAGE, ssGSEA, z-score) on bulk RNA-seq data (GSE176121, MSigDB Hallmark gene sets), showing rROMA's PC1-based module activity scores most consistently recover expected pathway shifts, e.g. TNFα signaling in IL17+TNFα-treated airway epithelial cultures.

## When to use

Use when a bulk RNA-seq normalized expression matrix (e.g. edgeR cpm) needs gene-set/module activity scores, and you must choose or benchmark among rROMA, GSVA, PLAGE, ssGSEA and z-score for differential pathway activity testing.

## When NOT to use

- The expression matrix is not normalized — rRoma explicitly requires normalized data (cpm/vst), so raw counts must first be normalized
- The dataset contains unimputed missing values — the authors recommend using datasets for which missing values have already been imputed rather than relying on the mice fallback
- The goal is a comprehensive view of biological systems with a single small gene-set collection — the README recommends combining multiple databases rather than relying on Hallmark alone

## Inputs

- Normalized bulk RNA-seq expression matrix (genes as rows, samples as columns; e.g. edgeR cpm), such as Rehman_CF_NCF_RNA_exp_matrix_norm.rda or Rehman_IL17_TNFa_RNA_exp_matrix_norm.rda
- Sample labels / grouping information defining the contrast (e.g. Rehman_CF_NCF_sample_labels.rda, Rehman_IL17_TNFa_sample_labels.rda)
- Gene set database: MSigDB Hallmark sets (v6.2 via SelectFromMSIGdb, or GMT file such as h.all.v2023.1.Hs.symbols.gmt)

## Outputs

- rRoma ModuleMatrix with L1 and Median Exp p/q values per module and sample-wise module activity scores
- Sample-wise pathway score matrices from GSVA, PLAGE, ssGSEA and z-score for the same 50 gene sets
- Comparison table: number of dysregulated pathways per method (scenario 1) and rank of the TNFa signaling pathway per method (scenario 2)

## How to apply

Start from a normalized expression matrix (cpm from edgeR in the paper; vst from DESeq2 is an accepted alternative) with genes as rows and samples as columns, plus sample labels defining the comparison groups. Retrieve the 50 MSigDB Hallmark gene sets with SelectFromMSIGdb('HALLMARK') (MSigDB v6.2) and strip the 'HALLMARK_' prefix; a GMT file can also be loaded via ReadGMTFile. Run rRoma.R with a fixed seed and recommended settings (PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, GeneOutDetection='L1OutExpOut', nSamples>=100) to obtain the ModuleMatrix with L1 and Median Exp p/q values; call modules shifted at 'ppv Median Exp' <= 0.05. Compute sample-wise scores for the same gene sets with the GSVA package using all four methods on the identical matrix, then test differential activity between groups (scenario 1: CF vs non-CF) and rank the biologically expected pathway (TNFa signaling in scenario 2: IL17+TNFa vs control) among all Hallmark sets by each method's significance. Rationale: rROMA quantifies gene-set activity as the maximum one-dimensional variance captured by PC1 via SVD, distinguishing shifted from over-dispersed sets, which proved more specific than the compared methods.

## Related tools

- **rROMA** (Primary method: computes gene-set activity via PC1 (SVD/irlba) with shift/over-dispersion significance testing and outlier gene filtering) — github.com/sysbio-curie/rROMA
- **GSVA** (Baseline sample-wise pathway scoring method used for benchmark comparison (also provides PLAGE and ssGSEA algorithms))
- **edgeR** (Normalization: cpm transformation of RNA-seq counts before scoring)
- **DESeq2** (Alternative normalization via vst, mentioned as acceptable for rRoma input)
- **mice** (Iterative PCA-based imputation of missing values in the global expression matrix)
- **irlba** (Efficient computation of the first principal component (first column of V, first value of S))

## Examples

```
library(rRoma); AllHall <- SelectFromMSIGdb("HALLMARK"); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Scenario 1 (CF vs non-CF): rROMA finds 3/50 dysregulated Hallmark pathways (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION), while GSVA/ssGSEA/z-score find 0 and PLAGE finds 45/50 — a benchmark run should reproduce this pattern
- Scenario 2 (IL17+TNFa vs control): TNFa signaling should rank 5/24 (GSVA), 5/27 (PLAGE), 7/17 (ssGSEA), 18/45 (z-score), and first (top-ranked) with rROMA
- Modules are called significantly shifted when 'ppv Median Exp' <= 0.05 in the rRoma ModuleMatrix
- The expression matrix is verified as normalized (e.g. cpm) before running rRoma.R, and the seed is fixed for reproducible permutation-based p-values
- Sample-wise score matrices from all five methods are computed on the identical normalized matrix and identical 50 gene sets, enabling a like-for-like comparison table

## Limitations

- The CF case study lacks statistical power; the authors state it 'would require us to include additional transcriptional dataset to take more samples into account and increase the statistical power of our analyses'
- PLAGE was highly non-specific in scenario 1 (45/50 pathways dysregulated), while GSVA/ssGSEA/z-score found none — sample-wise methods may fail to reproduce expected biology in disease-contrast scenarios
- Default rROMA hyperparameters (PCSignMode, PCSignThr, GeneOutDetection, nSamples) were tuned for these benchmarks; other datasets may need re-tuning, and gene-set size affects which methods yield testable rankings (e.g. only 17–45 sets remained significant per method in scenario 2)

## Evidence

- [readme] Here cpm from edgeR was used for normalisation, but you could also use vst from DeSeq2 or any other method you deem interesting.: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation, but you could also use vst from DeSeq2 or any other method you deem interesting."
- [readme] AllHall <- SelectFromMSIGdb("HALLMARK"): "## [1] "Searching in MsigDB v6.2"
## [1] "The following genesets have been selected:"
##  [1] "HALLMARK_TNFA_SIGNALING_VIA_NFKB (200 genes)""
- [discussion] out of the 50 hallmark pathways tested, 3 were significantly active: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [readme] Repository contaning data and codes to reproduce the comparison of sample-wise methods in the rRoma article.: [evidence span withheld — anchor status: external_doc]
- [readme] you can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed: "You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed"
