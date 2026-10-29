---
name: batch-correction-method-benchmarking
description: Use when comparing a pretrained single-cell embedding model's batch insensitivity
  against batch-correction baselines, e.g. computing kBET acceptance rates on multi-donor
  data such as the HCA Census of Immune Cells bone-marrow set (n=282,588 cells, 64
  healthy donors).
when_to_use_negative:
- The input is already a corrected/normalized feature table rather than raw ranked
  gene-expression data — tGPT expects gene expression rankings, not expression levels
- Data are from a single batch/donor with no batch variable available, making kBET
  and batch-correction comparison meaningless (the paper notes batch information is
  not always available)
- The analysis targets differential fold-change interpretation, since tGPT's rank-based
  input neglects fold changes among genes
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: tGPT
  role: Pretrained transformer model used to extract batch-insensitive cell embeddings
    from gene expression rankings
  repo: https://github.com/deeplearningplus/tGPT
- name: Scanpy
  role: Used for neighbor graph computation and analysis (sc.pp.neighbors with n_neighbors=20);
    version 1.6.0 cited in paper
  repo: https://github.com/theislab/scanpy
- name: Pegasus
  role: Batch-correction/analysis baseline in the benchmark; version 1.4.3
  repo: https://github.com/lilab-bcb/pegasus
- name: scVI
  role: Deep-learning batch-correction baseline in the benchmark; version 0.6.8 cited
  repo: https://github.com/theislab/scvelo
- name: MNN
  role: Batch-correction baseline; version 1.8.0
  repo: https://github.com/MarioniLab/MNN2017
- name: ComBat
  role: Batch-correction baseline; version 1.8.0
  repo: https://github.com/Jfortin1/ComBatHarmonization
- name: Harmony
  role: Batch-correction baseline; version 0.1.6
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Batch-correction baseline; version 1.7.1
  repo: https://github.com/brianhie/scanorama
- name: iMAP
  role: Batch-correction baseline; version 1.0.0
  repo: https://github.com/Svvord/iMAP
- name: scArches
  role: Batch-correction baseline; version 1.7.0
  repo: https://github.com/theislab/scarches
- name: Seurat
  role: Batch-correction baseline
- name: BBKNN
  role: Batch-correction baseline
- name: DESC
  role: Batch-correction baseline
- name: kBET
  role: Metric used to compute acceptance rates quantifying batch mixing on each embedding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:4e9aeb8ba170c8b6c5e631372ad4427f38093c96118ab44a7d9b73bb44ead079
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  repo_url: https://github.com/deeplearningplus/tGPT
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
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# batch-correction-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a transformer-pretrained representation (tGPT) against eleven established batch-correction methods (ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP, DESC) using kBET acceptance rates on the HCA bone-marrow dataset, where tGPT's rank-based input design is expected to confer batch insensitivity.

## When to use

Use when comparing a pretrained single-cell embedding model's batch insensitivity against batch-correction baselines, e.g. computing kBET acceptance rates on multi-donor data such as the HCA Census of Immune Cells bone-marrow set (n=282,588 cells, 64 healthy donors).

## When NOT to use

- The input is already a corrected/normalized feature table rather than raw ranked gene-expression data — tGPT expects gene expression rankings, not expression levels
- Data are from a single batch/donor with no batch variable available, making kBET and batch-correction comparison meaningless (the paper notes batch information is not always available)
- The analysis targets differential fold-change interpretation, since tGPT's rank-based input neglects fold changes among genes

## Inputs

- HCA Census of Immune Cells bone-marrow dataset (project cc95ff89-2e68-4a08-a234-480eca21ce79, n=282,588 cells, 64 healthy donors) from data.humancellatlas.org
- Per-cell gene expression rankings file (gene symbols ranked by expression, e.g. gzip of ranked gene-symbol lines)
- Donor/batch label annotations for kBET
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (62 top-expressing genes)

## Outputs

