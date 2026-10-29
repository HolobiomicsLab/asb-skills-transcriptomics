---
name: single-cell-embedding-extraction
description: Use when you need comparable cell embeddings from multiple single-cell
  foundation models (scPRINT, scGPT, Geneformer, scFoundation) for a downstream benchmark
  such as batch-effect correction scored by scIB avgBIO.
when_to_use_negative:
- Your goal is state-of-the-art batch correction itself rather than benchmarking foundation
  models — scGEN and scVI outperform the zero-shot foundation models on scIB
- Your task is gene network inference or denoising rather than producing cell-level
  embeddings for clustering/batch metrics
- Your dataset is not single-cell RNA-seq (e.g., spatial omics, which scPRINT's pipeline
  drops during preprocessing)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: Primary model whose zero-shot cell embeddings are taken from classifier output
    logits of the pretrained checkpoint
  repo: https://github.com/cantinilab/scPRINT
- name: scGPT
  role: Baseline model; embeddings extracted directly from the cell embedding token
    without fine-tuning
  repo: https://github.com/jkobject/scGPT
- name: Geneformer
  role: Baseline model; embeddings from the fine-tuned gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522
    checkpoint
  repo: https://huggingface.co/ctheodoris/Geneformer
- name: scFoundation
  role: Baseline model; embeddings computed on the 5000 most variable genes, then
    PCA-reduced from 3224 to 512 dimensions
  repo: https://github.com/biomap-research/scFoundation
- name: scIB
  role: Evaluation framework computing the avgBIO aggregate from label-based (ARI,
    NMI on KNN clusters) and label-free batch-correction metrics
- name: openproblems
  role: Benchmark setup providing the pancreas and lung datasets, preprocessing, and
    batch-wise train-test splits
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:1b2464ed4fa350369dd1f737c0b506532c024311c8251f6d4ed525d0ac0c7c5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - Geneformer
  - Python
  - openproblems
  - scFoundation
  - scGPT
  - scIB
  - scPRINT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-embedding-extraction

## Summary

Extracts zero-shot (or lightly adapted) cell embeddings from single-cell foundation models — scPRINT, scGPT, Geneformer v2, and scFoundation — and evaluates them for batch-effect correction with scIB's avgBIO aggregate on the human pancreas and lung open-problems datasets.

## When to use

Use when you need comparable cell embeddings from multiple single-cell foundation models (scPRINT, scGPT, Geneformer, scFoundation) for a downstream benchmark such as batch-effect correction scored by scIB avgBIO.

## When NOT to use

- Your goal is state-of-the-art batch correction itself rather than benchmarking foundation models — scGEN and scVI outperform the zero-shot foundation models on scIB
- Your task is gene network inference or denoising rather than producing cell-level embeddings for clustering/batch metrics
- Your dataset is not single-cell RNA-seq (e.g., spatial omics, which scPRINT's pipeline drops during preprocessing)

## Inputs

- Human pancreas and lung open-problems batch-correction datasets (.h5ad/AnnData) with batch labels and the open-problems train-test split
- Pretrained scPRINT checkpoint (e.g., from https://huggingface.co/jkobject/scPRINT)
- Pretrained scGPT checkpoint (cell embedding token extraction)
- Fine-tuned Geneformer v2 model 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522' (from https://huggingface.co/ctheodoris/Geneformer)
- Pretrained scFoundation model (cell-embedding mode per its model/README.md)

## Outputs

- Cell embedding matrices per model per dataset (512-dimensional for scFoundation after PCA; classifier-logit-based for scPRINT)
- scIB avgBIO scores per model per dataset (label-based ARI/NMI on KNN clusters plus label-free metrics)
- Collated benchmark table saved as scib_avgbio_benchmark.csv including open-problems-portal state-of-the-art results

## How to apply

Load the human pancreas and lung open-problems batch-correction datasets following the openproblems benchmark setup with the same train-test split by batch and preprocessing. Compute scPRINT embeddings zero-shot from the pretrained checkpoint (available on Hugging Face at jkobject/scPRINT) using the classifier output logits, with no fine-tuning. For baselines, extract scGPT embeddings directly from the model's cell embedding token without fine-tuning, extract Geneformer v2 embeddings using the fine-tuned 'gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522' checkpoint, and compute scFoundation embeddings on the 5000 most variable genes per dataset, reducing its 3224-dimensional output to 512 with PCA. Evaluate every embedding set with scIB, computing the avgBIO aggregate over label-based metrics (ARI and NMI on K-Nearest-Neighbor clusters) and label-free metrics, and collate per-dataset scores alongside open-problems-portal state-of-the-art results.

## Related tools

- **scPRINT** (Primary model whose zero-shot cell embeddings are taken from classifier output logits of the pretrained checkpoint) — https://github.com/cantinilab/scPRINT
- **scGPT** (Baseline model; embeddings extracted directly from the cell embedding token without fine-tuning) — https://github.com/jkobject/scGPT
- **Geneformer** (Baseline model; embeddings from the fine-tuned gf-12L-95M-i4096_MTLCellClassifier_CELLxGENE_240522 checkpoint) — https://huggingface.co/ctheodoris/Geneformer
- **scFoundation** (Baseline model; embeddings computed on the 5000 most variable genes, then PCA-reduced from 3224 to 512 dimensions) — https://github.com/biomap-research/scFoundation
- **scIB** (Evaluation framework computing the avgBIO aggregate from label-based (ARI, NMI on KNN clusters) and label-free batch-correction metrics)
- **openproblems** (Benchmark setup providing the pancreas and lung datasets, preprocessing, and batch-wise train-test splits)

## Examples

```
git clone https://github.com/jkobject/scGPT && cd scGPT/mytests/  # baseline embedding extraction runs, mirroring the scPRINT benchmark setup
```

## Evaluation signals

- The output table scib_avgbio_benchmark.csv contains an avgBIO score for every model on each dataset (pancreas and lung)
- scPRINT's avgBIO scores are competitive with the foundation-model baselines and the open-problems-portal state-of-the-art values
- scPRINT and scGPT embeddings are verified to be zero-shot (no fine-tuning on the batch-correction task), while Geneformer uses only the specified fine-tuned checkpoint
- scFoundation embeddings are 512-dimensional after PCA reduction from the raw 3224-dimensional output, computed on the 5000 most variable genes
- avgBIO decomposes into label-based (ARI and NMI on K-Nearest-Neighbor clusters) and label-free components consistent with the scIB definition

## Limitations

- scPRINT's batch-effect removal is convincing by scIB metrics but not on par with state-of-the-art methods scGEN and scVI
- Zero-shot extraction means scGPT is not fine-tuned on the batch-correction task, which may understate its performance relative to task-adapted use
- scFoundation requires dataset-specific gene subsetting (5000 most variable genes) and a PCA post-reduction, adding a preprocessing dependency other models do not share

## Evidence

- [abstract] competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [intro] scPRINT gains zero-shot abilities in many tasks: "This suggests that by learning a cell model, scPRINT gains zero-shot abilities in many tasks of cellular biology."
- [methods] scGPT runs in the fork's mytests folder: [evidence span withheld — anchor status: not_found]
- [methods] scFoundation cell embedding steps: [evidence span withheld — anchor status: not_found]
- [methods] scPRINT checkpoint on Hugging Face: "the file itself is also available on [hugging face](https://huggingface.co/jkobject/scPRINT/tree/main)"
