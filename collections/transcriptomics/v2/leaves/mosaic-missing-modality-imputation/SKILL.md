---
name: mosaic-missing-modality-imputation
description: Use when a mosaic scMultiome layout contains batches lacking a modality
  (e.g., no scRNA-seq or CITE-seq protein counts) and feature entities such as peaks
  and genes cannot be connected by the available input data matrices.
when_to_use_negative:
- All modalities are jointly profiled in every batch (vertical integration), so measured
  counts exist and no pseudo-count substitution is needed
- No region-to-gene connection matrix (GxR.npz) is available for the genome build/peak
  set, since the gene-activity aggregation depends on it
- The missing modality has no 1:1 protein-gene mapping or gene-body/TSS annotation
  (e.g., non-transcriptomic modalities like motif deviation matrices), which the pseudo-count
  procedure does not cover
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMoMaT
  role: 'Host framework: consumes the filled counts dictionary (with pseudo-count
    matrices replacing None placeholders) via scmomat.scmomat_model after utils.preprocess()'
  repo: https://github.com/PeterZZQ/scMoMaT
- name: Seurat v3
  role: Reference implementation for the gene activity (pseudo-scRNA-seq) computation
    from scATAC-seq peaks
- name: LIGER
  role: Reference implementation whose gene-activity procedure the pseudo-count calculation
    emulates
  repo: https://github.com/welch-lab/liger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:102dbbe0b2f1174f9712ab29f32fb0cb5d5e5c1e2ba5597860bd002bef9c2518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - LIGER
  - Seurat v3
  - scMoMaT
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mosaic-missing-modality-imputation

## Summary

Generates pseudo-count matrices to fill modalities missing from mosaic single-cell multi-omics batches in scMoMaT: a binarized pseudo-scRNA-seq (gene activity) matrix derived from scATAC-seq peaks via a region-to-gene connection matrix, and pseudo-protein counts projected from scRNA-seq via a 1:1 protein-gene mapping.

## When to use

Use when a mosaic scMultiome layout contains batches lacking a modality (e.g., no scRNA-seq or CITE-seq protein counts) and feature entities such as peaks and genes cannot be connected by the available input data matrices.

## When NOT to use

- All modalities are jointly profiled in every batch (vertical integration), so measured counts exist and no pseudo-count substitution is needed
- No region-to-gene connection matrix (GxR.npz) is available for the genome build/peak set, since the gene-activity aggregation depends on it
- The missing modality has no 1:1 protein-gene mapping or gene-body/TSS annotation (e.g., non-transcriptomic modalities like motif deviation matrices), which the pseudo-count procedure does not cover

## Inputs

- scATAC-seq peak count matrix (ncells x npeaks) per batch
- Gene annotations (TSS coordinates, gene bodies)
- Precomputed region-to-gene connection matrix GxR.npz
- scRNA-seq count matrix (ncells x ngenes) per batch
- 1:1 protein-gene mapping table
- Mosaic counts dictionary with None placeholders for missing modalities (keys: nbatches, feats_name, per-modality lists)

## Outputs

- Binarized pseudo-scRNA-seq (gene activity score) matrix for batches lacking scRNA-seq
- Pseudo-protein count matrix for batches lacking CITE-seq protein measurements
- Filled counts dictionary ready for utils.preprocess() and scmomat_model training

## How to apply

Load the raw scATAC-seq peak count matrix, gene annotations, and the precomputed region-to-gene connection matrix GxR.npz from the scMoMaT repository. In R (calc_pseudo_count/calc_pseudo_count.R, following the procedure used in Seurat and LIGER), aggregate per-cell peak counts into per-gene accessibility scores by summing peaks within 2000 bp upstream of each gene's TSS plus the gene body, then binarize the aggregated gene-activity matrix to obtain the pseudo-scRNA-seq counts. Separately, project the scRNA-seq count matrix onto protein features using the 1:1 protein-gene mapping table to produce pseudo-protein counts. Insert both pseudo-count matrices into the mosaic counts dictionary in place of the None placeholders for batches lacking the corresponding modality (matrices must be ordered by batch ID so scMoMaT detects parallel sequenced batches), then run utils.preprocess() on the filled matrices before the first-stage scmomat_model training.

## Related tools

- **scMoMaT** (Host framework: consumes the filled counts dictionary (with pseudo-count matrices replacing None placeholders) via scmomat.scmomat_model after utils.preprocess()) — https://github.com/PeterZZQ/scMoMaT
- **Seurat v3** (Reference implementation for the gene activity (pseudo-scRNA-seq) computation from scATAC-seq peaks)
- **LIGER** (Reference implementation whose gene-activity procedure the pseudo-count calculation emulates) — https://github.com/welch-lab/liger

## Examples

```
Rscript calc_pseudo_count/calc_pseudo_count.R
```

## Evaluation signals

- Generated gene-activity matrix matches the 'Calculating Pseudo-Count Matrices' spec: peaks summed within 2000 bp upstream of TSS plus gene body, then binarized (qualitative acceptance criterion; no numeric threshold reported for this stage)
- Pseudo-protein counts equal the scRNA-seq values subset/projected through the 1:1 protein-gene mapping, with protein feature names matching feats_name['protein']
- Every per-modality list in the counts dictionary has length equal to nbatches and contains no residual None after insertion; matrices ordered by batch ID
- utils.preprocess() runs without error on the filled dictionary and feature counts match feats_name entries
- Pseudo-scRNA-seq output is consistent with a Seurat/LIGER-style gene activity computation on the same peak matrix and GxR.npz

## Limitations

- The pseudo-scRNA-seq matrix is a binarized, approximation-level surrogate of true gene expression, valid only when peaks and genes can be linked through the supplied GxR.npz connection matrix
- Only handles the RNA-from-ATAC and protein-from-RNA substitutions described in the pseudo-count script; other unconnected feature entities are not addressed
- Learning cross-modality relationships via feature representations is deferred to future work rather than addressed in this study

## Evidence

- [methods] pseudo-count mechanism rationale: [evidence span withheld — anchor status: not_found]
- [readme] pseudo-count provision in repo: "The connection matrix (`GxR.npz`) that is used to generate pseudo-count is also provided in advance (used the code in folder `calc_pseudo_count/calc_pseudo_count.R`)."
- [readme] None placeholder convention: "The batches that does not have count matrix in corresponding modality is inserted `None` as placeholder"
- [readme] feature matching requirement: "The data matrix in each modality (each list) should have the same set of features."
