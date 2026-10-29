---
name: spliced-unspliced-umi-quantification-per-droplet
description: Use when processing raw 10X snRNA-seq droplet data (e.g. CellRanger outs/raw_feature_bc_matrix)
  to quantify per-droplet contamination via percent-spliced reads and estimate the
  sample-specific nuclear/background midpoint.
when_to_use_negative:
- Input is single-cell (not single-nucleus) data such as PBMC scRNA-seq, where the
  correlation between debris scoring and percent reads spliced was less successful
  due to smaller differences between background and cell type profiles
- You need contamination scores comparable across independently processed samples
  — the paper explicitly warns the debris score may not be on the same scale across
  experiments, so midpoints must be estimated per sample rather than transferred
- Input is already filtered to cell barcodes; background droplets must be present
  (raw_feature_bc_matrix) for the bimodal distribution and midpoint to be estimable
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0091
tools:
- name: DIEM
  role: Provides the R/SCE framework (read_10x, create_SCE, get_gene_pct) whose two-population
    debris/nucleus model motivates the bimodal percent-spliced midpoint approach
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores the raw 10X droplet counts as a column-wise sparse matrix prior to
    per-droplet UMI tabulation
- name: CellRanger
  role: Alignment pipeline whose outs/raw_feature_bc_matrix directory supplies the
    raw per-droplet counts
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger
- name: R
  role: Language used to compute percent spliced per droplet, fit the two-component
    Gaussian mixture via EM, and emit percent_spliced.csv and midpoint
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:4c898c37c877520f2e4445862d6c160767c706632413ac98c4b120ed995f6c81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - Matrix
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spliced-unspliced-umi-quantification-per-droplet

## Summary

Quantifies extranuclear RNA contamination per 10X droplet in snRNA-seq by computing the percent of reads spliced (from spliced vs. unspliced UMI counts, excluding mitochondrial genes), then estimates a sample-specific midpoint separating the bimodal nuclear/background droplet distributions.

## When to use

Use when processing raw 10X snRNA-seq droplet data (e.g. CellRanger outs/raw_feature_bc_matrix) to quantify per-droplet contamination via percent-spliced reads and estimate the sample-specific nuclear/background midpoint.

## When NOT to use

- Input is single-cell (not single-nucleus) data such as PBMC scRNA-seq, where the correlation between debris scoring and percent reads spliced was less successful due to smaller differences between background and cell type profiles
- You need contamination scores comparable across independently processed samples — the paper explicitly warns the debris score may not be on the same scale across experiments, so midpoints must be estimated per sample rather than transferred
- Input is already filtered to cell barcodes; background droplets must be present (raw_feature_bc_matrix) for the bimodal distribution and midpoint to be estimable

## Inputs

- Raw droplet count matrix from CellRanger output directory outs/raw_feature_bc_matrix (10X fresh mouse brain nuclei_2k or equivalent snRNA-seq)
- Aligned data with spliced/unspliced annotation (UMI counts per droplet split by splicing status)
- Mitochondrial gene list (genes matching ^mt-/MT- pattern from gene_data rownames) to be excluded

## Outputs

- percent_spliced.csv — per-droplet percent-spliced table, one row per quantified droplet
- midpoint — sample-specific single numeric value separating nuclear and background droplet distributions

## How to apply

Read the raw droplet counts (all barcodes, not just called cells) from the CellRanger output directory outs/raw_feature_bc_matrix into a column-wise sparse matrix via R/Matrix (e.g. diem's read_10x into an SCE object). For each droplet, tally spliced and unspliced UMI counts from the aligned data while explicitly excluding mitochondrial genes — grep genes matching ^mt- since mito-encoded reads are the canonical debris-upregulated signal that would confound the spliced metric. Compute percent spliced per droplet and write the per-droplet table to percent_spliced.csv. Fit a two-component Gaussian mixture (one low nuclear-content/contaminated cluster, one high nuclear-content cluster) to the percent-spliced distribution using EM, mirroring DIEM's two-population debris/nucleus model. Compute the density-equal midpoint between the two fitted components — analogous to the paper's debris-score threshold of 0.5 between debris and least-contaminated cluster averages — and write it to midpoint; this must be recomputed per experiment because the debris score cannot be assumed to exist on the same scale in independent experiments.

## Related tools

- **DIEM** (Provides the R/SCE framework (read_10x, create_SCE, get_gene_pct) whose two-population debris/nucleus model motivates the bimodal percent-spliced midpoint approach) — https://github.com/marcalva/diem
- **Matrix** (Stores the raw 10X droplet counts as a column-wise sparse matrix prior to per-droplet UMI tabulation)
- **CellRanger** (Alignment pipeline whose outs/raw_feature_bc_matrix directory supplies the raw per-droplet counts) — https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger
- **R** (Language used to compute percent spliced per droplet, fit the two-component Gaussian mixture via EM, and emit percent_spliced.csv and midpoint)

## Examples

```
library(diem); counts <- read_10x("outs/raw_feature_bc_matrix"); sce <- create_SCE(counts); sce <- get_gene_pct(x = sce, genes = mt_genes, name = "pct_spliced")
```

## Evaluation signals

- percent_spliced.csv contains one row per quantified droplet, matching the number of barcodes in outs/raw_feature_bc_matrix
- midpoint parses to a single numeric value between 30 and 70, consistent with the reported ~50% nuclear/background separation
- The per-droplet percent-spliced distribution is bimodal, with nuclear droplets centered below roughly 50% and background droplets above
- Mitochondrial genes (grep ^mt-) are excluded from the spliced/unspliced tally so the metric reflects extranuclear contamination rather than mito-encoded reads
- The midpoint differs across experiments — a reused cross-sample midpoint indicates the per-experiment estimation step was skipped

## Limitations

- Debris scores cannot be assumed to exist on the same scale in independent experiments, which is why the midpoint must be estimated per sample, particularly when integrating multiple samples with different distributions of extranuclear RNA
- The association of debris scoring with percent reads spliced was less successful in PBMC single-cell RNA-seq data, likely due to smaller differences between the background and cell type profiles in scRNA-seq
- Debris scoring relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, scores (and hence separation quality) will likely be inaccurate

## Evidence

- [intro] mito exclusion motivator: "Additionally, an ad hoc cutoff of the percent of reads originating from the mitochondria (a measure of extranuclear contamination) can help to filter droplets12."
- [methods] sparse matrix storage: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
- [discussion] cross-sample scale caveat: "Thus, when integrating multiple samples, the debris scores may not be comparable, particularly if the distribution of extranuclear RNA is different across the samples."
- [discussion] scRNA-seq failure mode: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
- [readme] 10X reading entry point: [evidence span withheld — anchor status: external_doc]
