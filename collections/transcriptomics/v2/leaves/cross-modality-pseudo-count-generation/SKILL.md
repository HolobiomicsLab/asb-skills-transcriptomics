---
name: cross-modality-pseudo-count-generation
description: Use when a mosaic multi-omics dataset (e.g. CITE-seq + ASAP-seq batches)
  has batch–modality slots with no count matrix and you need pseudo-counts for those
  slots before running scMoMaT first-stage training.
when_to_use_negative:
- All batch–modality slots already have real count matrices — no pseudo-counts are
  needed; proceed directly to utils.preprocess()
- Feature sets are not yet matched across batches — the intersection/remapping step
  must be done first, since each modality list must share the same feature set
- Using a baseline method (MultiMap, UINMF, StabMap, MultiMAP, LIGER) that does not
  consume the scMoMaT counts dictionary or GxR connection matrix
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMoMaT
  role: consumes the pseudo-count-completed counts dictionary for matrix tri-factorization;
    its calc_pseudo_count.R script and utils.preprocess() are used to build the connection
    matrix and normalize the filled counts
  repo: https://github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:a47906dc14a78718d850158165835f4c2013fdd03b83b2060000680e2ec48e0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - NumPy
  - PyTorch
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-modality-pseudo-count-generation

## Summary

Generates pseudo-count matrices for batch–modality slots missing in a mosaic single-cell multi-omics dataset, using a precomputed cross-modality connection matrix (GxR.npz), so that scMoMaT's matrix tri-factorization receives a complete count tensor.

## When to use

Use when a mosaic multi-omics dataset (e.g. CITE-seq + ASAP-seq batches) has batch–modality slots with no count matrix and you need pseudo-counts for those slots before running scMoMaT first-stage training.

## When NOT to use

- All batch–modality slots already have real count matrices — no pseudo-counts are needed; proceed directly to utils.preprocess()
- Feature sets are not yet matched across batches — the intersection/remapping step must be done first, since each modality list must share the same feature set
- Using a baseline method (MultiMap, UINMF, StabMap, MultiMAP, LIGER) that does not consume the scMoMaT counts dictionary or GxR connection matrix

## Inputs

- counts dictionary with keys nbatches, feats_name (per-modality feature name arrays), and per-modality lists of (ncells, nfeatures) matrices ordered by batch ID, with None placeholders for missing batch–modality slots
- precomputed cross-modality connection matrix GxR.npz (e.g. gene–peak linkage)
- feature-matched count matrices per modality (gene intersection or peak remapping already applied)

## Outputs

- pseudo-count matrices filling each missing batch–modality slot
- completed counts dictionary ready for utils.preprocess() and scmomat.scmomat_model construction

## How to apply

Assemble the counts dictionary with nbatches, feats_name, and per-modality lists ordered by batch ID, inserting None placeholders for every batch–modality slot that is missing. Obtain the connection matrix GxR.npz that links features across modalities (e.g. gene-to-peak associations), precomputed with the code in calc_pseudo_count/calc_pseudo_count.R from the scMoMaT repository. Use this connection matrix to generate pseudo-counts for each missing modality slot so every batch has an entry in every modality list, satisfying scMoMaT's requirement that lists have equal length equal to nbatches. Then preprocess each matrix (original and pseudo-count) with the scmomat utils.preprocess() function before constructing the scmomat_model with K=20–30 latent dimensions. The rationale is that scMoMaT's tri-factorization exploits cross-modality feature structure, so filled-in pseudo-counts let batches lacking a modality still contribute to and benefit from the shared factorization.

## Related tools

- **scMoMaT** (consumes the pseudo-count-completed counts dictionary for matrix tri-factorization; its calc_pseudo_count.R script and utils.preprocess() are used to build the connection matrix and normalize the filled counts) — https://github.com/PeterZZQ/scMoMaT

## Examples

```
Rscript calc_pseudo_count/calc_pseudo_count.R  # build GxR.npz connection matrix, then load it in Python to fill None slots in the counts dict before utils.preprocess()
```

## Evaluation signals

- Every per-modality list in the counts dictionary has length equal to nbatches, with no remaining None placeholders after pseudo-count generation
- Pseudo-count matrices have the same (ncells, nfeatures) shape and feature names as the real matrices of that modality, matching feats_name
- utils.preprocess() runs without error on all matrices, original and pseudo-count, before model construction
- scmomat.scmomat_model with K=30 trains for T=4000 iterations without shape or batch-alignment errors, and extract_cell_factors() returns one embedding per batch
- Downstream Leiden clustering (resolution=0.9) on the post-processed KNN graph yields labels that score sensibly against ground truth (e.g. GC/ARI/NMI vs Mimitou et al. PBMC labels, where scMoMaT outperformed MultiMap, UINMF, and StabMap)

## Limitations

- The connection matrix GxR.npz must be precomputed in advance using calc_pseudo_count/calc_pseudo_count.R; pseudo-count quality depends entirely on this linkage
- Feature sets must be matched across batches beforehand (gene intersection or peak remapping), otherwise pseudo-counts cannot be aligned to feats_name
- Learning cross-modality relationships via feature representations is deferred to future work rather than addressed in this study, so pseudo-counts are the mechanism used here to bridge missing modalities

## Evidence

- [readme] Connection matrix GxR.npz supplied in advance: "The connection matrix (`GxR.npz`) that is used to generate pseudo-count is also provided in advance"
- [readme] calc_pseudo_count.R script generates the connection matrix: "(used the code in folder `calc_pseudo_count/calc_pseudo_count.R`)"
- [readme] None placeholders required for missing batch–modality slots: "**The batches that does not have count matrix in corresponding modality is inserted `None` as placeholder**"
- [methods] Pseudo-count generation is a workflow step before scMoMaT training: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing after counts assembly: "preprocessed with `utils.preprocess()` function"
