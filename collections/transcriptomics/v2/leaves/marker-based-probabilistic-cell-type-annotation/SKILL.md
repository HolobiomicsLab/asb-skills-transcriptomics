---
name: marker-based-probabilistic-cell-type-annotation
description: Use when you need to annotate cell types in a scRNA-seq AnnData object
  without reference mapping — e.g., HGSOC adnexa/bowel samples — using marker gene
  sets and validating labels against an independent annotator such as CellAssign.
when_to_use_negative:
- When a trusted reference-based annotator or published labels already exist and only
  need transfer — phenotype_probability is a marker-set-driven, reference-free method
- When the input lacks raw counts in .X (e.g., already log-normalized/feature-selected
  data), since GeneVector requires raw count data and entropy-based quality control
  on counts
- When marker gene sets are unavailable for the tissue's cell types — the annotation
  is only as good as the curated markers
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GeneVector
  role: Provides CellEmbedding and phenotype_probability to score marker gene sets
    and assign probabilistic cell type labels in the mutual-information latent space
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads the raw-count AnnData (.X) and visualizes the pseudo-probability columns
    on the UMAP embedding
- name: AnnData
  role: Container format holding the embedded and annotated cell data written to hgsoc_cell_annotations.h5ad
- name: PyTorch
  role: Backend used to train the GeneVector embedding model that the CellEmbedding
    depends on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:09781f1e5945f177050fc00e3957f04e5c9d83b3d23a27cc3b5ea7f4ad891c9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# marker-based-probabilistic-cell-type-annotation

## Summary

Annotates single-cell RNA-seq cell types probabilistically by projecting cells into a GeneVector latent space built from signed mutual-information gene embeddings, then scoring curated marker gene sets with CellEmbedding.phenotype_probability to assign pseudo-probability labels (e.g., six HGSOC cell types plus EPCAM/CD24-defined cancer cells).

## When to use

Use when you need to annotate cell types in a scRNA-seq AnnData object without reference mapping — e.g., HGSOC adnexa/bowel samples — using marker gene sets and validating labels against an independent annotator such as CellAssign.

## When NOT to use

- When a trusted reference-based annotator or published labels already exist and only need transfer — phenotype_probability is a marker-set-driven, reference-free method
- When the input lacks raw counts in .X (e.g., already log-normalized/feature-selected data), since GeneVector requires raw count data and entropy-based quality control on counts
- When marker gene sets are unavailable for the tissue's cell types — the annotation is only as good as the curated markers

## Inputs

- Scanpy AnnData object with raw counts in .X (e.g., HGSOC adnexa or bowel sample)
- Dictionary of marker gene sets per cell type (e.g., CD3D, CD3E, TRAC for T cells; EPCAM, CD24 for cancer cells)
- Trained genes.vec embedding file from GeneVector (emb_dimension=100, 1000 epochs)
- Optional independent reference labels (e.g., CellAssign annotations) for concordance checking

## Outputs

- AnnData with per-cell-type pseudo-probability scores in .obs columns (columns containing 'Pseudo-probability')
- Integrated cell embedding (.X and UMAP) in AnnData format from CellEmbedding.get_adata()
- Cell type label per cell (obs column 'genevector'), e.g., six HGSOC cell types plus cancer cells
- Annotated AnnData saved as hgsoc_cell_annotations.h5ad

## How to apply

First load raw-count AnnData in .X, apply GeneVectorDataset.quality_control with entropy_threshold=1.0, and train a GeneVector model (signed_mi=True target, emb_dimension=100, 1000 epochs, threshold=1e-6) writing genes.vec. Build a GeneEmbedding with vector='average' and derive a CellEmbedding, then call cell_embed.phenotype_probability(adata, markers) with a dictionary of marker genes per type (T Cell: CD3D/CD3G/CD3E/TRAC/IL32/CD2; B/Plasma: CD79A/CD79B/MZB1/CD19/BANK1; Myeloid: LYZ/CST3/AIF1/CD68/C1QA/C1QB/C1QC; plus EPCAM and CD24 for HGSOC cancer cells). In the HGSOC SPECTRUM workflow this produced 99.7% concordance with CellAssign in three cell types and ≥94.1% overall agreement, which serves as the accuracy benchmark. Decision point: if concordance falls well below these levels, revisit marker sets or entropy filtering before trusting downstream vector arithmetic restricted to a cell type (e.g., the Vadnexa − Vbowel cancer-cell vector).

## Related tools

- **GeneVector** (Provides CellEmbedding and phenotype_probability to score marker gene sets and assign probabilistic cell type labels in the mutual-information latent space) — https://github.com/nceglia/genevector
- **Scanpy** (Loads the raw-count AnnData (.X) and visualizes the pseudo-probability columns on the UMAP embedding)
- **AnnData** (Container format holding the embedded and annotated cell data written to hgsoc_cell_annotations.h5ad)
- **PyTorch** (Backend used to train the GeneVector embedding model that the CellEmbedding depends on)

## Examples

```
cell_embed = CellEmbedding(dataset, gene_embed); adata = cell_embed.phenotype_probability(adata_embedded, markers)
```

## Evaluation signals

- Concordance with CellAssign labels reaches 99.7% in three cell types and a minimum of 94.1% overall agreement
- Each cell type yields a 'Pseudo-probability' column in .obs; probabilities should be near-binary (close to 0 or 1) for clearly marked populations
- Annotated data saved as hgsoc_cell_annotations.h5ad retains raw counts, UMAP, and metagene scores without loss of cells
- Marker genes (e.g., EPCAM, CD24 for cancer cells) are expressed predominantly in the assigned population on the UMAP
- Latent-space arithmetic restricted to the annotated cancer-cell compartment (Vadnexa − Vbowel) recovers expected biology (EMT gain, MHC-I loss), confirming annotation correctness

## Limitations

- GeneVector is unable to discern repression from activation, as its mutual information target is agnostic to the direction of statistical dependency
- Empirical MI estimation is limited by the available number of cells, fidelity of counts, and discretization strategy; no improved estimator is provided
- Marker-set choice strongly determines labels; annotation quality should be validated against an independent annotator (e.g., CellAssign) as done here

## Evidence

- [other] annotate cell types with cell_embed.phenotype_probability using marker gene sets for the six cell types plus EPCAM and CD24 for cancer cells: [evidence span withheld — anchor status: not_found]
- [other] checking that accuracy reaches 99.7% in three cell types and a minimum of 94.1% overall agreement: [evidence span withheld — anchor status: not_found]
- [methods] adata = cell_embed.phenotype_probability(adata, markers): "adata = cell_embed.phenotype_probability(adata, markers)"
- [readme] Perform automated cell type assignment annotated_adata = cell_embed.phenotype_probability(adata_embedded, markers): "# Perform automated cell type assignment
annotated_adata = cell_embed.phenotype_probability(adata_embedded, markers)"
- [abstract] GeneVector uses latent space arithmetic in a lower dimensional gene embedding to identify transcriptional programs and classify cell types.: "GeneVector uses latent space arithmetic in a lower dimensional gene embedding to identify transcriptional programs and classify cell types."
