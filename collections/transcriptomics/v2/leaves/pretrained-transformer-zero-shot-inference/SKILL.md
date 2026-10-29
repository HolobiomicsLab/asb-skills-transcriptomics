---
name: pretrained-transformer-zero-shot-inference
description: Use when a raw scRNA-seq count matrix needs denoising, label prediction,
  embedding/batch correction, or gene network extraction without labeled training
  data, e.g. rare cell states where smoothing-based methods lose signal.
when_to_use_negative:
- Input is spatial omics data or cells with fewer than 200 expressed genes — scPRINT's
  pretraining filters explicitly drop these
- The task requires benchmarking against neighbor-aggregating methods where shared
  biological signal across abundant cell types is the target (MAGIC/KNNsmoothing2
  with K=5–10 can match on bulk populations)
- No pretrained checkpoint covering the organism/gene vocabulary is available (scPRINT
  pretraining keeps only primary human or mouse cells)
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: Pretrained transformer cell model applied zero-shot for denoising, classification,
    and gene network inference
  repo: https://github.com/cantinilab/scPRINT
- name: scdataloader
  role: Preprocessing and cell sampling from the cellxgene database before inference
  repo: https://github.com/jkobject/scDataLoader
- name: MAGIC
  role: Baseline denoiser (Scanpy implementation, approximate solver, K=5) used for
    comparison
- name: KNNsmoothing2
  role: Baseline denoiser (default parameters, K=10) used for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:0d18949a4bb57cf06853c7939e73a30d1a7c5d114a83ab47aaccc1ca9b1717cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - KNNsmoothing2
  - MAGIC
  - Python
  - Scanpy
  - scPRINT
  - scdataloader
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pretrained-transformer-zero-shot-inference

## Summary

Apply a large pretrained cell model (scPRINT, pre-trained on >50 million cellxgene cells) directly to single-cell RNA tasks — denoising, batch effect correction, cell label prediction, gene network inference — without task-specific fine-tuning. Because scPRINT denoises each cell independently, it is particularly strong on rare cell states where neighbor-aggregating methods fail.

## When to use

Use when a raw scRNA-seq count matrix needs denoising, label prediction, embedding/batch correction, or gene network extraction without labeled training data, e.g. rare cell states where smoothing-based methods lose signal.

## When NOT to use

- Input is spatial omics data or cells with fewer than 200 expressed genes — scPRINT's pretraining filters explicitly drop these
- The task requires benchmarking against neighbor-aggregating methods where shared biological signal across abundant cell types is the target (MAGIC/KNNsmoothing2 with K=5–10 can match on bulk populations)
- No pretrained checkpoint covering the organism/gene vocabulary is available (scPRINT pretraining keeps only primary human or mouse cells)

## Inputs

- Raw scRNA-seq count matrices (e.g. h5ad from cellxgene test datasets such as ciliary body, colon, retina)
- Pretrained scPRINT model checkpoint (Hugging Face: huggingface.co/jkobject/scPRINT)
- List of highly variable genes per dataset (e.g. top 5000)

## Outputs

- Denoised per-cell expression profiles (no neighbor aggregation)
- Spearman correlation increases over the downsampled baseline (overall and per rare cell type)
- Gene networks extracted from averaged attention matrices
- Zero-shot cell labels and embeddings with batch effect correction

## How to apply

Load the pretrained scPRINT transformer via scPrint.load_from_checkpoint with precpt_gene_emb=None (checkpoints are hosted on Hugging Face at jkobject/scPRINT). Preprocess inputs with the scdataloader Preprocessor on raw counts, checking gene format compatibility, and drop cells with <200 expressed genes. For zero-shot denoising, first degrade profiles by removing 70% of transcript counts with scPRINT's zero-inflated Poisson downsampling (half by Poisson-sampled per-gene read drops, half by complete gene dropouts), then run the Denoiser on a representative subset of ~10,000 cells over the 5,000 most variable genes per dataset. For gene networks, average Keys/Queries attention matrices over the set of cells passed to the model and select heads with a RidgeClassifier (L2 penalty 1, positivity constraint) on ground-truth edges. Evaluate denoising as the average Spearman correlation between denoised and true profiles over genes initially expressed in each cell, expressed as the correlation increase over the downsampled baseline.

## Related tools

- **scPRINT** (Pretrained transformer cell model applied zero-shot for denoising, classification, and gene network inference) — https://github.com/cantinilab/scPRINT
- **scdataloader** (Preprocessing and cell sampling from the cellxgene database before inference) — https://github.com/jkobject/scDataLoader
- **MAGIC** (Baseline denoiser (Scanpy implementation, approximate solver, K=5) used for comparison)
- **KNNsmoothing2** (Baseline denoiser (default parameters, K=10) used for comparison)

## Examples

```
from scprint import scPrint; model = scPrint.load_from_checkpoint('jkobject/scPRINT', precpt_gene_emb=None); denoised = Denoiser(model, bulk=False)(downsampled_adata)
```

## Evaluation signals

- Average Spearman correlation increase to true profiles (over genes initially expressed in each cell) exceeds that of MAGIC and KNNsmoothing2
- On rare cell subsets (pericytes, microfold cells of epithelium of small intestine, microglial cells) scPRINT's correlation increase outperforms neighbor-aggregating baselines
- Denoising run uses a representative 10,000-cell subset and the 5000 most variable genes per dataset
- Downsampling baseline removes 70% of transcripts (half Poisson-sampled reads per gene, half full gene dropouts) before denoising
- Checkpoint is loaded with precpt_gene_emb=None (no task-specific gene embeddings supplied)

## Limitations

- Zero-shot performance relies on pretraining distribution: only primary human/mouse cells were kept, spatial omics excluded, datasets with <100 cells, <10,000 genes, or >95% removed cells were dropped
- Denoising is evaluated against a 70% downsampled baseline, not fully experimental ground truth; no perfect ground truths exist for these tasks
- Transformer ability to generate graphs and their explainability remain unresolved open questions

## Evidence

- [abstract] competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [intro] by learning a cell model, scPRINT gains zero-shot abilities in many tasks of cellular biology: "This suggests that by learning a cell model, scPRINT gains zero-shot abilities in many tasks of cellular biology."
- [methods] We downsample an expression profile using a zero-inflated Poisson model of the data.: "We downsample an expression profile using a zero-inflated Poisson model of the data."
- [results] Because scPRINT denoises each cell independently without aggregating profiles over neighboring cells, it outperforms MAGIC and KNNsmoothing2 on rare cell states: [evidence span withheld — anchor status: not_found]
- [readme] scPRINT: Large Cell Model for scRNAseq data: "# scPRINT: Large Cell Model for scRNAseq data"
