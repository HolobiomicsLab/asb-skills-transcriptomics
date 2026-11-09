---
name: immune-cell-transcriptome-preprocessing
description: Use when raw single-cell (e.g. HCA bone marrow, Tabula Muris, GSE96583)
  count matrices must be cleaned of low-quality cells, mitochondrial/ribosomal/non-coding
  genes, and restricted to HVGs before clustering, tGPT embedding, or batch correction.
when_to_use_negative:
- Input is already a filtered/normalized expression matrix or a precomputed tGPT gene-ranking
  file (e.g. Muris_gene_rankings.txt.gz) — re-filtering would distort rankings
- The downstream pipeline is Pegasus and Scanpy/scVI thresholds (min_genes 200, mito
  30%) would be misapplied instead of Pegasus's min_genes 500 / max_genes 6000 / percent_mito
  10
- The analysis is on bulk tissue transcriptomes, where the single-cell cell-level
  QC filters do not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Cell filtering (genes <200, mito >30%) and highly variable gene selection
    (max_mean 3, min_mean 0.0125) prior to embedding and clustering
  repo: https://github.com/theislab/scanpy
- name: Pegasus
  role: Alternative QC pipeline with recommended parameters min_genes 500, max_genes
    6000, percent_mito 10, robust genes percent_cells 0.05
  repo: https://github.com/lilab-bcb/pegasus
- name: scVI
  role: Downstream method whose pipeline uses the same cell filter (genes <200, mitochondrial
    counts >30%)
  repo: https://github.com/theislab/scvelo
- name: tGPT
  role: 'Consumer of the preprocessed data: tGPT takes ranked top-expressing genes
    derived after gene-level QC as input'
  repo: https://github.com/deeplearningplus/tGPT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:6a615d5df5751c5502a9d6510f4374b733019857dd009430ce569a762509b080
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  repo_url: https://github.com/theislab/scanpy
  tools:
  - BBKNN
  - ComBat
  - DESC
  - Harmony
  - MNN
  - Pegasus
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scArches
  - scVI
  - tGPT
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# immune-cell-transcriptome-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cell- and gene-level quality-control protocol for single-cell immune-cell transcriptome data as used in the tGPT paper: removing mitochondrial, ribosomal and non-protein-coding genes, filtering low-quality cells, and selecting highly variable genes before embedding or batch-effect benchmarking.

## When to use

Use when raw single-cell (e.g. HCA bone marrow, Tabula Muris, GSE96583) count matrices must be cleaned of low-quality cells, mitochondrial/ribosomal/non-coding genes, and restricted to HVGs before clustering, tGPT embedding, or batch correction.

## When NOT to use

- Input is already a filtered/normalized expression matrix or a precomputed tGPT gene-ranking file (e.g. Muris_gene_rankings.txt.gz) — re-filtering would distort rankings
- The downstream pipeline is Pegasus and Scanpy/scVI thresholds (min_genes 200, mito 30%) would be misapplied instead of Pegasus's min_genes 500 / max_genes 6000 / percent_mito 10
- The analysis is on bulk tissue transcriptomes, where the single-cell cell-level QC filters do not apply

## Inputs

- Raw single-cell gene count matrix (e.g. 10x/mtx or CSV) with per-cell metadata such as donor/batch labels
- Gene annotation enabling identification of mitochondrial, ribosomal and non-protein-coding genes
- Per-cell QC metrics (number of expressing genes, mitochondrial count fraction)

## Outputs

- Filtered count matrix (AnnData/h5ad) with low-quality cells and mitochondrial/ribosomal/non-coding genes removed
- Highly variable gene set selected with max_mean 3, min_mean 0.0125
- Cell-by-gene matrix ready for tGPT gene-ranking extraction or batch-correction baselines

## How to apply

First discard mitochondrial genes, ribosomal genes and non-protein-coding genes from the count matrix. Then, for Scanpy- and scVI-based pipelines, filter out cells with fewer than 200 expressing genes or mitochondrial counts above 30%. If the downstream pipeline is Pegasus, apply its recommended parameters instead: min_genes of 500, max_genes of 6000, and percent_mito of 10, plus robust-gene selection with percent_cells 0.05. For Scanpy pipelines, select highly variable genes with scanpy.pp.highly_variable_genes setting max_mean to 3 and min_mean to 0.0125. The choice of pipeline-specific thresholds is a decision point: the paper explicitly uses different cutoffs for Scanpy/scVI versus Pegasus, so match the cutoffs to the downstream tool.

## Related tools

- **Scanpy** (Cell filtering (genes <200, mito >30%) and highly variable gene selection (max_mean 3, min_mean 0.0125) prior to embedding and clustering) — https://github.com/theislab/scanpy
- **Pegasus** (Alternative QC pipeline with recommended parameters min_genes 500, max_genes 6000, percent_mito 10, robust genes percent_cells 0.05) — https://github.com/lilab-bcb/pegasus
- **scVI** (Downstream method whose pipeline uses the same cell filter (genes <200, mitochondrial counts >30%)) — https://github.com/theislab/scvelo
- **tGPT** (Consumer of the preprocessed data: tGPT takes ranked top-expressing genes derived after gene-level QC as input) — https://github.com/deeplearningplus/tGPT

## Examples

```
import scanpy as sc; sc.pp.filter_cells(adata, min_genes=200); adata = adata[:, adata.var['mt'] <= 0.3]; sc.pp.highly_variable_genes(adata, max_mean=3, min_mean=0.0125)
```

## Evaluation signals

- No mitochondrial, ribosomal or non-protein-coding gene symbols remain in the filtered gene set
- All retained cells have number of expressing genes >= 200 and mitochondrial counts <= 30% (Scanpy/scVI path)
- Pegasus path retains cells with 500 <= n_genes <= 6000 and percent_mito <= 10, with robust genes selected at percent_cells 0.05
- HVG selection yields genes restricted to max_mean <= 3 and min_mean >= 0.0125
- Downstream tGPT embeddings reproduce comparable kBET acceptance rates (~0.87 on HCA) and cluster metrics, indicating QC did not remove biological signal

## Limitations

- tGPT uses only the top-expressing genes and may miss information represented in low-expressing genes
- Thresholds are pipeline-specific (Scanpy/scVI vs Pegasus); applying one pipeline's cutoffs in another context changes cell retention
- HVG thresholds (max_mean 3, min_mean 0.0125) were tuned for the paper's datasets and may not transfer to other tissues or platforms

## Evidence
