---
name: probabilistic-marker-based-cell-assignment
description: Use when raw-count scRNA-seq AnnData needs automated cell type annotation
  from curated marker gene sets, and predicted labels should be validated against
  existing published cell type annotations.
when_to_use_negative:
- Input .X contains normalized, log-transformed, or log-normalized values instead
  of raw counts — the TICA benchmark (Supplementary Fig. 2C, D) shows such preprocessing
  severely degrades assignment accuracy
- No reliable marker gene sets exist for the cell types of interest, since assignment
  quality depends entirely on the curated markers supplied to phenotype_probability
- Genes have not passed entropy-based QC (entropy_threshold=1.0) or other variable-gene
  filtering, leaving low-information genes in the embedding
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: GeneVector
  role: Trains mutual-information-based gene embeddings and provides CellEmbedding.phenotype_probability
    for marker-set-based probabilistic cell type assignment
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Hosts the raw-count AnnData input and renders UMAP visualizations of annotated
    cells and pseudo-probability scores
  repo: https://github.com/nceglia/genevector
- name: AnnData
  role: Data container carrying raw counts in .X and receiving pseudo-probability
    and genevector label columns in .obs
- name: PyTorch
  role: Backend for training the neural gene embedding (model.train(1000, threshold=1e-6))
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:101b0fe163ad1e50b633921a68ee0211a89c2e547791fdbd619d57cd2448f9d2
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# probabilistic-marker-based-cell-assignment

## Summary

Assigns cell type labels to single-cell RNA-seq data by computing pseudo-probabilities of curated marker gene sets in a GeneVector cell embedding space, then labeling each cell by its highest-scoring phenotype. Validated against published annotations, e.g. across 18 cancers in the Tumor Immune Cell Atlas (TICA).

## When to use

Use when raw-count scRNA-seq AnnData needs automated cell type annotation from curated marker gene sets, and predicted labels should be validated against existing published cell type annotations.

## When NOT to use

- Input .X contains normalized, log-transformed, or log-normalized values instead of raw counts — the TICA benchmark (Supplementary Fig. 2C, D) shows such preprocessing severely degrades assignment accuracy
- No reliable marker gene sets exist for the cell types of interest, since assignment quality depends entirely on the curated markers supplied to phenotype_probability
- Genes have not passed entropy-based QC (entropy_threshold=1.0) or other variable-gene filtering, leaving low-information genes in the embedding

## Inputs

- Scanpy AnnData object with raw counts in .X (e.g. TICA Tumor Immune Cell Atlas h5ad)
- Curated marker gene sets as a Python dict mapping cell type names to lists of gene symbols
- Trained GeneVector embedding file (genes.vec) and GeneVectorDataset

## Outputs

- AnnData with per-cell-type pseudo-probability columns in .obs (columns containing 'Pseudo-probability')
- Assigned cell type label per cell in .obs['genevector']
- UMAP-embedded AnnData from CellEmbedding.get_adata() for visualization
- Accuracy metrics comparing predicted labels to published annotations (e.g. tica_preprocessing_benchmark.csv)

## How to apply

Load the scRNA-seq dataset as a Scanpy AnnData object with raw counts in .X and filter genes with GeneVectorDataset.quality_control(adata, entropy_threshold=1.0). Train a GeneVector model on the co-expression targets (e.g. emb_dimension=100, model.train(1000, threshold=1e-6), output_file='genes.vec'), then load gene embeddings with GeneEmbedding('genes.vec', dataset, vector='average') since averaging input and output weights is recommended. Build cell embeddings via CellEmbedding(dataset, gene_embed) and retrieve an AnnData with UMAP via get_adata(). Define a dictionary of marker genes per cell type (e.g. 'T Cell': ['CD3D','CD3G','CD3E','TRAC','IL32','CD2']) and call cell_embed.phenotype_probability(adata_embedded, markers); the 'genevector' column in obs holds the assigned label, chosen by maximum pseudo-probability. The rationale is that GeneVector's mutual-information-based latent space preserves cell-type-specific co-expression signal, so marker set membership translates into separable regions of the cell embedding; importantly, raw counts in .X are required because normalized or log-transformed inputs degrade assignment accuracy sharply.

## Related tools

- **GeneVector** (Trains mutual-information-based gene embeddings and provides CellEmbedding.phenotype_probability for marker-set-based probabilistic cell type assignment) — https://github.com/nceglia/genevector
- **Scanpy** (Hosts the raw-count AnnData input and renders UMAP visualizations of annotated cells and pseudo-probability scores) — https://github.com/nceglia/genevector
- **AnnData** (Data container carrying raw counts in .X and receiving pseudo-probability and genevector label columns in .obs)
- **PyTorch** (Backend for training the neural gene embedding (model.train(1000, threshold=1e-6)))

## Examples

```
cell_embed = CellEmbedding(dataset, GeneEmbedding("genes.vec", dataset, vector="average")); adata = cell_embed.phenotype_probability(adata_embedded, markers)
```

## Evaluation signals

- Predicted labels in .obs['genevector'] match published TICA annotations at significantly higher accuracy with raw counts than with normalized or log-normalized inputs (direction and significance of Supplementary Fig. 1-2)
- obs contains one 'Pseudo-probability' column per supplied marker set, and each cell's assigned label equals the marker set with maximal pseudo-probability
- Raw counts were present in .X before GeneVectorDataset creation, and entropy QC (entropy_threshold=1.0) was applied
- Gene embedding file genes.vec exists and GeneEmbedding was loaded with vector='average' (input and output weights averaged)
- Benchmark table (e.g. tica_preprocessing_benchmark.csv) tabulates per-condition accuracies and raw counts rank first

## Limitations

- Accuracy is highly sensitive to input preprocessing: normalized and log-normalized inputs performed very poorly on full and artificial-batch-effect TICA datasets (Supplementary Fig. 2C, D)
- Assignment quality depends on the curated marker gene sets; poor or incomplete markers will mislabel cells
- Entropy-based QC and embedding training are computationally expensive one-time costs; legacy MI computation can take ~26 hours for 2,000 genes x 5,000 cells without accelerated backends
- Artificially introduced library-size batch effects in subsetted data can degrade assignment and require batch correction verification

## Evidence

- [methods] Cell type assignment via marker pseudo-probabilities: "adata = cell_embed.phenotype_probability(adata, markers)"
- [methods] Raw count requirement: "GeneVector uses Scanpy AnnData objects with raw count data in ``.X``"
- [methods] Entropy QC threshold: "adata = GeneVectorDataset.quality_control(adata, entropy_threshold=1.0)"
- [methods] Training parameters: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [discussion] TICA validation scope: "accurate cell type assignment across 18 different cancers in 181 patients described in the Tumor Immune Cell Atlas (TICA)"
- [readme] Marker set example and annotation output: [evidence span withheld — anchor status: external_doc]
- [readme] Averaged weights recommended: [evidence span withheld — anchor status: external_doc]
- [results] Raw-count benchmark finding: [evidence span withheld — anchor status: not_found]
