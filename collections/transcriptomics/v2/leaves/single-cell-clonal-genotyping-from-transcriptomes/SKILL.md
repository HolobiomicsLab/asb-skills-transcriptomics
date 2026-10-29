---
name: single-cell-clonal-genotyping-from-transcriptomes
description: Use when single-cell gene expression data carry linked per-cell mutation
  calls (GoT) that are noisy (false negatives from shallow sequencing, false positives
  from ambient RNA) and the goal is to reconcile genotype-based clone labels with
  transcriptional cell-type annotations.
when_to_use_negative:
- Cells have no linked mutation/genotype calls, so there is no second annotation modality
  to reconcile with expression-based clusters
- Only a single prior annotation set exists (no conflicting cluster labels), leaving
  nothing for decision-level integration to resolve
- The goal is data-level multimodal integration via a joint latent space (e.g., Seurat-WNN
  or TotalVI) rather than decision-level label reconciliation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scTriangulate
  role: Performs decision-level integration of the clonal-genotype annotations and
    cellHarmony cell-type labels using the Shapley value with stability scores; lazy_run
    with win_fraction_cutoff 0.25 and abs_thresh 10 yields the final per-cell clonal
    assignment
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Framework into which scTriangulate is embedded; used for QC, filtering, scaling,
    and UMAP generation via scanpy_recipe before integration
- name: cellHarmony
  role: Transfers reference annotations (Hay et al. CD34+ cell populations) onto the
    normalized MPN cells, providing the transcriptional annotation set to be reconciled
    with clone labels
- name: ICGS2
  role: Performs cell barcode filtering and normalization with default parameters
    prior to reference-mapping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:dd056660024a720662ecfe192b1046da2d82a193fca52abd59d040f14152328e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ICGS2
  - cellHarmony
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-clonal-genotyping-from-transcriptomes

## Summary

Resolve clonal architecture within molecularly defined single-cell populations by treating genotype-of-transcriptome (GoT) clonal assignments and reference cell-type labels as conflicting annotation sets, then letting scTriangulate's Shapley-value decision-level integration reassign cells (e.g., SF3B1 single-mutant cells in an MPN patient) to transcriptionally more similar clones.

## When to use

Use when single-cell gene expression data carry linked per-cell mutation calls (GoT) that are noisy (false negatives from shallow sequencing, false positives from ambient RNA) and the goal is to reconcile genotype-based clone labels with transcriptional cell-type annotations.

## When NOT to use

- Cells have no linked mutation/genotype calls, so there is no second annotation modality to reconcile with expression-based clusters
- Only a single prior annotation set exists (no conflicting cluster labels), leaving nothing for decision-level integration to resolve
- The goal is data-level multimodal integration via a joint latent space (e.g., Seurat-WNN or TotalVI) rather than decision-level label reconciliation

## Inputs

- Pre-processed gene expression count matrix (e.g., from GEO GSE117825) as h5ad/AnnData
- Pre-processed genotyping-of-transcriptomes (GoT) count matrix for the same cell barcodes
- Per-cell clonal genotype assignments (WT, SF3B1, SF3B1-CALR, SF3B1-CALR-NFE2)
- Reference-atlas cell-type labels from label transfer (cellHarmony / Hay et al. CD34+ populations)

## Outputs

- Cell-level table with one row per cell and final clonal genotype plus cell-type annotation columns
- Clone-by-cell-type Sankey diagram (plot_two_column_sankey) showing reassignment patterns
- Combined AnnData with reconciled annotations and UMAP coordinates

## How to apply

Download pre-processed gene-expression and genotyping count matrices (e.g., GEO GSE117825) and assign each cell to a clonal genotype (WT, SF3B1 mutant, SF3B1-CALR mutant, SF3B1-CALR-NFE2 mutant) using both wild-type and mutant allele calls, acknowledging shallow-sequencing false negatives and ambient-RNA false positives. Normalize the RNA matrix and annotate cells to a reference atlas (e.g., Hay et al. CD34+ populations via cellHarmony after ICGS2 processing). Build a combined AnnData with scTriangulate.preprocessing.concat_rna_and_other, supplying the cellHarmony clusters and clonal-genotype subclusters as conflicting annotation columns. Run scTriangulate 0.9.0 lazy_run with win_fraction_cutoff = 0.25 and default pruning (abs_thresh = 10): the Shapley value from coalitional game theory selects the stable annotation per cell (assessed via reassign, TFIDF5/TFIDF10, and SCCAF stability scores), and cells in unstable clusters (wfc < 0.25) are reclassified by nearest-neighbor. Export the cell-level clone reassignment table and render the clone-by-cell-type Sankey (plot_two_column_sankey) to inspect how, e.g., SF3B1 single-mutant cells shift to transcriptionally more similar clonal populations.

## Related tools

- **scTriangulate** (Performs decision-level integration of the clonal-genotype annotations and cellHarmony cell-type labels using the Shapley value with stability scores; lazy_run with win_fraction_cutoff 0.25 and abs_thresh 10 yields the final per-cell clonal assignment) — https://github.com/frankligy/scTriangulate
- **scanpy** (Framework into which scTriangulate is embedded; used for QC, filtering, scaling, and UMAP generation via scanpy_recipe before integration)
- **cellHarmony** (Transfers reference annotations (Hay et al. CD34+ cell populations) onto the normalized MPN cells, providing the transcriptional annotation set to be reconciled with clone labels)
- **ICGS2** (Performs cell barcode filtering and normalization with default parameters prior to reference-mapping)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['cellHarmony_clusters', 'genotype_subclusters']); sctri.lazy_run(win_fraction_cutoff=0.25)
```

## Evaluation signals

- Output table exists with exactly one row per cell and contains final clone and cell-type columns
- Sankey diagram reproduces the reported pattern of SF3B1 single-mutant cells moving to transcriptionally more similar clonal populations (Fig. 6c,d)
- Unstable clusters are correctly flagged by the winning-fraction rule (wfc < 0.25) and reclassified by nearest neighbor
- Final annotations honor the abs_thresh = 10 minimum cluster size so no spurious tiny clusters remain
- Reconciled clone assignments for reassigned cells show more similar transcriptomes than their original genotype-only assignments

## Limitations

- GoT calls are noisy: shallow sequencing yields false-negative allele calls and ambient RNA causes false positives, so genotype assignment must use both WT and mutant alleles
- Programmatic default settings may not be optimal for all study designs
- Integrating cluster predictions from many sources significantly increases runtime; Shapley selection is automatically replaced by rank-based selection when over 15 annotation sets are provided
- Current cancer analysis was limited to select somatic mutations within a single tumor

## Evidence

- [methods] Pre-processed count matrices for gene expression and genotyping of transcriptomes were downloaded from GEO (GSE117825).: "Pre-processed count matrices for gene expression and genotyping of transcriptomes were downloaded from GEO (GSE117825)."
- [methods] Shapley value decision rule: "By default, this strategy employs the Shapley value adopted from the coalitional game theory."
- [methods] stability scores: "scTriangulate uses four embedded scores: the reassign score, the TFIDF10 score, the TFIDF5 score, and the SCCAF score."
- [methods] unstable-cluster cutoff: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [abstract] clonal architecture capability: "including the ability to automate reference cell-atlas construction, resolve clonal architecture within molecularly defined cell-populations and subdivide clusters"
- [readme] integration settings scope: "Integrate results from both unsupervised and supervised (i.e. cellHarmony, Seurat label transfer) clustering algorithms."
