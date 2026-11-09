---
name: cross-species-embedding-integration-interpretation
description: Use when trained cross-species cell embeddings exist and you need to
  judge cell-type-level alignment across species, e.g. homologous types (Vip, Sncg)
  mixing while species-specific types (Meis2, pericytes, SMC) stay separated.
when_to_use_negative:
- You need cell-type matches with confidence scores rather than qualitative visual
  alignment — use TACTiCS label transfer/evaluation instead of UMAP interpretation.
- 'The embeddings have not been trained yet: TACTiCS first requires protein-sequence-based
  gene matching (`embed_proteins`, `calc_dist`) and transfer learning before embeddings
  exist to visualize.'
- Data are raw/unprocessed counts — use the preprocessed 50k subsets from Zenodo rather
  than integrating raw matrices directly.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: TACTiCS
  role: Provides the trained cross-species cell embeddings that are visualized and
    interpreted; models are loaded via the `TACTiCS` class in `tactics.py` and saved
    with `TACTiCS.save`
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Framework required to load the TACTiCS models and extract cell embeddings
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:72ac3f3e84d6e751f72bea52681e7080df12983c0b223d8724676b00965628e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-species-embedding-integration-interpretation

## Summary

Visualize and interpret cross-species cell embeddings (e.g. TACTiCS-trained embeddings of human, mouse, and marmoset motor cortex scRNA-seq cells) in 2D with UMAP to assess whether homologous cell types align and species-specific types separate.

## When to use

Use when trained cross-species cell embeddings exist and you need to judge cell-type-level alignment across species, e.g. homologous types (Vip, Sncg) mixing while species-specific types (Meis2, pericytes, SMC) stay separated.

## When NOT to use

- You need cell-type matches with confidence scores rather than qualitative visual alignment — use TACTiCS label transfer/evaluation instead of UMAP interpretation.
- The embeddings have not been trained yet: TACTiCS first requires protein-sequence-based gene matching (`embed_proteins`, `calc_dist`) and transfer learning before embeddings exist to visualize.
- Data are raw/unprocessed counts — use the preprocessed 50k subsets from Zenodo rather than integrating raw matrices directly.

## Inputs

- Trained TACTiCS models (deposited on Zenodo, doi:10.5281/zenodo.7582460)
- 50k-subset preprocessed scRNA-seq data of primary motor cortex (human, mouse, marmoset) from the same Zenodo deposit
- Cell-type annotations per species

## Outputs

- 2D UMAP scatter plots colored by cell type and species (PNG files)
- Per-cell-type table of 2D embedding coordinates (TSV file)

## How to apply

Load the deposited trained TACTiCS models and 50k-subset preprocessed scRNA-seq data from Zenodo (doi:10.5281/zenodo.7582460), then use the `TACTiCS` class in `tactics.py` (PyTorch) to extract cell embeddings for each species. Reduce the embeddings to two dimensions with UMAP and render scatter plots colored by cell type and species. Interpret the plots at the cell type level rather than the individual-cell level: in the published result, individual human and mouse cells do not mix well, but corresponding cell types overlap partially or sit relatively close, while mouse-specific Meis2, pericytes, and smooth muscle cells cluster apart from human cells. Export the UMAP figures as PNG and write a per-cell-type table of embedding coordinates as TSV for downstream checking.

## Related tools

- **TACTiCS** (Provides the trained cross-species cell embeddings that are visualized and interpreted; models are loaded via the `TACTiCS` class in `tactics.py` and saved with `TACTiCS.save`) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Framework required to load the TACTiCS models and extract cell embeddings)

## Examples

```
model = TACTiCS.load('zenodo_7582460_trained_model'); emb = model.get_embeddings(); umap_2d = umap.UMAP(n_components=2).fit_transform(emb)
```

## Evaluation signals

- Homologous cell types across species (e.g. Vip and Sncg) form partially overlapping or proximate clusters in the 2D UMAP.
- Mouse-specific types (Meis2, pericytes, smooth muscle cells) cluster separately from human cells.
- Individual human and mouse cells do not fully mix — expect partial, not complete, species mixing.
- Per-cell-type TSV of embedding coordinates matches the rendered UMAP point positions.
- Embeddings are extracted from the deposited trained models (Zenodo 7582460) rather than retrained, ensuring reproducibility.

## Limitations

- Interpretation must be at the cell type level: individual cells of human and mouse do not mix well in the TACTiCS embedding space, so per-cell mixing is the wrong success criterion.
- Alignment is qualitative (partial overlap or proximity of clusters); UMAP does not provide quantitative cross-species match scores.
- Published demonstration is on primary motor cortex scRNA-seq of human, mouse, and marmoset (50k subsets); generalization to other tissues/species pairs is not established by this evidence.

## Evidence

- [other] In the TACTiCS embedding space, individual human and mouse cells do not mix well, but the UMAP aligns at the cell type level: corresponding cell types either overlap partially or are relatively close: [evidence span withheld — anchor status: not_found]
- [readme] To train and evaluate TACTiCS, use the TACTiCS class: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`."
- [abstract] The preprocessed datasets and trained models can be downloaded from Zenodo: "The preprocessed datasets and trained models can be downloaded from Zenodo (https://doi.org/10.5281/zenodo.7582460)"
- [readme] TACTiCS requires PyTorch.: "TACTiCS requires PyTorch."
