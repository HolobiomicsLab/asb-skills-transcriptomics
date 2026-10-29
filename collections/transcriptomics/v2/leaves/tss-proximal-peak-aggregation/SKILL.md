---
name: tss-proximal-peak-aggregation
description: Use when a mosaic dataset contains scATAC-seq batches but lacks scRNA-seq
  matrices for some batches (None placeholders in the counts dictionary), so that
  gene activity scores can substitute for missing gene expression.
when_to_use_negative:
- The batch already has a measured scRNA-seq count matrix — use the real counts rather
  than gene activity scores
- Feature entities (regions and genes) are already connected across all batches by
  the available input matrices, so no pseudo-count imputation is needed
- You need chromatin-region-level (peak) features in the output; aggregation to gene
  bodies discards per-peak resolution
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: scMoMaT
  role: Hosts the calc_pseudo_count/calc_pseudo_count.R script and GxR.npz connection
    matrix used to compute and insert the pseudo-count matrices
  repo: github.com/PeterZZQ/scMoMaT
- name: LIGER
  role: Reference method whose gene activity score procedure the pseudo-count calculation
    follows
  repo: welch-lab/liger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:2349bd7b2e72fdb3d4533b7b76a66c6e699c9743415f6dff9b6144dd4387a95b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LIGER
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tss-proximal-peak-aggregation

## Summary

Aggregates scATAC-seq per-cell peak counts into per-gene accessibility scores by summing peaks within 2000 bp upstream of each gene's TSS plus the gene body, then binarizes the result to form a pseudo-scRNA-seq matrix that fills missing gene-expression modalities in scMoMaT mosaic layouts.

## When to use

Use when a mosaic dataset contains scATAC-seq batches but lacks scRNA-seq matrices for some batches (None placeholders in the counts dictionary), so that gene activity scores can substitute for missing gene expression.

## When NOT to use

- The batch already has a measured scRNA-seq count matrix — use the real counts rather than gene activity scores
- Feature entities (regions and genes) are already connected across all batches by the available input matrices, so no pseudo-count imputation is needed
- You need chromatin-region-level (peak) features in the output; aggregation to gene bodies discards per-peak resolution

## Inputs

- Raw scATAC-seq peak count matrix (ncells x npeaks)
- Gene annotations (TSS coordinates, gene bodies)
- Pre-computed region-to-gene connection matrix GxR.npz
- scRNA-seq count matrix plus 1:1 protein-gene mapping table (for pseudo-protein counts)

## Outputs

- Binarized pseudo-scRNA-seq (gene activity) count matrix of shape (ncells, ngenes)
- Pseudo-protein count matrix produced by 1:1 protein-gene projection
- Filled mosaic counts dictionary with None placeholders replaced

## How to apply

Load the raw scATAC-seq peak count matrix together with gene annotations and the pre-computed region-to-gene connection matrix GxR.npz from the scMoMaT repository. In R (calc_pseudo_count/calc_pseudo_count.R), sum the peak counts of all peaks falling within 2000 bp upstream of each gene's TSS plus the gene body, mirroring the gene activity score procedure used in Seurat and LIGER. Binarize the aggregated gene-activity matrix to obtain pseudo-scRNA-seq counts. By an analogous 1:1 protein-gene mapping, project scRNA-seq counts onto protein features to derive pseudo-protein counts when protein modalities are missing. Insert the resulting pseudo-count matrices into the mosaic counts dictionary in place of the None placeholders and then run utils.preprocess() on the filled matrices before first-stage tri-factorization training.

## Related tools

- **scMoMaT** (Hosts the calc_pseudo_count/calc_pseudo_count.R script and GxR.npz connection matrix used to compute and insert the pseudo-count matrices) — github.com/PeterZZQ/scMoMaT
- **LIGER** (Reference method whose gene activity score procedure the pseudo-count calculation follows) — welch-lab/liger

## Examples

```
Rscript calc_pseudo_count/calc_pseudo_count.R  # uses GxR.npz, 2000bp TSS window + gene body; then insert pseudo-counts for None batches and run utils.preprocess()
```

## Evaluation signals

- Peak window matches the specification: 2000 bp upstream of TSS plus the gene body
- Gene activity matrix is binarized (values in {0,1}) before insertion as pseudo-scRNA-seq counts
- Pseudo-protein counts are generated strictly via the 1:1 protein-gene mapping table (no many-to-many projection)
- Filled counts dictionary has no remaining None placeholders and passes utils.preprocess() without shape errors
- List lengths equal nbatches and matrices are ordered by batch ID in each modality list

## Limitations

- No numeric accuracy threshold is reported for the pseudo-count stage; acceptance is qualitative only
- Pseudo-scRNA-seq counts are binarized proxies, not measured expression, so downstream interpretation should treat them as gene activity scores rather than transcript abundances
- The 1:1 protein-gene projection cannot handle proteins with no unique gene partner; unmatched proteins cannot be imputed this way

## Evidence

- [methods] Pseudo-count computation procedure: [evidence span withheld — anchor status: not_found]
- [methods] Peak aggregation window: [evidence span withheld — anchor status: not_found]
- [methods] Connection matrix availability: "The connection matrix (`GxR.npz`) that is used to generate pseudo-count is also provided in advance"
- [readme] Placeholder convention in counts dictionary: "**The batches that does not have count matrix in corresponding modality is inserted `None` as placeholder**"
- [readme] Preprocessing requirement: "The data matrix is of the shape `(ncells, nfeatures)`, and preprocessed with `utils.preprocess()` function."
