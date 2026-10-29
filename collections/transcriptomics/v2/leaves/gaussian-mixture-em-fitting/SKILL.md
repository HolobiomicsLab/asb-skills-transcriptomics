---
name: gaussian-mixture-em-fitting
description: 'Use when quantifying extranuclear RNA contamination per droplet in droplet-based
  snRNA-seq: the percent of reads spliced is bimodal with nuclear and background droplets
  straddling ~50%, and a sample-specific midpoint separating the two distributions
  must be estimated per experiment.'
when_to_use_negative:
- Data are single-cell RNA-seq (e.g. PBMCs) where background and cell/nuclear RNA
  profiles differ little — percent-spliced-based separation was less successful in
  the PBMC scRNA-seq data.
- Only the filtered CellRanger matrix (filtered_feature_bc_matrix) is available —
  background droplets must be present for the distribution to be bimodal.
- There are few or no genes increased in the background distribution — the scoring/fitting
  approach will likely be inaccurate.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: diem
  role: R package providing read_10x/create_SCE to load the 10X raw droplet matrix
    and the EM-based two-population (debris vs. cell) modeling paradigm that the Gaussian
    mixture midpoint estimation mirrors
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores the raw droplet counts as a column-wise sparse matrix in R before per-droplet
    percent-spliced computation
- name: CellRanger
  role: 10X alignment pipeline whose outs/raw_feature_bc_matrix output supplies the
    unfiltered droplet counts used for the bimodal percent-spliced fit
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:d5f5b373c61b614a8fc29a6357b7f9fe1939d07060749a39c1b0abba01d7e988
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - Matrix
  - R
  - diem
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# gaussian-mixture-em-fitting

## Summary

Fits a two-component Gaussian mixture (via EM) to the per-droplet percent-spliced-reads distribution of a snRNA-seq experiment, then computes the density-equal midpoint between the nuclear and background components as a sample-specific separation threshold, mirroring DIEM's EM-based two-population debris/cell model.

## When to use

Use when quantifying extranuclear RNA contamination per droplet in droplet-based snRNA-seq: the percent of reads spliced is bimodal with nuclear and background droplets straddling ~50%, and a sample-specific midpoint separating the two distributions must be estimated per experiment.

## When NOT to use

- Data are single-cell RNA-seq (e.g. PBMCs) where background and cell/nuclear RNA profiles differ little — percent-spliced-based separation was less successful in the PBMC scRNA-seq data.
- Only the filtered CellRanger matrix (filtered_feature_bc_matrix) is available — background droplets must be present for the distribution to be bimodal.
- There are few or no genes increased in the background distribution — the scoring/fitting approach will likely be inaccurate.

## Inputs

- CellRanger raw droplet count matrix directory (outs/raw_feature_bc_matrix, 10X MTX format) from the 10X nuclei_2k fresh mouse brain snRNA-seq dataset
- Alignment (BAM/counts) supporting per-droplet spliced vs. unspliced UMI classification

## Outputs

- percent_spliced.csv — per-droplet table of percent of reads spliced (one row per quantified droplet, mito reads excluded)
- midpoint — single numeric value (expected between 30 and 70) giving the density-equal boundary between the fitted nuclear and background components

## How to apply

Obtain the raw (unfiltered) droplet counts from the CellRanger output directory outs/raw_feature_bc_matrix of the 10X nuclei_2k fresh mouse brain dataset and load them with diem's read_10x into a column-wise sparse Matrix. Quantify spliced and unspliced UMI counts per droplet from the aligned data, excluding reads mapping to mitochondrial genes, then compute percent-spliced per droplet and write it to percent_spliced.csv (one row per quantified droplet). Fit a two-component Gaussian mixture to this distribution using EM, consistent with the paper's two-population model (one debris/background cluster and one nuclear-content cluster, whose parameters are inferred by semi-supervised EM). Compute the density-equal midpoint between the two fitted components — analogous to the paper's debris-score threshold of 0.5 between debris and least-contaminated cluster averages — and write it to midpoint. Re-estimate this midpoint independently for each experiment rather than reusing it, and validate that it parses to a single numeric value between 30 and 70, consistent with the reported ~50% nuclear/background separation.

## Related tools

- **diem** (R package providing read_10x/create_SCE to load the 10X raw droplet matrix and the EM-based two-population (debris vs. cell) modeling paradigm that the Gaussian mixture midpoint estimation mirrors) — https://github.com/marcalva/diem
- **Matrix** (Stores the raw droplet counts as a column-wise sparse matrix in R before per-droplet percent-spliced computation)
- **CellRanger** (10X alignment pipeline whose outs/raw_feature_bc_matrix output supplies the unfiltered droplet counts used for the bimodal percent-spliced fit) — https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger

## Examples

```
counts <- read_10x("outs/raw_feature_bc_matrix"); sce <- create_SCE(counts)
```

## Evaluation signals

- midpoint parses to a single numeric value between 30 and 70, consistent with the reported ~50% nuclear/background separation
- percent_spliced.csv contains one row per quantified droplet from outs/raw_feature_bc_matrix (mito reads excluded from the spliced/unspliced tally)
- The per-droplet percent-spliced distribution is visibly bimodal, with one component centered below and one above roughly 50%
- The fitted mixture has exactly two Gaussian components whose EM solution converges, and the reported midpoint equals the density-equal crossing point of the two weighted component densities
- The midpoint is recomputed independently per experiment rather than copied between samples, since debris/contamination scores are not on the same scale across independent experiments

## Limitations

- The percent-spliced correlation with extranuclear contamination was less successful in PBMC single-cell RNA-seq data, possibly due to smaller differences between background and cell type profiles in scRNA-seq
- Debris scores (and hence derived thresholds) cannot be assumed to exist on the same scale in independent experiments, particularly if the distribution of extranuclear RNA differs across samples
- The approach relies on accurate estimation of debris-enriched genes; if there are little or no genes increased in the background distribution, the resulting score will likely be inaccurate

## Evidence

- [other] The percent of reads spliced shows a bimodal distribution with nuclear and background droplets centered below and above roughly 50%: [evidence span withheld — anchor status: not_found]
- [abstract] Our likelihood-based approach models the gene expression distribution of debris and cell types, which are estimated using EM.: "Our likelihood-based approach models the gene expression distribution of debris and cell types, which are estimated using EM."
- [intro] The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris.: "The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris."
- [methods] This tutorial will use the publicly available 10X single-nucleus RNA-seq data that was generated from fresh mouse brain tissue.: "This tutorial will use the publicly available 10X single-nucleus RNA-seq data that was generated from fresh mouse brain tissue."
- [discussion] the debris score, however, cannot be assumed to exist on the same scale in independent experiments.: [evidence span withheld — anchor status: not_found]
- [discussion] this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq.: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
- [readme] counts <- read_10x("path/to/10x") # Read 10X data into sparse matrix: [evidence span withheld — anchor status: external_doc]
