---
name: single-cell-count-normalization
description: Use when a raw UMI count matrix (e.g., from GEO GSM4905334) must be library-size
  normalized to the clone's median UMI count before signature scoring, filterGenesFano
  gene selection, or Geary's C autocorrelation analysis.
when_to_use_negative:
- Input is already normalized or pre-standardized expression data (Hotspot's model='none'
  assumes values are pre-standardized, so re-normalizing would distort the null model).
- Analysis goal is purely phylogenetic (tree reconstruction with Cassiopeia or tree
  manipulation with ape's collapse.singles) with no expression data involved.
- You need dimensionality reduction of zero-inflated counts via dedicated models (ZIFA,
  ZINB-WaVE, scvi-tools) rather than simple library-size scaling before signature
  scoring.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PhyloVision
  role: consumes the normalized counts as input to compute cell-level signature Z-scores
    and Geary's C phylogenetic autocorrelation
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: front-end and signature-scoring framework whose filterGenesFano function selects
    informative genes from the normalized matrix
  repo: https://github.com/YosefLab/VISION
- name: Hotspot
  role: scales expression by per-cell UMI counts (umi_counts_obs_key) when fitting
    null models for autocorrelation and module detection
  repo: https://github.com/YosefLab/Hotspot
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:3d847eb150186f9d30308eaef5913c460c2ee843cd144b02e45e20ff8cb0a7f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - Hotspot
  - PhyloVision
  - R
  - VISION
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-count-normalization

## Summary

Library-size normalization of raw single-cell UMI count matrices as a preprocessing step for downstream PhyloVision/VISION signature scoring and phylogenetic autocorrelation analysis. Counts are scaled to a common per-cell total so that signature Z-scores are comparable across cells.

## When to use

Use when a raw UMI count matrix (e.g., from GEO GSM4905334) must be library-size normalized to the clone's median UMI count before signature scoring, filterGenesFano gene selection, or Geary's C autocorrelation analysis.

## When NOT to use

- Input is already normalized or pre-standardized expression data (Hotspot's model='none' assumes values are pre-standardized, so re-normalizing would distort the null model).
- Analysis goal is purely phylogenetic (tree reconstruction with Cassiopeia or tree manipulation with ape's collapse.singles) with no expression data involved.
- You need dimensionality reduction of zero-inflated counts via dedicated models (ZIFA, ZINB-WaVE, scvi-tools) rather than simple library-size scaling before signature scoring.

## Inputs

- raw UMI count matrix (cells x genes) for a single clone, filtered to cells shared between expression data and the phylogeny
- per-cell total UMI counts (used as the per-cell scaling factor)

## Outputs

- library-size-normalized expression matrix scaled to the clone's median UMI count
- normalized expression values feeding into VISION signature Z-scores and Hotspot/Geary's C autocorrelation statistics

## How to apply

Take the raw UMI count matrix (cells x genes) for the clone under analysis — in the Quinn et al. 2021 lung adenocarcinoma case, the clone of 1,127 cells from GSE161363/GSM4905334 kept in both the expression matrix and the Cassiopeia-reconstructed phylogeny. Scale each cell's counts by a per-cell total so every cell sums to the same size factor, using the median number of UMIs across cells in the clone as the target depth; this mirrors VISION/Hotspot, which scale expression by per-cell UMI counts. Downstream steps (VISION's filterGenesFano with default parameters to pick projection_genes, computation of signature Z-scores, and Geary's C autocorrelation with num_neighbors = 30) assume these normalized values, so normalization must precede signature construction and tree-based scoring. The rationale is that raw UMI totals vary with capture efficiency, and the null models in Hotspot (danb/normal) explicitly condition expectations on per-cell UMI counts.

## Related tools

- **PhyloVision** (consumes the normalized counts as input to compute cell-level signature Z-scores and Geary's C phylogenetic autocorrelation) — https://github.com/YosefLab/VISION
- **VISION** (front-end and signature-scoring framework whose filterGenesFano function selects informative genes from the normalized matrix) — https://github.com/YosefLab/VISION
- **Hotspot** (scales expression by per-cell UMI counts (umi_counts_obs_key) when fitting null models for autocorrelation and module detection) — https://github.com/YosefLab/Hotspot

## Examples

```
import hotspot; hs = hotspot.Hotspot(adata, layer_key="counts", model='danb', latent_obsm_key="X_pca", umi_counts_obs_key="total_counts")
```

## Evaluation signals

- After normalization, every cell's total expression equals the clone's median UMI count (column sums are constant).
- filterGenesFano (default parameters) runs without error on the normalized matrix and returns a projection_genes set.
- Downstream signature Z-scores are centered per cell, and Geary's C' statistics fall in [0,1] with 1 indicating perfect phylogenetic autocorrelation.
- Cells retained in the normalized matrix exactly match the intersection of the expression matrix barcodes and the phylogeny leaves (1,127 cells for the Quinn et al. 2021 clone CP007).
- Empirical p-values from randomly generated signatures yield FDR values after Benjamini-Hochberg correction for every evaluated signature.

## Limitations

- Simple library-size scaling does not model zero inflation; dedicated methods (ZIFA, ZINB-WaVE, scvi-tools) exist but are not part of the PhyloVision pipeline.
- PhyloVision is implemented in R, which may create interoperability challenges with Python tools like Cassiopeia in the same workflow.
- Per-gene null expectations depend on the scaling factor: Hotspot's models (danb, bernoulli, normal) each condition on per-cell UMI counts differently, so inconsistent normalization between steps can invalidate the null model.

## Evidence

- [readme] The per-cell scaling factor - By default, number of umi per cell is used: [evidence span withheld — anchor status: external_doc]
- [readme] model is used to fit per-cell expectations for each gene assuming no correlations: [evidence span withheld — anchor status: external_doc]
- [methods] filterGenesFano with default parameters: "Informative genes were found using the filterGenesFano function in VISION with default parameters"
- [results] An autocorrelation statistic will be then computed by evaluating the consistency of the signature scores: [evidence span withheld — anchor status: not_found]
