---
name: kbet-acceptance-rate-computation
description: Use when comparing how well cell embeddings (e.g., tGPT mean-pooled hidden
  states) mix cells across donor/batch labels against batch-corrected baselines, or
  when verifying that a representation is insensitive to batch effect on a multi-donor
  dataset.
when_to_use_negative:
- The dataset has no batch or donor labels, so kBET cannot define a batch variable
  to test against
- You need biological cell-type interpretation rather than batch-mixing assessment
  — kBET acceptance rewards mixing and can be misleading when cell-type composition
  genuinely differs across batches
- Input is already an expression matrix without per-cell gene rankings and you cannot
  extract tGPT embeddings (the comparison requires embeddings from each method on
  the same cells)
tools:
- name: tGPT
  role: 'Source of the embedding being benchmarked: autoregressive transformer over
    gene-expression rankings; embeddings extracted from last_hidden_state with mean
    pooling over non-padding tokens'
  repo: github.com/deeplearningplus/tGPT
- name: kBET
  role: Metric computing batch-effect acceptance rates on each embedding using donor
    labels
- name: Scanpy
  role: Downstream embedding evaluation (neighbor graph sc.pp.neighbors(adata, n_neighbors=20),
    Leiden, UMAP) after feature extraction
  repo: github.com/theislab/scanpy
- name: Harmony
  role: Batch-correction baseline producing corrected embeddings for the kBET comparison
  repo: github.com/immunogenomics/harmony
- name: Scanorama
  role: Batch-correction baseline producing corrected embeddings for the kBET comparison
  repo: github.com/brianhie/scanorama
- name: iMAP
  role: Batch-correction baseline producing corrected embeddings for the kBET comparison
  repo: github.com/Svvord/iMAP
- name: scArches
  role: Batch-correction baseline producing corrected embeddings for the kBET comparison
  repo: github.com/theislab/scarches
- name: Pegasus
  role: Baseline pipeline (v1.4.3) used in the benchmarking comparison
  repo: github.com/lilab-bcb/pegasus
- name: scVI
  role: Deep-learning batch-correction baseline (v0.13.0) producing corrected embeddings
  repo: github.com/theislab/scvelo
- name: MNN
  role: Batch-correction baseline (v1.8.0) producing corrected embeddings
  repo: github.com/MarioniLab/MNN2017
- name: ComBat
  role: Batch-correction baseline (v1.8.0) producing corrected embeddings
  repo: github.com/Jfortin1/ComBatHarmonization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:84945dbe623217e1a6b0a331ea9b95029680ac1ba131c47021d3abcd5437d192
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
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
  - kBET
  - scArches
  - scVI
  - tGPT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# kbet-acceptance-rate-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies batch mixing by computing kBET acceptance rates on cell embeddings, as done to show tGPT's rank-based embeddings reach a kBET acceptance rate of ~0.87 on the HCA bone-marrow dataset, comparable to eleven batch-correction methods (ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP, DESC).

## When to use

Use when comparing how well cell embeddings (e.g., tGPT mean-pooled hidden states) mix cells across donor/batch labels against batch-corrected baselines, or when verifying that a representation is insensitive to batch effect on a multi-donor dataset.

## When NOT to use

- The dataset has no batch or donor labels, so kBET cannot define a batch variable to test against
- You need biological cell-type interpretation rather than batch-mixing assessment — kBET acceptance rewards mixing and can be misleading when cell-type composition genuinely differs across batches
- Input is already an expression matrix without per-cell gene rankings and you cannot extract tGPT embeddings (the comparison requires embeddings from each method on the same cells)

## Inputs

- Gene-expression rankings per cell (top 64 expressing genes, gene symbols ranked by expression, e.g. Muris_gene_rankings.txt.gz-style gzipped text)
- HCA Census of Immune Cells bone-marrow count matrix (project cc95ff89-2e68-4a08-a234-480eca21ce79, n=282,588 cells from 64 healthy donors)
- Donor/batch label vector per cell (batch variable for kBET)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'

