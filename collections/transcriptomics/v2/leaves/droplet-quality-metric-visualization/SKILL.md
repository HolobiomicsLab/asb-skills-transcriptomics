---
name: droplet-quality-metric-visualization
description: Use when quantifying extranuclear RNA contamination per droplet in snRNA-seq,
  or when a sample-specific midpoint separating nuclear and background droplets must
  be estimated from a bimodal percent-spliced (or percent-mitochondrial) distribution.
when_to_use_negative:
- The droplet filtering itself is needed rather than visualization/metric quantification
  — use DIEM's run_em/estimate_dbr_score/call_targets pipeline instead of just plotting
  quality metrics
- The goal is ambient-RNA profile modeling on single-cell (scRNA-seq) data where percent
  spliced is not a bimodal nuclear/background discriminator; DIEM's correlation with
  percent spliced was less successful in PBMC scRNA-seq data
- The data are already a filtered count matrix without raw droplet-level (barcode)
  information — per-droplet metrics and the bimodal distribution cannot be computed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: diem
  role: R package providing read_10x, create_SCE, get_gene_pct, barcode_rank_plot,
    and plot_clust used to compute and visualize per-droplet quality metrics
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores raw counts as a column-wise sparse matrix read from the 10X output
- name: CellRanger
  role: Source of the raw 10X droplet count matrix (e.g., nuclei_2k fresh mouse brain
    dataset) that the metrics are computed on
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:a671ce18727fc892520f11d2df6f6f13c12850158f387168840438319b649fbe
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# droplet-quality-metric-visualization

## Summary

Visualizes per-droplet quality metrics from snRNA-seq/scRNA-seq data — percent mitochondrial reads, MALAT1%, percent-spliced reads, ranked total counts, and debris scores — to expose the bimodal nuclear-vs-background structure that DIEM's contamination model and sample-specific midpoint (near ~50% spliced) are built upon.

## When to use

Use when quantifying extranuclear RNA contamination per droplet in snRNA-seq, or when a sample-specific midpoint separating nuclear and background droplets must be estimated from a bimodal percent-spliced (or percent-mitochondrial) distribution.

## When NOT to use

- The droplet filtering itself is needed rather than visualization/metric quantification — use DIEM's run_em/estimate_dbr_score/call_targets pipeline instead of just plotting quality metrics
- The goal is ambient-RNA profile modeling on single-cell (scRNA-seq) data where percent spliced is not a bimodal nuclear/background discriminator; DIEM's correlation with percent spliced was less successful in PBMC scRNA-seq data
- The data are already a filtered count matrix without raw droplet-level (barcode) information — per-droplet metrics and the bimodal distribution cannot be computed

## Inputs

- Raw droplet/barcode count matrix from CellRanger output directory (e.g., outs/raw_feature_bc_matrix) of a 10X snRNA-seq experiment
- Alignment of reads to spliced/unspliced categories for computing percent-spliced per droplet
- SCE object with gene_data (gene names) for selecting mitochondrial (^mt-) and MALAT1 genes

## Outputs

- percent_spliced.csv with one row per quantified droplet
- midpoint file containing a single numeric sample-specific threshold between the nuclear and background percent-spliced distributions
- Diagnostic plots: barcode rank plot, and scatter plots of quality features (n_genes, pct.mt, MALAT1%) versus debris score

## How to apply

Start from the raw droplet counts (e.g., 10X CellRanger output outs/raw_feature_bc_matrix for the nuclei_2k fresh mouse brain dataset), read them into a column-wise sparse Matrix via diem's read_10x, and create an SCE object with create_SCE. Compute per-droplet quality fractions with get_gene_pct — adding 'pct.mt' from mitochondrial genes (grep '^mt-') and 'MALAT1%' (grep '^malat1') — excluding mito-encoded genes from the spliced/unspliced UMI quantification since they are canonical debris-upregulated markers. Compute percent-spliced per droplet and write the per-droplet table to percent_spliced.csv, then inspect the distribution with barcode_rank_plot and bivariate plots such as plot_clust(sce, feat_x = 'pct.mt', feat_y = 'score.debris'). The paper's rationale is that percent spliced shows a bimodal distribution with nuclear droplets centered below and background droplets above roughly 50%, so a two-component Gaussian mixture is fit via EM and the density-equal midpoint between components is computed — analogous to DIEM's midpoint debris-score threshold of 0.5 between the debris and least-contaminated cluster averages — and written to midpoint.

## Related tools

- **diem** (R package providing read_10x, create_SCE, get_gene_pct, barcode_rank_plot, and plot_clust used to compute and visualize per-droplet quality metrics) — https://github.com/marcalva/diem
- **Matrix** (Stores raw counts as a column-wise sparse matrix read from the 10X output)
- **CellRanger** (Source of the raw 10X droplet count matrix (e.g., nuclei_2k fresh mouse brain dataset) that the metrics are computed on)

## Examples

```
counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); sce <- get_gene_pct(x = sce, genes=mt_genes, name="pct.mt"); barcode_rank_plot(sce)
```

## Evaluation signals

- midpoint parses to a single numeric value between 30 and 70, consistent with the reported ~50% nuclear/background separation
- percent_spliced.csv contains one row per quantified droplet
- The percent-spliced histogram is clearly bimodal with nuclear and background droplets centered below and above roughly 50%
- pct.mt is computed only over mitochondrial genes matching '^mt-' (ignore.case=TRUE) and MALAT1% over '^malat1$', and mito genes are excluded from the spliced/unspliced quantification
- barcodes ranked in barcode_rank_plot include background droplets, not only cells above a hard count cutoff

## Limitations

- Debris scores may not be comparable across independent experiments because they are normalized relative to within-sample cluster means; the midpoint is therefore estimated independently for each experiment
- The debris score/contamination estimate depends on genes being enriched in the background distribution; if few or no genes are increased there, scores will likely be inaccurate
- Correlation of debris score with extranuclear contamination (percent reads spliced) was less successful in PBMC single-cell RNA-seq data, possibly due to smaller differences between background and cell type profiles

## Evidence

- [readme] mitochondrial metric selection: [evidence span withheld — anchor status: external_doc]
- [readme] quality-fraction computation: [evidence span withheld — anchor status: external_doc]
- [readme] bivariate quality-vs-debris-score plot: "plot_clust(sce, feat_x = "pct.mt", feat_y = "score.debris", 
           log_x = TRUE, log_y = FALSE)"
- [intro] percent-spliced as contamination measure: "an ad hoc cutoff of the percent of reads originating from the mitochondria (a measure of extranuclear contamination) can help to filter droplets"
- [methods] sparse matrix storage of raw counts: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
- [discussion] adjustable debris threshold rationale: "The debris score threshold that removes droplets can be adjusted according to the desired tolerance for contamination."
