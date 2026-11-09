---
name: transfer-learning-label-propagation
description: Use when aligning cell types across two species from scRNA-seq data (e.g.
  human vs mouse primary motor cortex) where one-to-one orthologous subclasses must
  be matched and label-transfer recall quantified.
when_to_use_negative:
- Input is a single-species dataset with no cross-species comparison goal — within-species
  classification alone does not need label propagation
- Cell types lack comparable protein-coding ortholog information across species, so
  the ProtBERT-based gene similarity matrix cannot be computed reliably
- The comparison requires benchmarking against Seurat or SAMap rather than reproducing
  TACTiCS's own matching scores
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: TACTiCS
  role: Implements within-species neural-network cell-type classification and cross-species
    transfer-learning label propagation; TACTiCS class in tactics.py, saving via TACTiCS.save
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Backend framework required to train and run the TACTiCS model
- name: transformers
  role: Used only for embedding protein sequences that feed the gene similarity matrix
    (embed_proteins in genes.py)
- name: calc_dist
  role: Computes the gene similarity distance matrix input required by TACTiCS from
    protein embeddings
  repo: https://github.com/kbiharie/TACTiCS
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:3ee0df840a71e582cdaba7f8ac4101b7b69ef5fc391fe1c0ece1ceb532176978
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  - calc_dist
  - transformers
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# transfer-learning-label-propagation

## Summary

Cross-species cell type matching by propagating cell-type labels between species via transfer learning, as implemented in TACTiCS: genes are matched by protein-sequence embeddings, cell types are classified within species by a neural network, and labels are transferred between species to score subclass correspondences (e.g. all 23 shared human-mouse cortical subclasses matched correctly).

## When to use

Use when aligning cell types across two species from scRNA-seq data (e.g. human vs mouse primary motor cortex) where one-to-one orthologous subclasses must be matched and label-transfer recall quantified.

## When NOT to use

- Input is a single-species dataset with no cross-species comparison goal — within-species classification alone does not need label propagation
- Cell types lack comparable protein-coding ortholog information across species, so the ProtBERT-based gene similarity matrix cannot be computed reliably
- The comparison requires benchmarking against Seurat or SAMap rather than reproducing TACTiCS's own matching scores

## Inputs

- Two preprocessed scRNA-seq count matrices (raw counts, 50k-cell subsets of human and mouse primary motor cortex from Allen Brain M1)
- Gene similarity distance matrix computed by calc_dist from ProtBERT protein embeddings (embed_proteins in genes.py)
- Protein sequences and/or pretrained protein embeddings from Zenodo 10.5281/zenodo.7582460
- Optional: pretrained TACTiCS model checkpoints from the same Zenodo deposit

## Outputs

- Per-cell-type cross-species matching score matrix (e.g. subclass_matching_results.csv)
- Within-species classification recall and cross-species label-transfer recall (expected 1.0 and 0.96)
- Trained TACTiCS model, preprocessed data and cell embeddings saved via TACTiCS.save

## How to apply

Load two preprocessed scRNA-seq count datasets (e.g. the 50k-subset Allen Brain M1 human and mouse data from Zenodo 10.5281/zenodo.7582460) together with a gene similarity distance matrix, which is computed by calc_dist from protein embeddings created with embed_proteins in genes.py. Instantiate the TACTiCS class in tactics.py (PyTorch backend), restore or train the model, first performing within-species cell-type classification and then propagating cell-type labels between species via transfer learning. Compute the per-cell-type matching score matrix between subclasses (e.g. mouse Peri-human VLMC score 0.5, human VLMC-mouse VLMC score 1.0, species-specific pair scores 0.04-0.5). Verify results against expected metrics: correct match for all 23 shared cell types at subclass resolution, within-species recall 1.0, and cross-species-resolution recall 0.96. Persist preprocessed data, trained model and cell embeddings with TACTiCS.save.

## Related tools

- **TACTiCS** (Implements within-species neural-network cell-type classification and cross-species transfer-learning label propagation; TACTiCS class in tactics.py, saving via TACTiCS.save) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Backend framework required to train and run the TACTiCS model)
- **transformers** (Used only for embedding protein sequences that feed the gene similarity matrix (embed_proteins in genes.py))
- **calc_dist** (Computes the gene similarity distance matrix input required by TACTiCS from protein embeddings) — https://github.com/kbiharie/TACTiCS

## Examples

```
from tactics import TACTiCS; model = TACTiCS(data1, data2, dist_matrix); model.train(); scores = model.match_cell_types(); model.save('tactiCS_model')
```

## Evaluation signals

- Subclass-resolution matching: TACTiCS correctly returns the cell type match for all 23 cell types shared between human and mouse
- Within-species classification recall equals 1.0 on the Allen Brain M1 50k subsets
- Cross-species-resolution label-transfer recall equals 0.96
- Specific score anchors: mouse Peri-human VLMC score 0.5, human VLMC-mouse VLMC score 1.0, species-specific pair scores between 0.04 and 0.5
- Output table schema check: matching scores written to subclass_matching_results.csv and embeddings reproducible via TACTiCS.save

## Limitations

- TACTiCS was applied to scRNA-seq data of the primary motor cortex of human, mouse, and marmoset; performance on other tissues or more distant species is not established here
- For 50k-cell subsets, pretrained models and preprocessed data must be obtained from the Zenodo deposit (10.5281/zenodo.7582460); the full unprocessed dataset requires a separate download (10.5281/zenodo.6786357)
- PyTorch is required; the transformers package is only needed when re-embedding protein sequences, so skipping it assumes embeddings are already provided
- An alternative embedding component (e.g. scVI in place of ProtBERT-derived embeddings) changes the baseline and requires re-evaluation against the original 23/23 subclass matching result

## Evidence

- [abstract] Afterward, TACTiCS uses transfer learning to propagate cell type labels between species: "Afterward, TACTiCS uses transfer learning to propagate cell type labels between species"
- [methods] To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`.: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`."
- [methods] As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities.: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [results] At the subclass resolution, TACTiCS correctly returns the cell type match for all 23 cell types shared between human and mouse.: [evidence span withheld — anchor status: not_found]
- [readme] The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`.: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