## Outputs

- Per-method kBET acceptance rates for tGPT and the 11 batch-correction baselines
- kbet_acceptance_rates.csv comparison table

## How to apply

Obtain per-cell embeddings for each method being compared: for tGPT, rank top-expressing genes per cell (max_len=64), tokenize with 'lixiangchun/transcriptome-gpt-1024-8-16-64', run the GPT2 transformer with output_hidden_states=True, and mean-pool last_hidden_state over non-padding positions (item[1:eos_idx]). Run the eleven correction baselines (ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP, DESC) on the same HCA count matrix (282,588 bone-marrow cells, 64 healthy donors) to produce corrected embeddings. For each embedding, compute kBET using donor/batch labels as the batch variable, yielding a per-method acceptance rate; assemble these into kbet_acceptance_rates.csv. The rationale is that tGPT's rank-based input (expression rankings, not levels) should make embeddings insensitive to batch, so its acceptance rate should fall within tolerance of the reported ~0.87 and be comparable to dedicated correction tools.

## Related tools

- **tGPT** (Source of the embedding being benchmarked: autoregressive transformer over gene-expression rankings; embeddings extracted from last_hidden_state with mean pooling over non-padding tokens) — github.com/deeplearningplus/tGPT
- **kBET** (Metric computing batch-effect acceptance rates on each embedding using donor labels)
- **Scanpy** (Downstream embedding evaluation (neighbor graph sc.pp.neighbors(adata, n_neighbors=20), Leiden, UMAP) after feature extraction) — github.com/theislab/scanpy
- **Harmony** (Batch-correction baseline producing corrected embeddings for the kBET comparison) — github.com/immunogenomics/harmony
- **Scanorama** (Batch-correction baseline producing corrected embeddings for the kBET comparison) — github.com/brianhie/scanorama
- **iMAP** (Batch-correction baseline producing corrected embeddings for the kBET comparison) — github.com/Svvord/iMAP
- **scArches** (Batch-correction baseline producing corrected embeddings for the kBET comparison) — github.com/theislab/scarches
- **Pegasus** (Baseline pipeline (v1.4.3) used in the benchmarking comparison) — github.com/lilab-bcb/pegasus
- **scVI** (Deep-learning batch-correction baseline (v0.13.0) producing corrected embeddings) — github.com/theislab/scvelo
- **MNN** (Batch-correction baseline (v1.8.0) producing corrected embeddings) — github.com/MarioniLab/MNN2017
- **ComBat** (Batch-correction baseline (v1.8.0) producing corrected embeddings) — github.com/Jfortin1/ComBatHarmonization

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); model = GPT2LMHeadModel.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_hidden_states=True).transformer; x = model(**batch); emb = x.last_hidden_state
```

## Evaluation signals

- tGPT kBET acceptance rate on the HCA dataset falls within tolerance of the reported ~0.87 (Figure S1L)
- tGPT acceptance rate is comparable to (within reasonable range of) the 11 batch-correction baselines on the same cells and batch labels
- The same count matrix and donor/batch labels are used for tGPT embeddings and every baseline's corrected embeddings
- Output table kbet_acceptance_rates.csv contains one acceptance-rate row per method covering all 12 approaches
- tGPT embeddings are mean-pooled over non-padding positions only (item[1:eos_idx]), verified against attention_mask sums

## Limitations

- tGPT uses only top-expressing genes, so it may miss information represented in low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels; fold changes among genes are neglected, which can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues such as clustering
- Pretrained tGPT checkpoint availability is documented only via the code repository; Figure S1L is supplementary and not reproduced in the main text

## Evidence

- [readme] Pretrained model with sequence of 62 top expressing genes.
checkpoint = "lixiangchun/transcriptome-gpt-1024-8-16-64": "checkpoint = "lixiangchun/transcriptome-gpt-1024-8-16-64""
