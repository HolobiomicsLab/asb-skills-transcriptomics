---
name: single-cell-rna-denoising-benchmarking
description: Use when evaluating how well a denoising method recovers true expression
  profiles from severely downsampled scRNA-seq data (70% transcript removal), particularly
  for rare cell states like pericytes, microfold cells, or microglial cells.
when_to_use_negative:
- Input is already full-depth data with no downsampled baseline — without 70% transcript
  removal there is no controlled correlation-increase target to compute.
- Goal is gene network (GRN) inference from single-cell data rather than benchmarking
  expression-profile denoising.
- Data are spatial omics or non-primary/non-human/non-mouse cells — the scPRINT preprocessing
  scope keeps only primary cells from humans or mice and drops spatial omics datasets.
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: scPRINT
  role: pretrained large cell model whose Denoiser recovers downsampled profiles zero-shot;
    loaded via scPrint.load_from_checkpoint with precpt_gene_emb=None
  repo: https://github.com/cantinilab/scPRINT
- name: MAGIC
  role: baseline denoiser run through its Scanpy implementation with default parameters
    and the approximate solver (K=5 on the reduced cell sets)
- name: KNNsmoothing2
  role: baseline denoiser run with default parameters and K=10
- name: scdataloader
  role: Preprocessor that loads and standardizes the cellxgene test datasets (raw
    counts, gene format checks) before downsampling
  repo: https://github.com/jkobject/scDataLoader
- name: Scanpy
  role: provides the MAGIC implementation and Spearman-correlation-friendly expression
    object handling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:f7777ac2b059f46ce5277938d0902bb39f29d66eb6ddd90b177fdbc80c47bb70
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
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-rna-denoising-benchmarking

## Summary

Reproduces scPRINT's zero-shot denoising benchmark: downsample scRNA-seq test profiles (70% transcript removal via zero-inflated Poisson), denoise with pretrained scPRINT, MAGIC (Scanpy), and KNNsmoothing2, and rank methods by Spearman correlation increase to true profiles, especially on rare cell states.

## When to use

Use when evaluating how well a denoising method recovers true expression profiles from severely downsampled scRNA-seq data (70% transcript removal), particularly for rare cell states like pericytes, microfold cells, or microglial cells.

## When NOT to use

- Input is already full-depth data with no downsampled baseline — without 70% transcript removal there is no controlled correlation-increase target to compute.
- Goal is gene network (GRN) inference from single-cell data rather than benchmarking expression-profile denoising.
- Data are spatial omics or non-primary/non-human/non-mouse cells — the scPRINT preprocessing scope keeps only primary cells from humans or mice and drops spatial omics datasets.

## Inputs

- ciliary body, colon, and retina test datasets from the cellxgene database (raw count AnnData/h5ad)
- pretrained scPRINT model checkpoint (e.g. from https://huggingface.co/jkobject/scPRINT)
- cell type annotations identifying rare populations (pericytes, microfold cells of epithelium of small intestine, microglial cells)
- list of genes initially expressed per cell (correlation evaluation mask)

## Outputs

- denoised expression matrices from scPRINT, MAGIC, and KNNsmoothing2 per dataset
- average Spearman correlation increase over the downsampled baseline, overall and per rare population
- denoising_benchmark_results.csv with overall and rare-population Spearman correlation increases for the three methods

## How to apply

Load the ciliary body, colon, and retina test datasets from cellxgene and preprocess them with the scdataloader Preprocessor (raw counts, gene format checks). Remove 70% of transcript counts per cell using scPRINT's zero-inflated Poisson downsampling — half the drops by Poisson sampling of reads per gene, half by complete gene dropout — to create the downsampled baseline. Denoise the downsampled profiles with the pretrained scPRINT model loaded via scPrint.load_from_checkpoint with precpt_gene_emb=None, running the Denoiser on a representative subset of 10,000 cells and the 5000 most variable genes per dataset. Run MAGIC through its Scanpy implementation with default parameters and the approximate solver (K=5 for the reduced cell sets) and KNNsmoothing2 with default parameters and K=10. For each method and dataset, compute the average Spearman correlation between denoised and true profiles over the genes initially expressed in each cell, and derive the correlation increase over the downsampled baseline; repeat restricted to rare populations (pericytes, microfold cells, microglial cells). The rationale for scPRINT's edge is that it denoises each cell independently without aggregating profiles over neighboring cells, unlike MAGIC and KNNsmoothing2, which smooth rare cell states away.

## Related tools

- **scPRINT** (pretrained large cell model whose Denoiser recovers downsampled profiles zero-shot; loaded via scPrint.load_from_checkpoint with precpt_gene_emb=None) — https://github.com/cantinilab/scPRINT
- **MAGIC** (baseline denoiser run through its Scanpy implementation with default parameters and the approximate solver (K=5 on the reduced cell sets))
- **KNNsmoothing2** (baseline denoiser run with default parameters and K=10)
- **scdataloader** (Preprocessor that loads and standardizes the cellxgene test datasets (raw counts, gene format checks) before downsampling) — https://github.com/jkobject/scDataLoader
- **Scanpy** (provides the MAGIC implementation and Spearman-correlation-friendly expression object handling)

## Examples

```
from scprint import scPrint; model = scPrint.load_from_checkpoint('scPRINT.ckpt', precpt_gene_emb=None)
```

## Evaluation signals

- scPRINT's average Spearman correlation increase to true profiles exceeds that of MAGIC and KNNsmoothing2 on each dataset.
- On rare cell populations (pericytes, microfold cells, microglial cells), scPRINT's correlation increase remains highest while MAGIC and KNNsmoothing2 degrade due to neighbor aggregation.
- A downsampled baseline correlation is established after removing exactly 70% of transcript counts (half Poisson per-gene drops, half complete gene dropouts) before any denoising.
- Denoising is restricted to the 5000 most variable genes per dataset on a representative subset of 10,000 cells, matching the benchmark configuration.
- denoising_benchmark_results.csv contains paired overall and rare-population Spearman correlation increases for all three methods across all three datasets.

## Limitations

- The scPRINT Denoiser is run only on a representative subset of 10,000 cells and the 5000 most variable genes per dataset, not the full datasets.
- MAGIC and KNNsmoothing2 aggregate profiles over neighboring cells, which systematically penalizes them on rare cell states; results are specific to that benchmark framing.
- Results are zero-shot and derived only from the ciliary body, colon, and retina cellxgene test datasets with 70% downsampling; other tissues or dropout rates may behave differently.

## Evidence

- [methods] zero-inflated Poisson downsampling procedure: "We downsample an expression profile using a zero-inflated Poisson model of the data."
- [methods] multi-task pretraining includes denoising: "The three tasks of the multi-task pretraining are the denoising task, the classification task, and the bottleneck learning task."
- [abstract] zero-shot denoising claim: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [abstract] scPRINT pretraining scope on cellxgene: "we present scPRINT, a large cell model for the inference of gene networks pre-trained on more than 50 million cells from the cellxgene database"
