---
name: batch-effect-correction-vector-arithmetic
description: Use when scRNA-seq AnnData objects with a trained GeneVector cell embedding
  (cell_embed) contain batch/treatment labels (e.g., control vs interferon-beta stimulated
  PBMCs) and batch effects must be removed while retaining stimulus-driven variation
  for downstream UMAP and silhouette benchmarking.
when_to_use_negative:
- Batches differ strongly in cell type composition — the article notes batch correction
  is 'challenging in the presence of batch-specific differences in cell type composition'
  and users must manually verify correction vectors against cell type signatures
- Input is a raw expression matrix with no trained GeneVector gene/cell embedding
  — train the embedding (task_001-style workflow) before vector arithmetic correction
- Goal is gene-level regulatory directionality (activation vs repression) — GeneVector's
  MI is agnostic to direction of statistical dependency and correction vectors cannot
  resolve this
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: GeneVector
  role: Provides GeneVectorDataset, GeneEmbedding, CellEmbedding and the batch_correct()
    vector-arithmetic operation on cell embeddings
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Consumes AnnData objects (.h5ad) and recomputes/visualizes UMAPs on corrected
    vs uncorrected embeddings
  repo: https://github.com/nceglia/genevector
- name: AnnData
  role: Container for raw counts in .X, batch labels in .obs, and corrected cell embeddings
  repo: https://github.com/nceglia/genevector
- name: PyTorch
  role: Neural embedding backend that trains the gene vectors underlying the corrected
    cell representations
  repo: https://github.com/nceglia/genevector
- name: Harmony
  role: Benchmark batch-correction baseline; shown to remove interferon-stimulated
    myeloid variation that GeneVector preserves
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:90e5d03798f1fff5e5dadfe2cb14fea488441e44632c36286845921a93825005
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - Harmony
  - NumPy
  - PyTorch
  - Scanpy
  - SciPy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# batch-effect-correction-vector-arithmetic

## Summary

Corrects batch effects in GeneVector cell embeddings by subtracting batch-average gene embeddings from a reference batch (Vcorrection = Vreference − Vbatch) and adding the resulting correction vector to each cell embedding, preserving biological variation (e.g., interferon-stimulated myeloid ISG programs) that Harmony-based alignment removes.

## When to use

Use when scRNA-seq AnnData objects with a trained GeneVector cell embedding (cell_embed) contain batch/treatment labels (e.g., control vs interferon-beta stimulated PBMCs) and batch effects must be removed while retaining stimulus-driven variation for downstream UMAP and silhouette benchmarking.

## When NOT to use

- Batches differ strongly in cell type composition — the article notes batch correction is 'challenging in the presence of batch-specific differences in cell type composition' and users must manually verify correction vectors against cell type signatures
- Input is a raw expression matrix with no trained GeneVector gene/cell embedding — train the embedding (task_001-style workflow) before vector arithmetic correction
- Goal is gene-level regulatory directionality (activation vs repression) — GeneVector's MI is agnostic to direction of statistical dependency and correction vectors cannot resolve this

## Inputs

- Scanpy AnnData object with raw counts in .X and batch/treatment labels in .obs (e.g., control vs interferon-beta stimulated PBMCs)
- GeneVector cell embedding (CellEmbedding object, cell_embed) trained on the dataset (e.g., genes.vec, emb_dimension=100)
- Optional: Harmony-corrected embedding for benchmarking comparison

## Outputs

- Batch-corrected cell embedding and recomputed Scanpy UMAP (adata_corrected)
- Per-cell-type batch silhouette coefficient summary table (ARI/kBET/cLISI/silhouette scheme)
- Three-panel UMAP comparison figure (batch_correction_umap_comparison.png)
- Decomposed correction vector transcriptional signatures (optional interpretability output)

## How to apply

Load the interferon-beta stimulated PBMC AnnData and the cell_embed produced from the trained GeneVector model, then compute the batch correction vector as Vcorrection = Vreference − Vbatch by subtracting the average gene embedding of one batch from the reference batch, and add this vector to each individual cell embedding (per-cell vector addition). Call cell_embed.batch_correct(column="sample", reference="control") per the README, then regenerate the UMAP with Scanpy on the corrected representation and in parallel on uncorrected and Harmony-corrected embeddings for benchmarking. Compute per-cell-type batch silhouette coefficients (silhouette score over batch labels within each cell type, following the paper's ARI/kBET/cLISI/silhouette benchmarking scheme) on all three representations. Verify that CD14+ Monocytes show the largest batch silhouette coefficient after correction, and score stimulation programs (ISG metagenes) per cell type to confirm interferon-stimulated myeloid variation is preserved in the GeneVector-corrected UMAP but removed by Harmony. The correction vectors can additionally be decomposed into transcriptional signatures for interpretability.

## Related tools

- **GeneVector** (Provides GeneVectorDataset, GeneEmbedding, CellEmbedding and the batch_correct() vector-arithmetic operation on cell embeddings) — https://github.com/nceglia/genevector
- **Scanpy** (Consumes AnnData objects (.h5ad) and recomputes/visualizes UMAPs on corrected vs uncorrected embeddings) — https://github.com/nceglia/genevector
- **AnnData** (Container for raw counts in .X, batch labels in .obs, and corrected cell embeddings) — https://github.com/nceglia/genevector
- **PyTorch** (Neural embedding backend that trains the gene vectors underlying the corrected cell representations) — https://github.com/nceglia/genevector
- **Harmony** (Benchmark batch-correction baseline; shown to remove interferon-stimulated myeloid variation that GeneVector preserves)

## Examples

```
from genevector.embedding import CellEmbedding; cell_embed.batch_correct(column="sample", reference="control"); adata_corrected = cell_embed.get_adata()
```

## Evaluation signals

- Batch silhouette coefficient for CD14+ Monocytes is the largest among cell types in the corrected embedding
- Interferon-stimulated (ISG) myeloid transcriptional variation remains detectable in the GeneVector-corrected UMAP
- The same variation is absent under the Harmony-corrected UMAP (benchmark contrast)
- Cell types (e.g., control vs stimulated) align across batches in the corrected UMAP while treatment-related within-type structure is retained
- Decomposition of the correction vector yields interpretable transcriptional signatures consistent with the known interferon response

## Limitations

- Batch correction is unreliable when batches differ in cell type composition; correction vectors must be manually verified against cell type signatures
- Benchmarking metrics (ARI, kBET, cLISI, silhouette score) are referenced only as Supplementary Fig. 6A–L and are not reproduced in the main text (reproducibility gap)
- GeneVector cannot distinguish activation from repression, so correction vectors cannot resolve directionality of regulatory changes between batches

## Evidence

- [abstract] Abstract: batch correction with vector arithmetic: "we show in four single cell RNA-seq datasets that GeneVector was able to capture phenotype-specific pathways, perform batch effect correction"
- [readme] README: batch_correct API call: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion: interpretable correction vectors: "GeneVector can produce an interpretable batch correction by decomposing the derived correction vectors into transcriptional signatures"
- [discussion] Discussion: batch composition caveat: "Batch correction is challenging in the presence of batch-specific differences in cell type composition"
- [discussion] Discussion: benchmarking metrics scheme: "using a series of benchmarking metrics (ARI, kBET, cLISI, and silhouette score)"
- [methods] Methods: AnnData input format: "GeneVector uses Scanpy AnnData objects with raw count data in ``.X``"