- tGPT cell-embedding matrix (mean-pooled hidden states over non-padding positions, max_len=64)
- Batch-corrected embeddings from ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP, DESC
- kbet_acceptance_rates.csv — per-method kBET acceptance rate comparison table (tGPT ~0.87 on HCA)

## How to apply

Download the HCA Census of Immune Cells bone-marrow dataset (project cc95ff89-2e68-4a08-a234-480eca21ce79, 282,588 cells from 64 donors) from data.humancellatlas.org. Compute per-cell gene expression rankings and extract tGPT cell embeddings from the pretrained checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (GPT2Model with output_hidden_states=True, max_len=64 top-expressing genes, mean-pooled over non-padding token positions as in the tGPT README). Run the eleven baselines (ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP, DESC) on the same count matrix to obtain corrected embeddings, following the paper's versioned tools (e.g. Scanpy v1.6.0, Pegasus v1.4.3, scVI, MNN v1.8.0, ComBat v1.8.0). Compute kBET acceptance rates on tGPT embeddings and each batch-corrected embedding using donor/batch labels as the batch variable. Assemble per-method acceptance rates into kbet_acceptance_rates.csv; the rationale is that tGPT consumes expression rankings of top-expressing genes rather than actual expression levels, so it should be naturally insensitive to batch effects without explicit correction.

## Related tools

- **tGPT** (Pretrained transformer model used to extract batch-insensitive cell embeddings from gene expression rankings) — https://github.com/deeplearningplus/tGPT
- **Scanpy** (Used for neighbor graph computation and analysis (sc.pp.neighbors with n_neighbors=20); version 1.6.0 cited in paper) — https://github.com/theislab/scanpy
- **Pegasus** (Batch-correction/analysis baseline in the benchmark; version 1.4.3) — https://github.com/lilab-bcb/pegasus
- **scVI** (Deep-learning batch-correction baseline in the benchmark; version 0.6.8 cited) — https://github.com/theislab/scvelo
- **MNN** (Batch-correction baseline; version 1.8.0) — https://github.com/MarioniLab/MNN2017
- **ComBat** (Batch-correction baseline; version 1.8.0) — https://github.com/Jfortin1/ComBatHarmonization
- **Harmony** (Batch-correction baseline; version 0.1.6) — https://github.com/immunogenomics/harmony
- **Scanorama** (Batch-correction baseline; version 1.7.1) — https://github.com/brianhie/scanorama
- **iMAP** (Batch-correction baseline; version 1.0.0) — https://github.com/Svvord/iMAP
- **scArches** (Batch-correction baseline; version 1.7.0) — https://github.com/theislab/scarches
- **Seurat** (Batch-correction baseline)
- **BBKNN** (Batch-correction baseline)
- **DESC** (Batch-correction baseline)
- **kBET** (Metric used to compute acceptance rates quantifying batch mixing on each embedding)

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); model = GPT2LMHeadModel.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_hidden_states=True).transformer
```

## Evaluation signals

- tGPT's kBET acceptance rate on the HCA bone-marrow data is within tolerance of the reported ~0.87 (Figure S1L)
- tGPT's kBET acceptance rate is comparable to those of ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP and DESC
- Embedding matrix dimensions match the number of input cells, with rows mean-pooled over non-padding token positions only (attention-mask-aware pooling)
- kbet_acceptance_rates.csv contains one acceptance-rate row per method (tGPT + 11 baselines) computed with donor/batch as the batch variable
- Clustering metrics from 62 top-expressing genes (max_len=64 checkpoint) are comparable to those from 126 genes, confirming the checkpoint choice does not degrade representation quality

## Limitations

- tGPT uses only top-expressing genes and may miss information specifically represented within low-expressing genes
- tGPT uses gene expression rankings, not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; last-layer representations are not guaranteed to give better clustering
- Pretrained tGPT weights/checkpoint availability beyond the code repository link (and the Hugging Face checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64') is not explicitly stated

## Evidence

- [discussion] benchmark tool set: [evidence span withheld — anchor status: not_found]
