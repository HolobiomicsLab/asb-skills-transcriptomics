---
name: pca-dimensionality-reduction-visualization
description: Use when after a differential-expression test on background-subtracted
  microarray data, you need a low-dimensional (PCA) visualization to confirm that
  samples cluster by response class using only the significant genes.
when_to_use_negative:
- The input data has not been background-subtracted and normalized via the upstream
  PIMENTo Preprocess Data step (quantile or LOESS)
- No response class vector is defined, so there is no class structure for the PCA
  to reveal
- The analysis target is RNA-seq rather than Illumina/Affymetrix-style microarray
  data processed by PIMENTo
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: R package providing Sig_GenesSAM and the downstream similarity/PCA plotting
    of samples on significant genes
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: samr
  role: R package supplying the Significance Analysis of Microarrays code used by
    Sig_GenesSAM to define the gene set fed into the PCA
- name: RStudio
  role: IDE in which the PIMENTo commands with autocomplete are run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:4d11e94bd695c2e058e0b9e2e11af3dd5acd90cbd2d52fa43a95ca985fd10dc1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - samr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pca-dimensionality-reduction-visualization

## Summary

Generate a principal component analysis (PCA) plot of microarray samples restricted to the significant genes identified by a SAM significance test (e.g. PIMENTo's Sig_GenesSAM run with fdr.cutoff 0.4), to visually assess separation between response classes such as wild-type vs. villin-gp130Act samples.

## When to use

Use when, after a differential-expression test on background-subtracted microarray data, you need a low-dimensional (PCA) visualization to confirm that samples cluster by response class using only the significant genes.

## When NOT to use

- The input data has not been background-subtracted and normalized via the upstream PIMENTo Preprocess Data step (quantile or LOESS)
- No response class vector is defined, so there is no class structure for the PCA to reveal
- The analysis target is RNA-seq rather than Illumina/Affymetrix-style microarray data processed by PIMENTo

## Inputs

- Background-subtracted microarray expression matrix (tab-separated, e.g. example_unnorm.txt preprocessed via PIMENTo Preprocess Data)
- Response class vector c(1,1,1,2,2,2) encoding sample groups (class 1 = wild-type, class 2 = villin-gp130Act)
- Significant-genes file exported by SigGenesSAM at fdr.cutoff 0.4

## Outputs

- PCA plot of samples computed on significant genes (Fig. 6)
- Companion Euclidean distance similarity plot of samples (Fig. 5)
- Tabular significant-genes output file

## How to apply

First run the significance test on the background-subtracted expression matrix (e.g. PIMENTo's Sig_GenesSAM, which wraps the R 'samr' package) with a response class vector such as c(1,1,1,2,2,2) encoding three wild-type and three villin-gp130Act samples, and an fdr.cutoff of 0.4 to select significant genes. Then compute a PCA on the samples using only those significant genes and plot the samples (Fig. 6 in the PIMENTo workflow), ideally alongside the Euclidean distance similarity plot of the same gene set (Fig. 5). The rationale is that dimensionality reduction on the filtered gene set should reveal class structure that was statistically detected by SAM; validation requires that wild-type and villin-gp130Act samples separate visually, consistent with the response vector. PIMENTo is an R package, so all steps run in R/RStudio with dependencies (affy, limma, DESeq2, samr) installed from CRAN and Bioconductor.

## Related tools

- **PIMENTo** (R package providing Sig_GenesSAM and the downstream similarity/PCA plotting of samples on significant genes) — https://www.github.com/MUSC-CGM/PIMENTo
- **samr** (R package supplying the Significance Analysis of Microarrays code used by Sig_GenesSAM to define the gene set fed into the PCA)
- **RStudio** (IDE in which the PIMENTo commands with autocomplete are run)

## Examples

```
install.packages("devtools"); source("https://bioconductor.org/biocLite.R"); biocLite(c("affy", "limma", "DESeq2")); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- The significant-genes file is populated at FDR 0.4 (non-empty SAM output)
- The PCA plot visually separates wild-type (class 1) and villin-gp130Act (class 2) samples consistent with the response vector c(1,1,1,2,2,2)
- The companion Euclidean distance matrix plot shows samples within the same response class are similar, while samples of different classes differ (Fig. 5)
- All commands execute in R/RStudio after installing PIMENTo and its dependencies (affy, limma, DESeq2) via devtools and Bioconductor

## Limitations

- No version numbers are stated for any software dependency (limma, affy, samr, R, Bioconductor)
- The analysis dataset (wild-type and villin-gp130Act small intestinal crypt expression profiles, Array Express accession E-MEXP-E-MTAB-2400) has incomplete accession/citation metadata; the samr citation appears as a '[ref]' placeholder
- PCA separation is judged visually in this workflow; no quantitative cluster-separation threshold is specified

## Evidence

- [discussion] Significance testing is carried out using the code for Significance Analysis of Microarrays from the R package “samr” [ref]: [evidence span withheld — anchor status: not_found]
