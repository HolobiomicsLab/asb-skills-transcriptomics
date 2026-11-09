---
name: single-nucleus-rnaseq-ambient-rna-classification
description: Use when a 10X raw droplet count matrix (e.g. raw_feature_bc_matrix from
  fresh mouse brain nuclei) shows extranuclear RNA contamination and you must decide
  which droplets to keep versus remove as background.
when_to_use_negative:
- Input is already a filtered expression matrix with low-quality droplets removed
  — classification needs the raw droplet count matrix.
- The data are scRNA-seq with little difference between background and cell profiles
  (e.g. PBMC scRNA-seq), where debris-score correlation with extranuclear contamination
  was less successful; use call_targets with clusters = 'debris' instead of a score
  threshold.
- You have no genes increased in the background distribution — the scoring approach
  will likely produce inaccurate debris scores.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: diem
  role: R package providing read_10x, create_SCE, get_gene_pct, run_em, estimate_dbr_score,
    and call_targets for debris classification of droplets
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores the raw counts as a column-wise sparse matrix after 10X import
- name: EmptyDrops
  role: Comparison filtering method based on a Dirichlet-Multinomial model of ambient
    RNA
- name: CellRanger
  role: Alignment pipeline whose 10X output directory read_10x consumes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:fb545b53edac35f41dbe268e00ba2d1c7dfd4df6e711db91fd4e95e2ee9ba86e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - EmptyDrops
  - Matrix
  - R
  - diem
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# single-nucleus-rnaseq-ambient-rna-classification

## Summary

Classify droplets in droplet-based snRNA-seq data as nuclear targets versus debris/background using DIEM's likelihood-based multinomial model with semi-supervised EM, and benchmark it against hard-count and EmptyDrops filtering by cross-tabulating removed droplets with percent-spliced-read labels.

## When to use

Use when a 10X raw droplet count matrix (e.g. raw_feature_bc_matrix from fresh mouse brain nuclei) shows extranuclear RNA contamination and you must decide which droplets to keep versus remove as background.

## When NOT to use

- Input is already a filtered expression matrix with low-quality droplets removed — classification needs the raw droplet count matrix.
- The data are scRNA-seq with little difference between background and cell profiles (e.g. PBMC scRNA-seq), where debris-score correlation with extranuclear contamination was less successful; use call_targets with clusters = 'debris' instead of a score threshold.
- You have no genes increased in the background distribution — the scoring approach will likely produce inaccurate debris scores.

## Inputs

- 10X raw droplet count matrix (raw_feature_bc_matrix, e.g. nuclei_2k fresh mouse brain snRNA-seq) readable by read_10x
- Splice-gene annotation for computing percent spliced (nuclear-localized) reads per droplet
- SCE object with per-droplet total counts, number of genes detected, and barcode rank ordering

## Outputs

- Per-droplet debris scores and target/debris calls from call_targets
- Contingency table of removed vs. passing droplets classified as background vs. nuclear by percent-spliced reads (e.g. quantile_filter_summary.csv)
- Filtered SCE object convertible to Seurat via convert_to_seurat

## How to apply

Load the raw count matrix into R as a column-wise sparse Matrix via diem's read_10x and wrap it with create_SCE. Run the DIEM pipeline: set_debris_test_set, filter_genes, get_pcs, init, run_em, assign_clusters, estimate_dbr_score, then call_targets with a debris-score threshold (e.g. thresh_score = 0.5 for single-nucleus data) — DIEM first fixes all droplets below a hard count threshold as debris, then scores droplets on expression of debris-enriched genes with semi-supervised EM. Annotate droplets biologically with get_gene_pct, e.g. percent of spliced (nuclear-localized) reads or mt-/MALAT1 percent, and treat low percent-spliced droplets as background. For validation against a quantile-based hard-count baseline, compute the threshold as the 99th percentile of counts of the top 3,000 barcodes divided by 10, split droplets at that cutoff, and tabulate how many removed droplets are background (low percent-spliced). In the mouse brain nuclei_2k data, DIEM removed 117/171 (68.4%) background droplets and EmptyDrops 65/101 (64.4%), whereas the quantile filter removed only 26/98 (26.5%), so a high background fraction among removed droplets indicates the filter discriminates debris from nuclei.

## Related tools

- **diem** (R package providing read_10x, create_SCE, get_gene_pct, run_em, estimate_dbr_score, and call_targets for debris classification of droplets) — https://github.com/marcalva/diem
- **Matrix** (Stores the raw counts as a column-wise sparse matrix after 10X import)
- **EmptyDrops** (Comparison filtering method based on a Dirichlet-Multinomial model of ambient RNA)
- **CellRanger** (Alignment pipeline whose 10X output directory read_10x consumes)

## Examples

```
counts <- read_10x("nuclei_2k/raw_feature_bc_matrix"); sce <- create_SCE(counts); sce <- call_targets(sce, thresh_score = 0.5, min_genes = 0)
```

## Evaluation signals

- In mouse brain nuclei_2k, DIEM-classified removals should show a substantially higher background fraction (117/171 = 68.4% by percent-spliced classification) than the quantile hard-count filter (26/98 = 26.5%).
- EmptyDrops comparison should reproduce 65/101 = 64.4% of removed droplets classified as background.
- The quantile baseline threshold (99th percentile of top 3,000 barcodes / 10) should reproduce 1,832 of 2,083 passing nuclear droplets and 26 of 98 removed background droplets.
- DIEM-filtered data should yield higher-quality clusters and fewer spurious cell types than hard-cutoff or EmptyDrops filtering.
- Debris-score distributions should correlate with percent of extranuclear (unspliced/mitochondrial) reads in snRNA-seq data.

## Limitations

- Debris scores cannot be assumed to exist on the same scale in independent experiments, so they may not be comparable when integrating multiple samples, especially if extranuclear RNA distributions differ across samples.
- Scoring relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, the debris score will likely be inaccurate.
- EmptyDrops was originally developed and tested on single-cell data, so its model assumptions differ from DIEM's, which may degrade snRNA-seq performance.
- All droplets below a hard count threshold are fixed as debris by the semi-supervised EM, so a poor initial threshold can bias the model.

## Evidence

- [intro] semi-supervised EM fixes low-count droplets as debris: "where all droplets below a hard count threshold are fixed as debris. Then, the droplets are scored based on their expression of"
- [intro] EmptyDrops ambient RNA model: "EmptyDrops12 addresses this filtering issue for scRNA-seq by estimating a Dirichlet-Multinomial distribution of the ambient RNA."
- [methods] read_10x convenience function: "We provide a convenience function `read_10x` that can read the 10X output into a sparse Matrix."
- [methods] percent spliced reads via get_gene_pct: "We can generate this using the `get_gene_pct` function"
- [discussion] debris score threshold call_targets: "Droplets in the snRNA-seq experiments are filtered using a threshold for the debris score."
- [discussion] score scale incomparability across experiments: [evidence span withheld — anchor status: not_found]
- [readme] quick README workflow: "counts <- read_10x("path/to/10x") # Read 10X data into sparse matrix
sce <- create_SCE(counts) # Create SCE object from counts"
- [readme] call_targets with debris-score threshold: [evidence span withheld — anchor status: external_doc]
