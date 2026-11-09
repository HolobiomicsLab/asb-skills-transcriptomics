---
name: single-cell-embedding-extraction-transformer
description: Use when raw single-cell count matrices or per-cell gene expression rankings
  must be converted into fixed-length cell embeddings for clustering, UMAP visualization,
  or batch-effect benchmarking without explicit normalization or batch correction.
when_to_use_negative:
- Input is raw counts and downstream methods require absolute expression levels or
  fold changes — tGPT uses gene expression rankings, so fold changes among genes are
  neglected and biological interpretation can be affected
- The question depends on low-expressing genes — tGPT uses only top-expressing genes
  and may miss information specifically represented within low-expressing genes
- A batch-correction baseline that operates on count matrices is required — embeddings
  from tGPT are not corrected counts and cannot substitute for ComBat/MNN/scVI-corrected
  matrices in such benchmarks
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: tGPT
  role: Transformer model whose checkpoint ('lixiangchun/transcriptome-gpt-1024-8-16-64')
    supplies the hidden-state embeddings
  repo: https://github.com/deeplearningplus/tGPT
- name: Scanpy
  role: Builds AnnData from features and performs neighbor graph (sc.pp.neighbors
    n_neighbors=20), Leiden clustering, and UMAP visualization
  repo: https://github.com/theislab/scanpy
- name: Pegasus
  role: Used downstream for diffusion pseudo-time map construction from tGPT-derived
    representations
  repo: https://github.com/lilab-bcb/pegasus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:c2f8a5e6497300b7d627824430122a3b4a5fa7e3957498a6bc8596ce8802ae69
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
  - scArches
  - scVI
  - tGPT
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# single-cell-embedding-extraction-transformer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-cell feature embeddings from the tGPT transformer by feeding ranked lists of top-expressing genes into the pretrained checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' and mean-pooling hidden states, producing batch-insensitive representations for clustering and downstream single-cell analysis.

## When to use

Use when raw single-cell count matrices or per-cell gene expression rankings must be converted into fixed-length cell embeddings for clustering, UMAP visualization, or batch-effect benchmarking without explicit normalization or batch correction.

## When NOT to use

- Input is raw counts and downstream methods require absolute expression levels or fold changes — tGPT uses gene expression rankings, so fold changes among genes are neglected and biological interpretation can be affected
- The question depends on low-expressing genes — tGPT uses only top-expressing genes and may miss information specifically represented within low-expressing genes
- A batch-correction baseline that operates on count matrices is required — embeddings from tGPT are not corrected counts and cannot substitute for ComBat/MNN/scVI-corrected matrices in such benchmarks

## Inputs

- Gzipped text file of per-cell gene symbols ranked by expression (e.g. ./data/Muris_gene_rankings.txt.gz), one cell per line with gene names sanitized via regex r'\-|\.' -> '_'
- Cell type annotation file (e.g. ./data/Muris_cell_labels.txt.gz) for coloring/validation
- Pretrained tGPT Hugging Face checkpoint: lixiangchun/transcriptome-gpt-1024-8-16-64

## Outputs

- numpy feature matrix (n_cells x hidden_dim) of mean-pooled transformer embeddings
- sc.AnnData object with embeddings in .X and celltype in .obs
- Leiden cluster assignments, UMAP coordinates, and UMAP plots colored by celltype and leiden

## How to apply

Compute per-cell gene expression rankings (descending order of top-expressing genes), format each cell as a token sequence [<s>, G1, G2, ..., <e>], and tokenize with the PreTrainedTokenizerFast from the 'lixiangchun/transcriptome-gpt-1024-8-16-64' checkpoint with max_length=64 (62 top-expressing genes; 126-gene variant also reported) and truncation/padding enabled. Load GPT2LMHeadModel.from_pretrained(checkpoint, output_hidden_states=True).transformer in eval mode and run batches (batch size 64) under torch.no_grad(). Mean-pool the last_hidden_state over non-padding token positions (item[1:eos_idx].mean(dim=0), where eos_idx = attention_mask.sum(dim=1)-1) to obtain one vector per cell. Build a sc.AnnData from the feature matrix, then compute a KNN graph (sc.pp.neighbors, n_neighbors=20), Leiden clustering (resolution 0.6), and UMAP for validation. Rank-based input (expression rankings rather than levels) is the rationale for batch-effect insensitivity.

## Related tools

- **tGPT** (Transformer model whose checkpoint ('lixiangchun/transcriptome-gpt-1024-8-16-64') supplies the hidden-state embeddings) — https://github.com/deeplearningplus/tGPT
- **Scanpy** (Builds AnnData from features and performs neighbor graph (sc.pp.neighbors n_neighbors=20), Leiden clustering, and UMAP visualization) — https://github.com/theislab/scanpy
- **Pegasus** (Used downstream for diffusion pseudo-time map construction from tGPT-derived representations) — https://github.com/lilab-bcb/pegasus

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64'); model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True).transformer; batch = tokenizer(lines, max_length=64, truncation=True, padding=True, return_tensors='pt')
```

## Evaluation signals

- UMAP colored by known celltype labels shows clusters that align with cell labels/states (clusters 'highly aligned with known cell labels and states')
- Leiden clusters at resolution 0.6 on the KNN graph (n_neighbors=20) form coherent, separable groups in UMAP
- On the HCA bone-marrow dataset (n=282,588 cells, 64 donors), kBET acceptance rate of tGPT embeddings is ~0.87, comparable to the eleven batch-correction baselines (Figure S1L)
- Clustering metrics from 62 top-expressing genes are comparable to the 126-gene variant (Figure S4), so max_len=64 output should not degrade markedly vs. the longer checkpoint
- Feature matrix shape equals number of input cells, and each embedding is a mean over non-padding token positions (1:eos_idx) rather than including pad tokens

## Limitations

- tGPT uses only the top-expressing genes and may miss information specifically represented within low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, affecting biological interpretation
- The language-modeling pretraining objective predicts gene rankings and is not directly related to cell clustering, so last-layer representations are not guaranteed to give better clustering
- Clustering is sensitive to the grid-searched n_neighbors (5-100, step 5) and resolution (0.1-2, step 0.2) parameters

## Evidence

- [intro] rank-based input rationale: [evidence span withheld — anchor status: not_found]
