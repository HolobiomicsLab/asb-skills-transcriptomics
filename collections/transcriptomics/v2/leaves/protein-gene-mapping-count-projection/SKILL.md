---
name: protein-gene-mapping-count-projection
description: Use when a mosaic integration layout leaves feature entities (e.g., genes
  or proteins) unconnected by any shared data matrix, so scMoMaT would otherwise receive
  None placeholders for a modality in some batches.
when_to_use_negative:
- When every feature entity is already connected by jointly profiled data matrices
  and no None placeholders remain in the counts dictionary
- When the protein panel lacks a valid 1:1 gene counterpart for a protein (the projection
  relies on 1:1 mapping only)
- When the goal is cross-modality feature relationship learning itself — the paper
  defers feature representations to future work rather than providing this
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3169
tools:
- name: scMoMaT
  role: Target framework whose mosaic counts dictionary receives the pseudo-count
    matrices; supplies calc_pseudo_count/calc_pseudo_count.R and GxR.npz
  repo: https://github.com/PeterZZQ/scMoMaT
- name: LIGER
  role: Reference method whose gene activity score procedure scMoMaT's pseudo-count
    computation follows
  repo: https://github.com/welch-lab/liger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:aa7d01bde915b3f538564ff064e398ae318a63bba496932c924b090cb414b592
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - LIGER
  - scMoMaT
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Protein-gene mapping count projection

## Summary

Generates pseudo-count matrices for mosaic integration: a pseudo-scRNA-seq gene-activity matrix from scATAC-seq peaks (Seurat/LIGER-like procedure) and pseudo-protein counts from scRNA-seq via a 1:1 protein-gene mapping, filling the None placeholders in scMoMaT's counts dictionary when a batch lacks a modality.

## When to use

Use when a mosaic integration layout leaves feature entities (e.g., genes or proteins) unconnected by any shared data matrix, so scMoMaT would otherwise receive None placeholders for a modality in some batches.

## When NOT to use

- When every feature entity is already connected by jointly profiled data matrices and no None placeholders remain in the counts dictionary
- When the protein panel lacks a valid 1:1 gene counterpart for a protein (the projection relies on 1:1 mapping only)
- When the goal is cross-modality feature relationship learning itself — the paper defers feature representations to future work rather than providing this

## Inputs

- Raw scATAC-seq peak count matrix (ncells x npeaks) with gene annotations
- Pre-computed region-to-gene connection matrix GxR.npz from the scMoMaT repository
- scRNA-seq count matrix (ncells x ngenes)
- 1:1 protein-gene mapping table
- scMoMaT mosaic counts dictionary containing None placeholders for missing modalities

## Outputs

- Binarized pseudo-scRNA-seq (gene activity) count matrix
- Pseudo-protein count matrix (scRNA-seq counts projected onto protein features)
- Filled counts dictionary ready for utils.preprocess()

## How to apply

In R (calc_pseudo_count/calc_pseudo_count.R), aggregate per-cell scATAC-seq peak counts into per-gene accessibility scores by summing peaks falling within 2000 bp upstream of each gene's TSS plus the gene body, using the pre-computed region-to-gene connection matrix GxR.npz; then binarize the aggregated gene-activity matrix to obtain pseudo-scRNA-seq counts, mirroring the gene activity score procedure in Seurat and LIGER. Separately, project the scRNA-seq count matrix onto protein features using a 1:1 protein-gene mapping table to produce pseudo-protein counts. Insert both pseudo-count matrices into the mosaic counts dictionary in place of the None placeholders for batches lacking the corresponding modality, ensuring each modality list has length nbatches with matrices ordered by batch ID, then run utils.preprocess() on the filled matrices.

## Related tools

- **scMoMaT** (Target framework whose mosaic counts dictionary receives the pseudo-count matrices; supplies calc_pseudo_count/calc_pseudo_count.R and GxR.npz) — https://github.com/PeterZZQ/scMoMaT
- **LIGER** (Reference method whose gene activity score procedure scMoMaT's pseudo-count computation follows) — https://github.com/welch-lab/liger

## Examples

```
Rscript calc_pseudo_count/calc_pseudo_count.R  # then: counts['rna'][b] = pseudo_rna; counts['protein'][b] = pseudo_protein; utils.preprocess(counts)
```

## Evaluation signals

- Peak aggregation window matches the paper specification: 2000 bp upstream of each gene's TSS plus the gene body
- Gene activity matrix is binarized before use as pseudo-scRNA-seq counts
- Protein projection uses strictly 1:1 protein-gene mapping (no many-to-one expansion)
- Each modality list in counts has length equal to nbatches with no residual None for batches that received pseudo-counts, and matrices are ordered by batch ID
- Filled dictionary passes utils.preprocess() without errors

## Limitations

- No numeric threshold is reported for this stage — acceptance is qualitative against the 'Calculating Pseudo-Count Matrices' specification
- Learning cross-modality relationships via feature representations is deferred to future work, so pseudo-counts are an approximation, not learned feature coupling
- The GxR.npz connection matrix and 1:1 protein-gene mapping must be provided in advance; scMoMaT does not learn them

## Evidence

- [methods] pseudo-count computation follows Seurat/LIGER procedure: [evidence span withheld — anchor status: not_found]
- [methods] peak window specification: [evidence span withheld — anchor status: not_found]
- [readme] GxR.npz provided in advance: "The connection matrix (`GxR.npz`) that is used to generate pseudo-count is also provided in advance (used the code in folder `calc_pseudo_count/calc_pseudo_count.R`)"
- [readme] None placeholders in counts dictionary: "The batches that does not have count matrix in corresponding modality is inserted `None` as placeholder"
