---
name: lineage-marker-fold-change-quantification
description: 'Use when comparing normalization methods on CITE-seq protein data: after
  assigning cluster-level cell identities, compute lineage-marker fold changes per
  cluster to measure which normalization (dsb or CLR) yields sharper lineage separation.'
when_to_use_negative:
- The dataset has no antibody-derived tag (ADT) measurements — this skill operates
  on protein CITE-seq matrices, not mRNA-only data
- Clusters have not yet been assigned or validated — fold changes require cluster-level
  cell identities first
- The lineage marker of interest was removed by QC (proteins with maximum raw UMI
  count < 5 across all cells were dropped, e.g., CD34 in two datasets)
edam_operation: http://edamontology.org/operation_3223
tools:
- name: dsb
  role: Normalization under test; DSBNormalizeProtein produces denoised, background-scaled
    values whose cluster-level marker fold changes are quantified
  repo: https://cran.r-project.org/package=dsb
- name: Seurat
  role: Provides the CLR baseline via NormalizeData with margin=2 on the protein matrix
- name: mclust
  role: Fits the per-cell 2-component Gaussian mixture used by dsb to estimate the
    background mean for denoising before fold-change computation
- name: limma
  role: removeBatchEffect() is used within dsb for robust matrix decomposition of
    the technical noise component
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:7295d4534473d233753a0440e33b8a9c4f54094d310b1e8df5852308a46ec647
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_3223
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# lineage-marker-fold-change-quantification

## Summary

Quantify per-cluster lineage-marker fold changes (e.g., CD56 for NK cells, CD8/CD4 for T cells) on normalized CITE-seq ADT matrices to benchmark how well a normalization method (dsb vs CLR) un-masks biologically meaningful cell populations.

## When to use

Use when comparing normalization methods on CITE-seq protein data: after assigning cluster-level cell identities, compute lineage-marker fold changes per cluster to measure which normalization (dsb or CLR) yields sharper lineage separation.

## When NOT to use

- The dataset has no antibody-derived tag (ADT) measurements — this skill operates on protein CITE-seq matrices, not mRNA-only data
- Clusters have not yet been assigned or validated — fold changes require cluster-level cell identities first
- The lineage marker of interest was removed by QC (proteins with maximum raw UMI count < 5 across all cells were dropped, e.g., CD34 in two datasets)

## Inputs

- Raw cell-containing droplet ADT matrix (e.g., dsb::cells_citeseq_mtx, >50,000 cells x 87 antibodies PBMC CITE-seq)
- Empty-droplet background ADT matrix (e.g., dsb::empty_drop_citeseq_mtx)
- Isotype control antibody rows (rows 67:70)
- Cluster/cell-identity assignments from k-medoids clustering of the normalized protein matrices

## Outputs

- fold_change_comparison.csv — lineage-marker fold changes (CD56, CD8, CD4, etc.) per cluster under dsb and CLR normalization
- gap_statistic_table.csv — one row per k (1..20), columns for dsb and CLR Gap-Statistics

## How to apply

First normalize the ADT matrix two ways in parallel: DSBNormalizeProtein (denoise.counts=TRUE, use.isotype.control=TRUE with isotype controls from rows 67:70, quantile.clipping=TRUE, quantile.clip=c(0.001, 0.9995)) and Seurat::NormalizeData with margin=2 (CLR). Run k-medoids clustering on each normalized protein matrix (k=1..20), assign cluster-level cell identities, then compute the mean normalized expression of lineage markers within each assigned cluster versus the rest — canonical markers include CD56 for NK cells and CD8/CD4 for T cells. The rationale, per the dsb paper, is that a normalization that 'better unmasks biologically meaningful cell populations' produces sharper marker fold changes; dsb removes ambient background (empty-droplet-derived) and cell-to-cell technical noise (isotype/background shared variance via mclust GMM + PC1 regression), so its residuals should give cleaner marker separation. Write results to fold_change_comparison.csv with one row per marker/cluster/method pair alongside gap_statistic_table.csv.

## Related tools

- **dsb** (Normalization under test; DSBNormalizeProtein produces denoised, background-scaled values whose cluster-level marker fold changes are quantified) — https://cran.r-project.org/package=dsb
- **Seurat** (Provides the CLR baseline via NormalizeData with margin=2 on the protein matrix)
- **mclust** (Fits the per-cell 2-component Gaussian mixture used by dsb to estimate the background mean for denoising before fold-change computation)
- **limma** (removeBatchEffect() is used within dsb for robust matrix decomposition of the technical noise component)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE, isotype.control.name.vec = rownames(cells_citeseq_mtx)[67:70], quantile.clipping = TRUE, quantile.clip = c(0.001, 0.9995))
```

## Evaluation signals

- fold_change_comparison.csv contains a row per marker x cluster x method with fold changes for both dsb and CLR columns
- dsb-normalized data should show sharper (higher-magnitude) lineage-marker fold changes than CLR, consistent with Supplementary Fig. 3f,g
- CD56 fold change should be enriched in the NK-cell cluster and CD8/CD4 in their respective T-cell clusters under dsb
- Gap-Statistics from gap_statistic_table.csv should be consistently higher for dsb than CLR across k=1..20
- Normalized dsb values should be residual-scaled with extreme values clipped at quantiles c(0.001, 0.9995)

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note
- Assessment details (noise-estimation robustness, empty-droplet sensitivity, batch effects) are in the Supplementary Note, not the main text

## Evidence

- [abstract] dsb improves downstream clustering coherence: "Compared to existing normalization methods, our approach improves downstream analyses by better unmasking biologically meaningful cell populations"
- [discussion] Joint clustering basis for cluster identity assignment: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [discussion] Low-marker QC filter before normalization: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization"
- [methods] DSBNormalizeProtein call grounded in methods: "dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx"
- [readme] Outputs integrate into analysis containers: [evidence span withheld — anchor status: external_doc]
