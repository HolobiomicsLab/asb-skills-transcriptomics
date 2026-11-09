---
name: per-cell-type-scaled-expression-differential-analysis
description: Use when a gene metagene or embedding-derived gene program must be validated
  as cell-type-independent by comparing per-cell scaled expression of its member genes
  between conditions (e.g., stimulated vs control) within each annotated cell type.
when_to_use_negative:
- When the goal is batch correction or cell type annotation rather than validating
  a metagene's cell-type independence — use CellEmbedding batch_correct or phenotype_probability
  instead.
- When genes must be distinguished as activated versus repressed — GeneVector's mutual
  information is agnostic to the direction of statistical dependency, so scaled-expression
  contrasts cannot establish repression.
- When the input is not an AnnData object with raw counts in .X and existing cell
  type annotations; the within-cell-type comparison requires an obs-level cell type
  key.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GeneVector
  role: Trains the gene embedding and derives the metagenes whose scaled expression
    is compared per cell type; GeneEmbedding.get_adata(resolution=40) and get_metagenes
    produce the metagene membership
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Provides the AnnData object with raw counts in .X and utilities for per-cell-type
    expression computation and annotation
  repo: https://github.com/nceglia/genevector
- name: AnnData
  role: Container holding the PBMC expression matrix, condition labels, and cell type
    annotations used for the stratified comparison
  repo: https://github.com/nceglia/genevector
- name: NumPy
  role: Array arithmetic for mean scaled expression per cell across metagene genes
- name: PyTorch
  role: Backend dependency for training the GeneVector embedding; also used to fit
    the 10-latent-dimension LDVAE contrast model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:b81ba129ed6fc8f34cb47c90c1e03b7eb9df1d1576fe628a757f0e1fd290581c
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
  - SciPy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-cell-type scaled expression differential analysis

## Summary

Confirms that a GeneVector-derived metagene (e.g., the IFIT1-containing ISG program in interferon beta-stimulated PBMCs) is elevated across every annotated cell type by computing mean scaled expression per cell in stimulated versus control conditions within each cell type, avoiding the cell-type confounding seen in PCA and LDVAE loadings.

## When to use

Use when a gene metagene or embedding-derived gene program must be validated as cell-type-independent by comparing per-cell scaled expression of its member genes between conditions (e.g., stimulated vs control) within each annotated cell type.

## When NOT to use

- When the goal is batch correction or cell type annotation rather than validating a metagene's cell-type independence — use CellEmbedding batch_correct or phenotype_probability instead.
- When genes must be distinguished as activated versus repressed — GeneVector's mutual information is agnostic to the direction of statistical dependency, so scaled-expression contrasts cannot establish repression.
- When the input is not an AnnData object with raw counts in .X and existing cell type annotations; the within-cell-type comparison requires an obs-level cell type key.

## Inputs

- Scanpy AnnData object with raw counts in .X and cell type annotations in .obs
- GeneEmbedding object loaded from the trained genes.vec file (vector='average')
- Condition labels distinguishing interferon beta-stimulated versus control cells
- Metagene membership list (Leiden cluster containing IFIT1/IFIT2/IFIT3/ISG15/ISG20/RSAD2/OAS1/MX1)

## Outputs

- Per-cell-type table of mean scaled metagene expression for stimulated vs control cells
- isg_metagene_summary.csv containing metagene membership, per-cell-type expression comparisons, and the PCA/LDVAE contrast table
- Record of PC2 contributions and LDVAE latent loadings showing confounded genes (CXCL10, IFITM3, PRF1, LAG3)

## How to apply

After extracting the metagene from the gene embedding (build the embedding AnnData with get_adata(resolution=40), extract metagenes with get_metagenes, and select the Leiden cluster containing IFIT1/IFIT2/IFIT3/ISG15/ISG20/RSAD2/OAS1/MX1), compute the mean scaled expression of the metagene's genes per cell in the PBMC AnnData. Stratify cells by annotated cell type and compare stimulated versus control cells within each cell type rather than across pooled populations, since pooling conflates the ISG program with cell-type composition. Record per-cell-type stimulated-vs-control scaled expression in a summary table (isg_metagene_summary.csv) alongside metagene membership. As a negative contrast, fit PCA and a 10-latent-dimension LDVAE model on the same data and document how CXCL10, IFITM3, PRF1, and LAG3 co-occur with ISG loadings in the linear models, demonstrating the confounding that the metagene analysis avoids.

## Related tools

- **GeneVector** (Trains the gene embedding and derives the metagenes whose scaled expression is compared per cell type; GeneEmbedding.get_adata(resolution=40) and get_metagenes produce the metagene membership) — https://github.com/nceglia/genevector
- **Scanpy** (Provides the AnnData object with raw counts in .X and utilities for per-cell-type expression computation and annotation) — https://github.com/nceglia/genevector
- **AnnData** (Container holding the PBMC expression matrix, condition labels, and cell type annotations used for the stratified comparison) — https://github.com/nceglia/genevector
- **NumPy** (Array arithmetic for mean scaled expression per cell across metagene genes)
- **PyTorch** (Backend dependency for training the GeneVector embedding; also used to fit the 10-latent-dimension LDVAE contrast model)

## Examples

```
gene_embed = GeneEmbedding("genes.vec", dataset, vector="average"); metagenes = gene_embed.get_metagenes(gene_embed.get_adata(resolution=40))
```

## Evaluation signals

- Every annotated cell type shows elevated mean scaled metagene expression in stimulated vs control cells (the metagene is cell-type-independent)
- The selected Leiden metagene contains only interferon pathway related genes — cell-type markers such as CXCL10, IFITM3, PRF1, and LAG3 are absent from the metagene but appear in PCA PC2 and LDVAE loadings
- Metagene membership matches the expected ISG set: IFIT1, IFIT2, IFIT3, ISG15, ISG20, RSAD2, OAS1, MX1
- isg_metagene_summary.csv contains all three blocks: metagene membership, per-cell-type stimulated-vs-control scaled expression, and the PCA/LDVAE contrast table
- The input AnnData carried raw counts in .X and gene filtering (e.g., entropy_threshold=1.0 or seurat_v3 HVG selection) was applied before embedding training

## Limitations

- Mutual information in GeneVector is agnostic to the direction of statistical dependency, so it cannot discern repression from activation; the scaled-expression contrast only detects elevation, not repression
- Empirical MI estimation is limited by the available number of cells, fidelity of counts, and discretization strategy (zero maps to bin 0, nonzero values split into 10 quantile-based bins)
- The MI calculation is computationally expensive (legacy v0.2 takes ~26 hours for 2,000 genes × 5,000 cells without accelerated backends), though results are cached in ~/.genevector/cache/

## Evidence

- [discussion] cell type independence of ISG metagene: "In interferon-stimulated PBMCs, we identify a cell type independent ISG metagene"
- [readme] readme metagene extraction: "# Generate metagenes through clustering

gene_adata = gene_embed.get_adata(resolution=40)

metagenes = gene_embed.get_metagenes(gene_adata)"
- [readme] readme data format requirement: [evidence span withheld — anchor status: external_doc]
- [methods] entropy QC threshold: "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
- [methods] signing of MI: "When ``signed_mi=True`` (default), MI is multiplied by the sign of the Pearson
correlation between the two genes."
