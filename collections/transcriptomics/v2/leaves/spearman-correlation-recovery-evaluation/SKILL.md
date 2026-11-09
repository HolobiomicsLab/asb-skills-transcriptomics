---
name: spearman-correlation-recovery-evaluation
description: Use when comparing denoising performance of scRNA-seq methods against
  a ground-truth full-depth profile, especially to test whether a method that denoises
  cells independently outperforms neighbor-aggregating methods on rare cell populations.
when_to_use_negative:
- When no full-depth ground-truth profiles are available to define the correlation
  recovery target
- When the comparison is about gene network inference quality rather than denoising
  of expression profiles
- When the method under test requires aggregated neighbors and the evaluation goal
  is not rare cell-state robustness — neighbor-aggregating methods (MAGIC, KNNsmoothing2)
  are expected to blur rare states
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: Provides the zero-inflated Poisson downsampling procedure and the pretrained
    model + Denoiser used to denoise each cell independently
  repo: https://github.com/cantinilab/scPRINT
- name: MAGIC
  role: Baseline denoising method run via the Scanpy implementation with default parameters
    and the approximate solver (K=5 on reduced cell sets)
- name: KNNsmoothing2
  role: Baseline neighbor-smoothing denoising method run with default parameters and
    K=10
- name: Scanpy
  role: Hosts the MAGIC implementation used for the benchmark
- name: scdataloader
  role: Loads and preprocesses cellxgene test datasets with raw counts and gene format
    checks
  repo: https://github.com/jkobject/scDataLoader
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:3801b845a408fdcc66e7f375279656ffb5670e36c149c215a93686f100ef907f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# spearman-correlation-recovery-evaluation

## Summary

Benchmark zero-shot single-cell denoising by downsampling expression profiles (70% transcript removal via zero-inflated Poisson), denoising with competing methods, and scoring recovery as the average Spearman correlation increase to true profiles over initially expressed genes, including rare cell states.

## When to use

Use when comparing denoising performance of scRNA-seq methods against a ground-truth full-depth profile, especially to test whether a method that denoises cells independently outperforms neighbor-aggregating methods on rare cell populations.

## When NOT to use

- When no full-depth ground-truth profiles are available to define the correlation recovery target
- When the comparison is about gene network inference quality rather than denoising of expression profiles
- When the method under test requires aggregated neighbors and the evaluation goal is not rare cell-state robustness — neighbor-aggregating methods (MAGIC, KNNsmoothing2) are expected to blur rare states

## Inputs

- Raw-count single-cell RNA-seq AnnData objects (e.g. cellxgene test datasets: ciliary body, colon, retina)
- Downsampled expression profiles (70% transcripts removed, zero-inflated Poisson)
- Cell type annotations for rare population subsetting (pericytes, microfold cells, microglial cells)

## Outputs

- Per-cell Spearman correlations between denoised and true profiles over initially expressed genes
- Spearman correlation increase over downsampled baseline per method and dataset (overall and rare-population)
- denoising_benchmark_results.csv with overall and rare-population correlation increases for scPRINT, MAGIC, and KNNsmoothing2

## How to apply

Load test datasets (e.g. ciliary body, colon, retina from cellxgene) via scdataloader's Preprocessor with raw counts and gene format checks. Downsample each cell by removing 70% of transcript counts using scPRINT's zero-inflated Poisson procedure: half the drops by Poisson sampling of reads per gene, half by complete gene dropout. Denoise the downsampled data with each method — scPRINT (pretrained checkpoint, precpt_gene_emb=None, Denoiser on ~10,000 cells and the 5000 most variable genes per dataset), MAGIC via the Scanpy implementation with the approximate solver (K=5 on reduced cell sets), and KNNsmoothing2 with K=10. For each cell, compute the average Spearman correlation between the denoised and true profiles over genes initially expressed in that cell, and report the correlation increase over the downsampled baseline, both overall and restricted to rare populations (pericytes, microfold cells of epithelium of small intestine, microglial cells).

## Related tools

- **scPRINT** (Provides the zero-inflated Poisson downsampling procedure and the pretrained model + Denoiser used to denoise each cell independently) — https://github.com/cantinilab/scPRINT
- **MAGIC** (Baseline denoising method run via the Scanpy implementation with default parameters and the approximate solver (K=5 on reduced cell sets))
- **KNNsmoothing2** (Baseline neighbor-smoothing denoising method run with default parameters and K=10)
- **Scanpy** (Hosts the MAGIC implementation used for the benchmark)
- **scdataloader** (Loads and preprocesses cellxgene test datasets with raw counts and gene format checks) — https://github.com/jkobject/scDataLoader

## Examples

```
from scprint import scPrint; model = scPrint.load_from_checkpoint('scPRINT.ckpt', precpt_gene_emb=None)
```

## Evaluation signals

- scPRINT's average Spearman correlation increase to the true profiles exceeds that of MAGIC and KNNsmoothing2, particularly on rare cell populations (pericytes, microfold cells, microglial cells)
- Downsampling removes exactly 70% of transcript counts, split between Poisson-sampled per-gene drops and complete gene dropouts
- Correlations are computed only over genes initially expressed in each cell (nonzero in the true profile)
- Results table contains both overall and rare-population correlation increases for all three methods across all three datasets (ciliary body, colon, retina)

## Limitations

- Rare-cell-state advantage stems from per-cell denoising; neighbor-aggregating baselines (MAGIC, KNNsmoothing2) blur rare states, so results favor independent denoisers by design
- Benchmarking is restricted to a representative subset of 10,000 cells and the 5000 most variable genes per dataset
- Results depend on the specific downsampling scheme (zero-inflated Poisson, 70% removal) and may not transfer to other noise models

## Evidence

- [methods] We downsample an expression profile using a zero-inflated Poisson model of the data.: "We downsample an expression profile using a zero-inflated Poisson model of the data."
- [abstract] competitive zero-shot denoising: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [full_text] rare cell state advantage: [evidence span withheld — anchor status: not_found]
