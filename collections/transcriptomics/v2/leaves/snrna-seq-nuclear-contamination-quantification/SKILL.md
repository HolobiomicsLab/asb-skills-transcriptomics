---
name: snrna-seq-nuclear-contamination-quantification
description: Use when processing raw 10X droplet snRNA-seq counts and you need a per-droplet
  contamination metric (percent-spliced reads) plus an experiment-specific nuclear/background
  midpoint threshold to validate or calibrate debris filtering.
when_to_use_negative:
- 'Input is single-cell RNA-seq (scRNA-seq) rather than nuclei: the paper notes the
  percent-spliced vs debris-score correlation ''was less successful in the PBMC single-cell
  RNA-seq data'' because background and cell type profiles differ less'
- You only have a filtered CellRanger matrix or pre-filtered droplet set — background
  droplets needed for the bimodal fit must come from the raw_feature_bc_matrix
- The goal is to compute DIEM's multinomial debris score from expression profiles
  rather than the splicing-based contamination metric used here for validation
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DIEM
  role: Reference debris-scoring framework whose two-population model and 0.5 midpoint
    debris-score threshold the percent-spliced midpoint estimation mirrors for validation
  repo: https://github.com/marcalva/diem
- name: CellRanger
  role: Source of the raw droplet count matrix (outs/raw_feature_bc_matrix) used as
    the input droplet universe
- name: R
  role: Implementation language for per-droplet quantification and EM-fitted Gaussian
    mixture
- name: Matrix
  role: Stores the raw counts as a column-wise sparse matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:a999e5ee4e957f85c1e786bf52366fbe7189637ef30ec50d5bac97b9f3097b75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - Matrix
  - R
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# snrna-seq-nuclear-contamination-quantification

## Summary

Quantifies extranuclear RNA contamination per droplet in droplet-based snRNA-seq using the percent of reads that are spliced (mitochondrial genes excluded), and estimates a sample-specific midpoint from the bimodal percent-spliced distribution to separate nuclear droplets from background droplets, validating DIEM-style debris scoring.

## When to use

Use when processing raw 10X droplet snRNA-seq counts and you need a per-droplet contamination metric (percent-spliced reads) plus an experiment-specific nuclear/background midpoint threshold to validate or calibrate debris filtering.

## When NOT to use

- Input is single-cell RNA-seq (scRNA-seq) rather than nuclei: the paper notes the percent-spliced vs debris-score correlation 'was less successful in the PBMC single-cell RNA-seq data' because background and cell type profiles differ less
- You only have a filtered CellRanger matrix or pre-filtered droplet set — background droplets needed for the bimodal fit must come from the raw_feature_bc_matrix
- The goal is to compute DIEM's multinomial debris score from expression profiles rather than the splicing-based contamination metric used here for validation

## Inputs

- Raw droplet count matrix from CellRanger outs/raw_feature_bc_matrix (e.g. 10X nuclei_2k fresh mouse brain nuclei dataset)
- Aligned BAM/splice-aware alignments for spliced vs unspliced UMI quantification
- Gene annotation to identify mitochondrial genes for exclusion

## Outputs

- percent_spliced.csv — per-droplet table with one row per quantified droplet containing percent-spliced reads
- midpoint file — single numeric density-equal midpoint value (expected between 30 and 70) separating nuclear and background percent-spliced distributions

## How to apply

Obtain the raw droplet count matrix from the CellRanger output directory outs/raw_feature_bc_matrix (e.g. the 10X nuclei_2k fresh mouse brain dataset) and load it as a column-wise sparse matrix via the R Matrix package or DIEM's read_10x. Quantify spliced and unspliced UMI counts per droplet from the aligned data, explicitly excluding reads mapping to mitochondrial genes because mito-encoded genes are the canonical debris-upregulated markers that would confound a splicing-based contamination signal. Compute percent-spliced per droplet and write percent_spliced.csv with one row per quantified droplet. Fit a two-component Gaussian mixture to the per-droplet percent-spliced distribution using EM, matching the paper's two-population model of low nuclear-content/contaminated and high nuclear-content droplets. Compute the density-equal midpoint between the two fitted components — analogous to DIEM's midpoint debris-score threshold of 0.5 between the debris and least-contaminated cluster averages — and write it to the midpoint output file.

## Related tools

- **DIEM** (Reference debris-scoring framework whose two-population model and 0.5 midpoint debris-score threshold the percent-spliced midpoint estimation mirrors for validation) — https://github.com/marcalva/diem
- **CellRanger** (Source of the raw droplet count matrix (outs/raw_feature_bc_matrix) used as the input droplet universe)
- **R** (Implementation language for per-droplet quantification and EM-fitted Gaussian mixture)
- **Matrix** (Stores the raw counts as a column-wise sparse matrix)

## Examples

```
counts <- read_10x("outs/raw_feature_bc_matrix"); sce <- create_SCE(counts); sce <- get_gene_pct(x = sce, genes = mt_genes, name = "pct.mt")
```

## Evaluation signals

- midpoint file parses to a single numeric value between 30 and 70, consistent with the reported ~50% nuclear/background separation
- percent_spliced.csv contains exactly one row per quantified droplet from the raw matrix
- Percent-spliced distribution shows clear bimodality with nuclear droplets centered below and background droplets above roughly 50%
- The two EM-fitted Gaussian components converge with distinct means (low nuclear-content/contaminated vs high nuclear-content)
- Mito-encoded genes are confirmed excluded from the spliced/unspliced quantification so the metric reflects splicing, not mitochondrial contamination

## Limitations

- Debris scores are normalized relative to within-sample cluster means and 'cannot be assumed to exist on the same scale in independent experiments', so a single midpoint is computed independently per experiment
- Scoring relies on accurate estimation of debris-enriched genes; if no genes are increased in the background distribution, resulting scores will likely be inaccurate
- Splicing-based contamination correlation underperformed on PBMC single-cell RNA-seq data, where background and cell type profiles are less differentiated

## Evidence

- [results] The percent of reads spliced shows a bimodal distribution with nuclear and background droplets centered below and above roughly 50%: [evidence span withheld — anchor status: not_found]
- [methods] raw matrix source: "This tutorial will use the publicly available 10X single-nucleus RNA-seq data that was generated from fresh mouse brain tissue."
- [methods] sparse matrix storage: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
- [methods] midpoint threshold analogy: "mb_small <- call_targets(mb_small,
                         thresh_score = 0.5,
                         min_genes = 0)"
- [discussion] cross-sample comparability caveat: [evidence span withheld — anchor status: not_found]
- [discussion] scRNA-seq failure mode: "this was less successful in the PBMC single-cell RNA-seq data. This may be due to smaller differences between the background and cell type profiles in scRNA-seq."
- [readme] read_10x convenience function: [evidence span withheld — anchor status: external_doc]
