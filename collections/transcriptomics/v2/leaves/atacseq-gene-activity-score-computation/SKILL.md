---
name: atacseq-gene-activity-score-computation
description: Use when a mosaic integration layout contains feature entities (e.g.,
  chromatin regions and genes) that cannot be connected by any available input matrix,
  so a pseudo-scRNA-seq gene activity matrix must be derived from the scATAC-seq counts.
when_to_use_negative:
- The mosaic layout already provides matrices connecting all feature entities (e.g.,
  a paired scATAC-seq/scRNA-seq batch), so no pseudo-count imputation is needed
- The scATAC-seq data is already summarized as gene-level features rather than peak-level
  counts
- A different gene activity definition is required (this skill implements the specific
  2000 bp upstream of TSS plus gene body window used by scMoMaT, mirroring Seurat/LIGER)
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scMoMaT
  role: Provides the calc_pseudo_count R script and the GxR.npz region-to-gene connection
    matrix used to generate pseudo-counts; the pseudo matrices are inserted into its
    counts dictionary and preprocessed with utils.preprocess()
  repo: github.com/PeterZZQ/scMoMaT
- name: Seurat v3
  role: Reference implementation whose gene activity scoring procedure the scMoMaT
    pseudo-count calculation mirrors
- name: LIGER
  role: Reference implementation whose gene activity scoring procedure the scMoMaT
    pseudo-count calculation mirrors
  repo: welch-lab/liger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:bb1e49ef24eb63a254956f8d25e584f3dcaf0cc69eb83ab2d7843ead0ee305bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LIGER
  - Seurat v3
  - scMoMaT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# atacseq-gene-activity-score-computation

## Summary

Computation of pseudo-scRNA-seq (gene activity) scores and pseudo-protein counts from scATAC-seq peak matrices, as used by scMoMaT to bridge unconnected feature entities (regions/genes/proteins) in mosaic integration layouts.

## When to use

Use when a mosaic integration layout contains feature entities (e.g., chromatin regions and genes) that cannot be connected by any available input matrix, so a pseudo-scRNA-seq gene activity matrix must be derived from the scATAC-seq counts.

## When NOT to use

- The mosaic layout already provides matrices connecting all feature entities (e.g., a paired scATAC-seq/scRNA-seq batch), so no pseudo-count imputation is needed
- The scATAC-seq data is already summarized as gene-level features rather than peak-level counts
- A different gene activity definition is required (this skill implements the specific 2000 bp upstream of TSS plus gene body window used by scMoMaT, mirroring Seurat/LIGER)

## Inputs

- Raw scATAC-seq peak count matrix (ncells x nregions)
- Gene annotations (TSS coordinates)
- Pre-computed region-to-gene connection matrix GxR.npz
- scRNA-seq count matrix
- 1:1 protein-gene mapping table

## Outputs

- Binarized pseudo-scRNA-seq (gene activity) count matrix (ncells x ngenes)
- Pseudo-protein count matrix (ncells x nproteins)
- Filled mosaic counts dictionary with None placeholders replaced by pseudo-count matrices

## How to apply

Load the raw scATAC-seq peak count matrix together with gene annotations and the pre-computed region-to-gene connection matrix GxR.npz from the scMoMaT repository. Run the R script calc_pseudo_count/calc_pseudo_count.R, which aggregates per-cell peak counts into per-gene accessibility scores by summing peaks falling within 2000 bp upstream of each gene's TSS plus the gene body, following a procedure similar to Seurat and LIGER. Binarize the aggregated gene-activity matrix to obtain the pseudo-scRNA-seq counts. Then project the scRNA-seq counts onto protein features via a 1:1 protein-gene mapping table to produce pseudo-protein counts. Finally, insert both pseudo-count matrices into the mosaic counts dictionary in place of the None placeholders for batches lacking the corresponding modality, and run utils.preprocess() on the filled matrices.

## Related tools

- **scMoMaT** (Provides the calc_pseudo_count R script and the GxR.npz region-to-gene connection matrix used to generate pseudo-counts; the pseudo matrices are inserted into its counts dictionary and preprocessed with utils.preprocess()) — github.com/PeterZZQ/scMoMaT
- **Seurat v3** (Reference implementation whose gene activity scoring procedure the scMoMaT pseudo-count calculation mirrors)
- **LIGER** (Reference implementation whose gene activity scoring procedure the scMoMaT pseudo-count calculation mirrors) — welch-lab/liger

## Examples

```
Rscript calc_pseudo_count/calc_pseudo_count.R
```

## Evaluation signals

- Pseudo-count calculation uses a peak window of 2000 bp upstream of each gene's TSS plus the gene body, per the paper's 'Calculating Pseudo-Count Matrices' specification
- Aggregated gene-activity matrix is binarized before being used as pseudo-scRNA-seq counts
- Pseudo-protein counts are produced by projecting scRNA-seq counts through the 1:1 protein-gene mapping table (no arbitrary protein markers introduced)
- After insertion, the counts dictionary has lists of length nbatches with no None placeholders for the rna/protein modalities, and utils.preprocess() runs without error
- Pseudo-count matrices match the 'Calculating Pseudo-Count Matrices' specification in the paper's Methods (the qualitative acceptance criterion; no numeric threshold is reported for this stage)

## Limitations

- Pseudo-counts are an approximation of true measured counts: they substitute unmeasured modalities only where entities are unconnected by available input matrices
- Requires a pre-computed region-to-gene connection matrix (GxR.npz); the paper defers learning cross-modality relationships via feature representations to future work
- Feature sets must be matched in advance (gene intersection or peak remapping) before matrices can enter the counts dictionary

## Evidence

- [methods] Pseudo-count computation rationale: [evidence span withheld — anchor status: not_found]
- [readme] Connection matrix provided for pseudo-counts: "The connection matrix (`GxR.npz`) that is used to generate pseudo-count is also provided in advance"
- [readme] Feature matching requirement for counts dictionary: "The data matrix in each modality (each list) should have the same set of features. You can do thie by 1. using the intersection of all genes/proteins"
