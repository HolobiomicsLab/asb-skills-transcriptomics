---
name: clr-normalization-of-adt-counts
description: Use when a protein/ADT count matrix from CITE-seq or related assays must
  be normalized as a baseline (e.g., benchmarking against dsb), by applying CLR across
  cells.
when_to_use_negative:
- When the goal is to correct ambient background noise and cell-to-cell technical
  noise, CLR does not model empty-droplet background or isotype-control covariance
  — use dsb's ambient-correction and denoising steps instead
- When empty droplets or isotype controls are absent and only a low-level baseline
  scaling is needed, note that the paper defers caveats for isotype-free usage to
  the Supplementary Note rather than treating CLR as equivalent
- When the input is already normalized ADT data (e.g., dsb residuals or clipped values)
  — applying CLR again would double-normalize
tools:
- name: Seurat
  role: Provides NormalizeData with normalization.method = 'CLR' and margin = 2 for
    across-cell CLR normalization of the ADT matrix; also used for joint WNN clustering
    combining 30 mRNA PCs with CLR-normalized protein data
  repo: https://github.com/satijalab/seurat-data
- name: dsb
  role: Supplies the raw test data matrices (cells_citeseq_mtx, empty_drop_citeseq_mtx)
    and the comparator normalization method against which CLR is benchmarked
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Used by the dsb comparator method (Gaussian mixture fitting); not used by
    CLR itself
- name: limma
  role: Used by the dsb comparator method (removeBatchEffect matrix decomposition);
    not used by CLR itself
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:ab9e4a941f9f6dbd0bf5a2062c9e9ef35f6a6e6c13aa4fa709c2da71d572f927
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-data
  tools:
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# CLR normalization of ADT counts

## Summary

Applies centered log-ratio (CLR) transformation to antibody-derived tag (ADT) count matrices across cells as a baseline normalization for CITE-seq protein data, serving here as the comparator to dsb normalization in clustering-coherence and lineage-marker fold-change assessments.

## When to use

Use when a protein/ADT count matrix from CITE-seq or related assays must be normalized as a baseline (e.g., benchmarking against dsb), by applying CLR across cells.

## When NOT to use

- When the goal is to correct ambient background noise and cell-to-cell technical noise, CLR does not model empty-droplet background or isotype-control covariance — use dsb's ambient-correction and denoising steps instead
- When empty droplets or isotype controls are absent and only a low-level baseline scaling is needed, note that the paper defers caveats for isotype-free usage to the Supplementary Note rather than treating CLR as equivalent
- When the input is already normalized ADT data (e.g., dsb residuals or clipped values) — applying CLR again would double-normalize

## Inputs

- Raw cell-containing ADT UMI count matrix (cells x proteins), e.g. dsb::cells_citeseq_mtx
- Protein list filtered to those with maximum raw UMI count >= 5 across all cells

## Outputs

- CLR-normalized ADT matrix (same dimensions as input) suitable for clustering and fold-change analysis
- Comparison artifacts such as gap_statistic_table.csv and fold_change_comparison.csv contrasting CLR vs dsb

## How to apply

Start from the raw cell-containing droplet ADT count matrix (e.g., dsb::cells_citeseq_mtx) after filtering proteins with maximum raw UMI count < 5 across all cells. Apply CLR normalization across cells using Seurat::NormalizeData with margin = 2 (cells on the margin), running it in parallel to the dsb::DSBNormalizeProtein pipeline so that both methods receive identical input matrices. The CLR output is a normalized protein matrix on which the same downstream analyses (k-medoids clustering for k = 1..20, Gap-Statistic computation, cluster-level lineage-marker fold changes such as CD56 for NK and CD8/CD4 for T cells) are run as for dsb, enabling a like-for-like comparison. In the paper's workflow, the CLR-normalized protein values were also combined with 30 mRNA principal components for joint (WNN) clustering to assess how each protein normalization method affects multimodal clustering. Rationale: CLR is a standard baseline, and dsb was shown to yield higher Gap-Statistics (better within-cluster coherence) and sharper lineage-marker fold changes than CLR, consistent with dsb 'better unmask[s] biologically meaningful cell populations'.

## Related tools

- **Seurat** (Provides NormalizeData with normalization.method = 'CLR' and margin = 2 for across-cell CLR normalization of the ADT matrix; also used for joint WNN clustering combining 30 mRNA PCs with CLR-normalized protein data) — https://github.com/satijalab/seurat-data
- **dsb** (Supplies the raw test data matrices (cells_citeseq_mtx, empty_drop_citeseq_mtx) and the comparator normalization method against which CLR is benchmarked) — https://cran.r-project.org/package=dsb
- **mclust** (Used by the dsb comparator method (Gaussian mixture fitting); not used by CLR itself)
- **limma** (Used by the dsb comparator method (removeBatchEffect matrix decomposition); not used by CLR itself)

## Examples

```
clr_norm <- Seurat::NormalizeData(cell_protein_matrix, normalization.method = "CLR", margin = 2)
```

## Evaluation signals

- Gap-Statistic from k-medoids clustering (k = 1..20) on CLR-normalized protein-only data should be consistently LOWER than for dsb-normalized data, replicating the paper's Supplementary Fig. 3f,g finding
- Lineage-marker fold changes computed at cluster level (e.g., CD56 for NK, CD8/CD4 for T cells) should be less sharp under CLR than under dsb in fold_change_comparison.csv
- Output matrix must have identical dimensions and row/column ordering to the filtered input ADT matrix (proteins with max UMI < 5 already removed)
- When CLR-normalized protein values are combined with the same 30 mRNA principal components for joint clustering, cluster assignments should be comparable in structure to the dsb arm to ensure the comparison is like-for-like

## Limitations

- CLR provides no ambient-background correction from empty droplets and no technical-denoising via isotype controls, which the paper identifies as two major ADT noise sources
- The paper notes 'defining a reasonable subset of barcodes as background droplets still requires care' — this consideration affects dsb, but CLR's poorer downstream coherence (lower Gap-Statistics) is the relevant baseline limitation in this workflow
- Assessment details and caveats (robustness, batch effects, isotype-free usage) are deferred to the Supplementary Note, not the main text

## Evidence

- [discussion] CLR as baseline comparator in multimodal clustering: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [abstract] dsb outperforms CLR in unmasking cell populations: "Compared to existing normalization methods, our approach improves downstream analyses by better unmasking biologically meaningful cell populations"
- [discussion] Low-signal protein filtering before normalization: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization"
- [intro] Noise sources CLR does not address: [evidence span withheld — anchor status: not_found]
- [readme] Output interoperability with Seurat containers: [evidence span withheld — anchor status: external_doc]
