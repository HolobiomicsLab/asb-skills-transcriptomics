---
name: latent-space-vector-arithmetic-site-comparison
description: Use when comparing the same cell type across two sample sites or conditions
  in scRNA-seq AnnData objects and you need site-/condition-specific gene signatures,
  such as EMT gain and MHC class I loss in metastasis.
when_to_use_negative:
- When the goal is to determine whether gene regulation is activating or repressing
  — GeneVector's mutual information is agnostic to the direction of statistical dependency
- When batches differ substantially in cell type composition and no manual verification
  of correction vectors is possible, since batch correction is unreliable in that
  setting
- When the input is not raw single-cell counts in an AnnData .X matrix (e.g., already-normalized
  or bulk feature tables)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Trains the mutual-information-based gene embedding, provides latent-space
    arithmetic, metagene clustering, and cell type annotation via phenotype_probability
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads AnnData objects with raw counts in .X and visualizes cell embeddings
    and annotation pseudo-probabilities on UMAP
- name: AnnData
  role: Container format for the per-site single-cell matrices and the annotated output
    with cell type probabilities and metagene scores
- name: PyTorch
  role: Neural embedding backend used to train the GeneVector model for 1000 epochs
    with threshold=1e-6
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:640fac0c255589eb09578b5f53a50566c344520023f05adefaae84bcb238a559
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# latent-space-vector-arithmetic-site-comparison

## Summary

Identify transcriptional programs that differ between anatomical sites (e.g., primary adnexa vs. metastatic bowel in HGSOC) by computing directional vectors (Vadnexa − Vbowel) in the GeneVector latent space and scoring clustered metagenes against them.

## When to use

Use when comparing the same cell type across two sample sites or conditions in scRNA-seq AnnData objects and you need site-/condition-specific gene signatures, such as EMT gain and MHC class I loss in metastasis.

## When NOT to use

- When the goal is to determine whether gene regulation is activating or repressing — GeneVector's mutual information is agnostic to the direction of statistical dependency
- When batches differ substantially in cell type composition and no manual verification of correction vectors is possible, since batch correction is unreliable in that setting
- When the input is not raw single-cell counts in an AnnData .X matrix (e.g., already-normalized or bulk feature tables)

## Inputs

- Scanpy AnnData objects (.h5ad) with raw counts in .X for each tumor site (e.g., HGSOC adnexa and bowel)
- Marker gene sets for cell type annotation (e.g., EPCAM, CD24 for cancer cells; T/B/myeloid markers)
- Optional reference cell type labels (e.g., CellAssign) for annotation concordance checks

## Outputs

- Trained gene vector file (genes.vec, emb_dimension=100)
- Site-directional vector (e.g., Vadnexa − Vbowel) restricted to cancer cells
- Clustered metagenes from the gene co-expression graph (resolution=40) with site-direction scores
- Annotated AnnData with cell type pseudo-probabilities and metagene scores (e.g., hgsoc_cell_annotations.h5ad)

## How to apply

Load each site's AnnData with raw counts in .X via Scanpy and apply entropy-based gene quality control (GeneVectorDataset.quality_control with entropy_threshold=1.0). Build a GeneVectorDataset using signed mutual information (signed_mi=True, mi_backend='auto' with caching to ~/.genevector/cache/) and train GeneVector with emb_dimension=100 for 1000 epochs (threshold=1e-6) to produce genes.vec. Construct a GeneEmbedding (vector='average'), derive a CellEmbedding, annotate cell types with cell_embed.phenotype_probability using marker sets (e.g., EPCAM and CD24 for cancer cells), and validate against CellAssign (expect ~99.7% concordance in three cell types, ≥94.1% overall). Restrict to the cell type of interest (cancer cells) and compute the directional vector Vadnexa − Vbowel; cluster the gene embedding with embed.get_adata(resolution=40), extract metagenes via embed.get_metagenes, and score each metagene against the directional vector to identify Hallmark-enriched programs gained or lost at the metastatic site.

## Related tools

- **GeneVector** (Trains the mutual-information-based gene embedding, provides latent-space arithmetic, metagene clustering, and cell type annotation via phenotype_probability) — https://github.com/nceglia/genevector
- **Scanpy** (Loads AnnData objects with raw counts in .X and visualizes cell embeddings and annotation pseudo-probabilities on UMAP)
- **AnnData** (Container format for the per-site single-cell matrices and the annotated output with cell type probabilities and metagene scores)
- **PyTorch** (Neural embedding backend used to train the GeneVector model for 1000 epochs with threshold=1e-6)

## Examples

```
dataset = GeneVectorDataset(adata, signed_mi=True, mi_backend='auto'); model = GeneVector(dataset, output_file='genes.vec', emb_dimension=100); model.train(1000, threshold=1e-6)
```

## Evaluation signals

- Cell type annotation concordance with CellAssign reaches 99.7% in three cell types and a minimum of 94.1% overall agreement
- Metagenes scored against the directional vector recover Hallmark-enriched programs, with EMT pathway genes gained and MHC class I genes (HLA-A, HLA-B, HLA-C, HLA-E, HLA-F) plus B2M lost from adnexa to bowel in cancer cells
- MI score cache appears under ~/.genevector/cache/ and reloads instantly on re-runs with the same data and parameters
- The output AnnData contains per-cell-type 'Pseudo-probability' columns and metagene score columns and saves cleanly to .h5ad
- Training converges within 1000 iterations (or earlier) with the threshold=1e-6 stopping criterion

## Limitations

- GeneVector cannot distinguish activation from repression, as MI is agnostic to the direction of statistical dependency
- Empirical MI estimation is limited by the available number of cells, fidelity of counts, and the discretization strategy; the MI calculation is computationally expensive (up to ~26 hours legacy for 2,000 genes × 5,000 cells without accelerated backends)
- Batch correction is unreliable when batches differ in cell type composition and must be manually verified against cell type signatures

## Evidence

- [intro] We use vector space arithmetic to directly map metagenes to site specific changes in primary and metastatic sites in the HGSOC dataset, capturing changes in MHC class I expression and: "We use vector space arithmetic to directly map metagenes to site specific changes in primary and metastatic sites in the HGSOC dataset, capturing changes in MHC class I expression and"
- [methods] training parameters: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [methods] quality control: "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
- [readme] metagene clustering: "# Generate metagenes through clustering
gene_adata = gene_embed.get_adata(resolution=40)
metagenes = gene_embed.get_metagenes(gene_adata)"
- [discussion] MI directionality limitation: "GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency"
