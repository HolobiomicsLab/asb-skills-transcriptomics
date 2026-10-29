---
name: clonal-diversity-index-computation
description: Use when a normalized barcode counts table (e.g. counts/all_counts_combined.tsv)
  is available and the goal is to test whether drug treatments (e.g. DMSO, IBET-151,
  AraC) or dose-escalation time courses differ in clonal composition or diversity.
when_to_use_negative:
- Input is raw sequencing data rather than a barcode counts table — first run the
  BARtab pipeline (FastQC, fastp filtering, cutadapt trimming, alignment/clustering)
  to generate counts
- Data are single-cell or spatial transcriptomics barcodes requiring lineage reconstruction
  rather than population-level diversity summarization
- Barcodes are not clonal synthetic barcodes but evolving lineages — the paper states
  these tools 'currently do not officially support evolving barcode methods'
tools:
- name: bartools
  role: R package providing the diversity-index functions (Shannon, Simpson, Gini)
    and visualization workflows applied to the normalized barcode counts table
  repo: https://github.com/DaneVass/bartools
- name: R
  role: Analysis environment in which PCA (prcomp), Pearson correlation, and hierarchical
    clustering are performed with tidyverse
- name: BARtab
  role: Upstream Nextflow pipeline that generates the barcode counts table feeding
    this diversity analysis
  repo: https://github.com/DaneVass/BARtab
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:5bc4727a034cc8e3246bac021f96242ed42cf056b450d960d2e64bca32384fbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - bartools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# clonal-diversity-index-computation

## Summary

Compute per-sample clonal diversity metrics (Shannon, Simpson, Gini) and global clonal composition analyses (PCA, pairwise replicate correlations, hierarchical clustering) from normalized barcode counts tables using bartools in R, to quantify whether treatments alter clonal composition.

## When to use

Use when a normalized barcode counts table (e.g. counts/all_counts_combined.tsv) is available and the goal is to test whether drug treatments (e.g. DMSO, IBET-151, AraC) or dose-escalation time courses differ in clonal composition or diversity.

## When NOT to use

- Input is raw sequencing data rather than a barcode counts table — first run the BARtab pipeline (FastQC, fastp filtering, cutadapt trimming, alignment/clustering) to generate counts
- Data are single-cell or spatial transcriptomics barcodes requiring lineage reconstruction rather than population-level diversity summarization
- Barcodes are not clonal synthetic barcodes but evolving lineages — the paper states these tools 'currently do not officially support evolving barcode methods'

## Inputs

- Normalized combined barcode counts table (counts/all_counts_combined.tsv) produced by BARtab
- Sample metadata mapping samples to treatment (DMSO, IBET-151, AraC), dose, and time point

## Outputs

- PCA ordination plot (PC1 vs PC2 colored by treatment)
- Per-sample Shannon, Simpson, and Gini diversity indices and diversity-vs-time-point trend plots
- Pairwise Pearson correlation matrix between biological replicates (DMSO and IBET-151)
- Hierarchical clustering dendrogram of samples based on barcode-composition dissimilarity

## How to apply

Load the normalized combined barcode counts table in R with tidyverse and normalize counts across samples. Run prcomp on log-normalized barcode counts and plot PC1 vs PC2 colored by treatment (DMSO, IBET-151, AraC) to check whether treatments separate along PC1 and the dose-escalation time course separates along PC2. Compute Shannon, Simpson, and Gini diversity indices per sample using bartools functions and plot diversity trends across time points. Compute pairwise Pearson correlations between biological replicates for DMSO and IBET-151 samples, verifying correlations exceed 0.95, and hierarchically cluster samples on correlation dissimilarity to confirm that AraC IC90 samples form a distinct cluster separate from gradual dose-escalation samples.

## Related tools

- **bartools** (R package providing the diversity-index functions (Shannon, Simpson, Gini) and visualization workflows applied to the normalized barcode counts table) — https://github.com/DaneVass/bartools
- **R** (Analysis environment in which PCA (prcomp), Pearson correlation, and hierarchical clustering are performed with tidyverse)
- **BARtab** (Upstream Nextflow pipeline that generates the barcode counts table feeding this diversity analysis) — https://github.com/DaneVass/BARtab

## Examples

```
library(tidyverse); counts <- read_tsv("counts/all_counts_combined.tsv") # then prcomp on log-normalized counts and bartools diversity indices per sample
```

## Evaluation signals

- Biological replicate Pearson correlations for DMSO and IBET-151 samples all exceed 0.95
- Treatments are clearly separated along PC1 and the dose-escalation time course along PC2 in the PCA plot
- The hierarchical clustering dendrogram places AraC IC90 samples in a distinct cluster independent of gradual dose-escalation samples
- Shannon, Simpson, and Gini indices are computed for every sample with no missing values, and replicate samples show similar overall clonal composition

## Limitations

- BARtab and bartools currently do not officially support evolving barcode methods
- Benchmarking was hindered by the lack of a gold-standard dataset with known barcode abundance, so diversity indices are relative rather than absolute accuracy-validated measures
- Interpretation depends on adequate sequencing depth per sample; collapsed or filtered barcodes (e.g. default Levenshtein clustering distance of 3) affect diversity estimates

## Evidence

- [intro] we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
- [abstract] BARtab and bartools contain methods to simplify the extraction, quality control, analysis, and visualization of lineage barcodes: "BARtab and bartools contain methods to simplify the extraction, quality control, analysis, and visualization of lineage barcodes"
- [readme] Analysis of population-level dose escalation dataset using bartools in dose-escalation/: [evidence span withheld — anchor status: external_doc]
- [methods] The default Levenshtein distance for clustering is 3 which is conservative to avoid collapsing truly distingt barcodes.: "The default Levenshtein distance for clustering is 3 which is conservative to avoid collapsing truly distingt barcodes."
