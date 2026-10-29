---
name: hierarchical-sample-clustering-by-barcode-composition
description: Use when you have a normalized barcode counts table (samples x barcodes)
  and need to test whether samples group by treatment condition based on overall clonal
  composition rather than single-clone abundance.
when_to_use_negative:
- Input is raw (unnormalized) barcode counts — composition-based correlation will
  be dominated by sequencing depth rather than clonal structure
- The goal is pre-processing or barcode extraction from FASTQ — that is BARtab's pipeline
  (SOFTWARE_CHECK through counts table generation), not downstream clustering
- Lineage reconstruction from single-cell phylogenetic data is required — tools like
  Cassiopeia or LinRace are the appropriate choice per the article's discussion
edam_operation: http://edamontology.org/operation_3432
tools:
- name: R
  role: Language in which the correlation matrix, dissimilarity transform, and hclust
    dendrogram are computed
- name: tidyverse
  role: Used to load, normalize, and wrangle the combined barcode counts table before
    clustering
- name: bartools
  role: R package providing population-level barcoding workflows for normalizing counts
    and computing sample-level diversity summaries that contextualize the clustering
  repo: https://github.com/DaneVass/bartools
- name: BARtab
  role: Upstream Nextflow pipeline that generates the barcode counts table which is
    the clustering input
  repo: https://github.com/DaneVass/BARtab
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:5849feaa797e293e2b773d5f2987aa1306f42f656c5943fa803aa82f8340ee06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - bartools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Hierarchical sample clustering by barcode composition

## Summary

Clusters biological samples based on similarity of their synthetic barcode abundance profiles using hierarchical clustering on correlation-based dissimilarity, to reveal whether treatment groups (e.g., AraC IC90 vs. gradual dose-escalation DMSO/IBET-151 samples) occupy distinct clonal-composition states.

## When to use

Use when you have a normalized barcode counts table (samples x barcodes) and need to test whether samples group by treatment condition based on overall clonal composition rather than single-clone abundance.

## When NOT to use

- Input is raw (unnormalized) barcode counts — composition-based correlation will be dominated by sequencing depth rather than clonal structure
- The goal is pre-processing or barcode extraction from FASTQ — that is BARtab's pipeline (SOFTWARE_CHECK through counts table generation), not downstream clustering
- Lineage reconstruction from single-cell phylogenetic data is required — tools like Cassiopeia or LinRace are the appropriate choice per the article's discussion

## Inputs

- Normalized barcode counts table (counts/all_counts_combined.tsv; samples as columns, barcodes as rows) produced by BARtab and normalized in R
- Sample metadata mapping each column to treatment (DMSO, IBET-151, AraC), dose/time point, and replicate

## Outputs

- Hierarchical clustering dendrogram of samples based on correlation dissimilarity of barcode composition
- Assignment of samples to clusters (e.g., AraC IC90 cluster vs. dose-escalation cluster)
- Supporting pairwise sample correlation matrix

## How to apply

Start from the normalized combined barcode counts table (counts/all_counts_combined.tsv) loaded in R with tidyverse and log-normalized across samples. Compute a pairwise Pearson correlation matrix of sample barcode compositions, convert to a dissimilarity (e.g., 1 - correlation), and run hierarchical clustering (hclust) to produce a dendrogram of samples. Interpret the dendrogram alongside the replicate-correlation check: biological replicates of DMSO and IBET-151 samples should show Pearson correlations > 0.95, so they must co-cluster. Correct application is demonstrated when AraC IC90 samples fall into a distinct cluster from the gradual dose-escalation time-course samples, mirroring the PCA separation along PC1.

## Related tools

- **R** (Language in which the correlation matrix, dissimilarity transform, and hclust dendrogram are computed)
- **tidyverse** (Used to load, normalize, and wrangle the combined barcode counts table before clustering)
- **bartools** (R package providing population-level barcoding workflows for normalizing counts and computing sample-level diversity summaries that contextualize the clustering) — https://github.com/DaneVass/bartools
- **BARtab** (Upstream Nextflow pipeline that generates the barcode counts table which is the clustering input) — https://github.com/DaneVass/BARtab

## Examples

```
hc <- hclust(as.dist(1-cor(log_norm_counts, method="pearson"))); plot(hc, labels=sample_metadata$treatment)
```

## Evaluation signals

- Biological replicate pairs for DMSO and IBET-151 samples have pairwise Pearson correlations > 0.95 and co-cluster in the dendrogram
- The dendrogram places AraC IC90 samples in a distinct cluster, separate from the gradual dose-escalation samples
- Clustering topology is concordant with the PCA result in which treatments separate along PC1 and the dose-escalation time course separates along PC2
- Distance matrix is symmetric with zero diagonal (1 - correlation on identical samples = 0), and all correlations are within [-1, 1]

## Limitations

- Benchmarking BARtab/bartools analyses was difficult due to the lack of a gold-standard dataset with known barcode abundance, so clustering results have no absolute ground truth to compare against
- Clustering reflects overall clonal composition similarity only; it does not by itself establish causal lineage relationships, which would require lineage-reconstruction tools
- Default barcode clustering distance choices (e.g., Levenshtein distance 3) affect which barcodes are collapsed upstream, which propagates into the sample clustering input

## Evidence

- [other] Hierarchically cluster samples by barcode composition (e.g., distance on correlation dissimilarity) to demonstrate that AraC IC90 samples cluster independently of gradual dose-escalation samples.: [evidence span withheld — anchor status: not_found]
- [intro] we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
- [readme] Analysis of population-level dose escalation dataset using bartools in dose-escalation/: [evidence span withheld — anchor status: external_doc]
- [results] replicate samples displayed similar overall clonal composition with DMSO and IBET-151 correlations all > 0.95: [evidence span withheld — anchor status: not_found]
- [methods] The default Levenshtein distance for clustering is 3 which is conservative to avoid collapsing truly distingt barcodes.: "The default Levenshtein distance for clustering is 3 which is conservative to avoid collapsing truly distingt barcodes."
