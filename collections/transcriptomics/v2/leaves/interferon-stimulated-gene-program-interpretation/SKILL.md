---
name: interferon-stimulated-gene-program-interpretation
description: Use when analyzing interferon beta-stimulated PBMC scRNA-seq (or a similar
  perturbation dataset) and you need to extract a co-regulated gene program and verify
  it is not confounded by cell type composition.
when_to_use_negative:
- The question is about repression or mutually exclusive expression — GeneVector's
  signed MI cannot discern activation from repression direction.
- The dataset lacks both stimulated and control conditions or cell type annotations,
  making the cell-type-independent expression check infeasible.
- The goal is generic dimensionality reduction rather than testing whether a gene
  program is confounded by cell type — plain PCA/LDVAE alone would suffice for the
  former.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Trains mutual-information-based gene embeddings and clusters them into metagenes
    via get_adata(resolution=40) and get_metagenes
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads and manipulates the PBMC AnnData; computes per-cell scaled metagene
    expression and visualizes UMAPs
- name: AnnData
  role: Data container for raw counts in .X and per-cell metagene scores in obs
- name: PyTorch
  role: Backend for GeneVector neural embedding training (model.train(1000, threshold=1e-6))
- name: NumPy/SciPy
  role: Numerical computation of mean scaled metagene expression and PCA/LDVAE loading
    comparisons
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:ceeb595174a810b0279a4a3a1f292e039cf343faa07a755d7712d58d3c95b3d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - NumPy/SciPy
  - PyTorch
  - Python
  - Scanpy
  - SciPy
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# interferon-stimulated-gene-program-interpretation

## Summary

Identifies a cell-type-independent interferon-stimulated gene (ISG) metagene in interferon beta-stimulated PBMCs using GeneVector gene embeddings, and demonstrates that PCA/LDVAE loadings conflate ISGs with cell-type markers while GeneVector does not.

## When to use

Use when analyzing interferon beta-stimulated PBMC scRNA-seq (or a similar perturbation dataset) and you need to extract a co-regulated gene program and verify it is not confounded by cell type composition.

## When NOT to use

- The question is about repression or mutually exclusive expression — GeneVector's signed MI cannot discern activation from repression direction.
- The dataset lacks both stimulated and control conditions or cell type annotations, making the cell-type-independent expression check infeasible.
- The goal is generic dimensionality reduction rather than testing whether a gene program is confounded by cell type — plain PCA/LDVAE alone would suffice for the former.

## Inputs

- PBMC interferon beta stimulation scRNA-seq AnnData object with raw counts in .X (stimulated and control conditions, cell type annotations)
- Trained GeneVector gene embedding file genes.vec
- Marker-based cell type annotations (e.g., from cell_embed.phenotype_probability with T cell, B/plasma, myeloid marker sets)

## Outputs

- ISG metagene membership list (Leiden cluster of IFIT1/IFIT2/IFIT3/ISG15/ISG20/RSAD2/OAS1/MX1)
- Per-cell-type stimulated-vs-control scaled expression of the ISG metagene
- Contrast table of PCA PC2 and LDVAE latent loadings showing cell-type-marker contamination (CXCL10, IFITM3, PRF1, LAG3)
- isg_metagene_summary.csv combining membership, per-cell-type expression, and PCA/LDVAE contrast

## How to apply

Train a GeneVector embedding on the raw-count PBMC AnnData (GeneVectorDataset with entropy quality_control at entropy_threshold=1.0, GeneVector model trained 1000 iterations with threshold=1e-6, emb_dimension=100), then load the output genes.vec with GeneEmbedding(dataset, vector='average') and rank IFIT1's nearest neighbors via compute_similarities('IFIT1'). Cluster the embedding into metagenes using gene_embed.get_adata(resolution=40) and get_metagenes(gdata), selecting the Leiden cluster containing IFIT1/IFIT2/IFIT3/ISG15/ISG20/RSAD2/OAS1/MX1. Compute the mean scaled expression of the metagene genes per cell and compare stimulated versus control cells within each annotated cell type to confirm the program is cell-type independent. As a negative contrast, fit PCA (examine PC2 contributions) and a 10-latent-dimension LDVAE model on the same data and record how myeloid genes (CXCL10, IFITM3) and T cell markers (PRF1, LAG3) co-occur with ISG loadings in those linear models.

## Related tools

- **GeneVector** (Trains mutual-information-based gene embeddings and clusters them into metagenes via get_adata(resolution=40) and get_metagenes) — https://github.com/nceglia/genevector
- **Scanpy** (Loads and manipulates the PBMC AnnData; computes per-cell scaled metagene expression and visualizes UMAPs)
- **AnnData** (Data container for raw counts in .X and per-cell metagene scores in obs)
- **PyTorch** (Backend for GeneVector neural embedding training (model.train(1000, threshold=1e-6)))
- **NumPy/SciPy** (Numerical computation of mean scaled metagene expression and PCA/LDVAE loading comparisons)

## Examples

```
gene_embed = GeneEmbedding("genes.vec", dataset, vector="average"); gdata = gene_embed.get_adata(resolution=40); metagenes = gene_embed.get_metagenes(gdata); sims = gene_embed.compute_similarities("IFIT1")
```

## Evaluation signals

- The selected Leiden metagene contains only interferon pathway related genes (IFIT1, IFIT2, IFIT3, ISG15, ISG20, RSAD2, OAS1, MX1) with no cell-type markers.
- Mean scaled metagene expression is elevated in stimulated versus control cells within every annotated cell type, not restricted to one lineage.
- compute_similarities('IFIT1') ranks other ISGs as top nearest neighbors in the genes.vec embedding.
- The PCA PC2 / LDVAE loading contrast table reproduces the conflation: CXCL10 and IFITM3 (myeloid) and PRF1 and LAG3 (T cell) co-occur with ISG loadings in the linear models.
- isg_metagene_summary.csv is produced with metagene membership, per-cell-type expression, and PCA/LDVAE contrast columns populated.

## Limitations

- GeneVector builds MI that is agnostic to the direction of statistical dependency and cannot discern repression from activation.
- Empirical MI estimation is limited by the available number of cells, fidelity of the counts, and the discretization strategy; the MI calculation is computationally expensive (mitigated by numba/rust/CUDA backends or score caching).
- Results depend on the Leiden clustering resolution (resolution=40) and on the pre-selection of genes (seurat_v3 highly variable genes or entropy_threshold=1.0 quality control).

## Evidence

- [discussion] Cell-type-independent ISG metagene: "In interferon-stimulated PBMCs, we identify a cell type independent ISG metagene"
- [results] PCA/LDVAE conflation finding: [evidence span withheld — anchor status: not_found]
- [other] Workflow: embedding load and similarity ranking: [evidence span withheld — anchor status: not_found]
- [readme] Metagene clustering parameters: "# Generate metagenes through clustering
gene_adata = gene_embed.get_adata(resolution=40)
metagenes = gene_embed.get_metagenes(gene_adata)"
- [methods] Gene quality control threshold: "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
- [methods] Training parameters: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
