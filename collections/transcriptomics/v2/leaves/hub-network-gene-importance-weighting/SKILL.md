---
name: hub-network-gene-importance-weighting
description: Use when scoring single-cell transcriptomes for senescence with SenePy
  score_hub and you want each hub gene's contribution weighted by its hub-network
  connection count, with binarized expression making normalization optional.
when_to_use_negative:
- When you want to reproduce default scanpy.score_genes behavior — in that case use
  score_hub(binarize=False, importance=False) on normalized, log1p-transformed data
  instead of the weighted/binarized defaults
- When the AnnData expression matrix has been scaled (score_hub requires unscaled
  data even though binarization makes normalization optional)
- When using universal signatures from Universal_mouse.txt/Universal_human.txt, which
  carry no importance weights — retrieve weights via hubs.merge_hubs() if weighting
  is needed
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides score_hub, which performs binarization and hub-connection importance
    weighting to compute per-cell senescence scores
  repo: https://github.com/jaleesr/senepy
- name: scanpy
  role: Baseline comparator — score_hub with binarize=False, importance=False mimics
    default scanpy.score_genes scoring
- name: AnnData
  role: Input container for the single-cell expression matrix; scores are written
    to adata.obs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:21e18a035bf1ffee7a2ae538a7a31521e611c11655716ad2d5128a1c9e7c9d0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# hub-network-gene-importance-weighting

## Summary

SenePy's default score_hub weighting scheme: each signature gene's contribution to a cell's senescence score is scaled by its number of connections in the hub network, applied to binarized expression so raw or normalized (but unscaled) AnnData can be scored directly.

## When to use

Use when scoring single-cell transcriptomes for senescence with SenePy score_hub and you want each hub gene's contribution weighted by its hub-network connection count, with binarized expression making normalization optional.

## When NOT to use

- When you want to reproduce default scanpy.score_genes behavior — in that case use score_hub(binarize=False, importance=False) on normalized, log1p-transformed data instead of the weighted/binarized defaults
- When the AnnData expression matrix has been scaled (score_hub requires unscaled data even though binarization makes normalization optional)
- When using universal signatures from Universal_mouse.txt/Universal_human.txt, which carry no importance weights — retrieve weights via hubs.merge_hubs() if weighting is needed

## Inputs

- AnnData object with single-cell gene expression matrix (raw or normalized, but not scaled)
- SenePy hub gene signature (hubs.hubs[(tissue, cell_type, hub_num)]) from sp.load_hubs(species='Mouse' or 'Human')

## Outputs

- Per-cell senescence score vector, typically stored as adata.obs['sp_score']
- Outlier-cell (senescent) classifications derived from the score distribution
- Per-cell scores exported to senescence_scores.csv

## How to apply

Load the appropriate signature hub via sp.load_hubs (e.g. hubs.get_genes(('Lung', 'fibroblast', 1)) for tissue, cell, hub_num) and pass hubs.hubs[('Lung','fibroblast',1)] to sp.score_hub on an AnnData object. Keep the defaults: binarize=True converts the expression matrix to binary detection calls, so neither sc.pp.normalize_total nor sc.pp.log1p is required — the only requirement is that the data are not scaled — and importance=True weights each gene by its number of connections in the hub network, giving highly connected hub genes more influence. The weighted signature expression is contrasted against expression-bin-matched background genes to yield a relative per-cell score. To instead mimic default scanpy.score_genes behavior, call score_hub with binarize=False, importance=False after normalizing and log-transforming the data. Store the result in adata.obs['sp_score'] and compare outlier-cell classifications against a reference-scored dataset for validation.

## Related tools

- **SenePy** (Provides score_hub, which performs binarization and hub-connection importance weighting to compute per-cell senescence scores) — https://github.com/jaleesr/senepy
- **scanpy** (Baseline comparator — score_hub with binarize=False, importance=False mimics default scanpy.score_genes scoring)
- **AnnData** (Input container for the single-cell expression matrix; scores are written to adata.obs)

## Examples

```
adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)], binarize=True, importance=True)
```

## Evaluation signals

- Reconstructed scores match sp.score_hub output called directly on the same AnnData and hub signature
- Outlier-cell classifications from the computed scores agree with the reference-scored SenePy dataset
- With binarize=False and importance=False, the result reproduces default scanpy.score_genes scoring on normalized, log1p-transformed data
- Gene importance weights equal each gene's number of connections in the hub network as defined by hubs.merge_hubs metadata
- Input matrix is confirmed unscaled before scoring (binarization tolerates raw or normalized counts)

## Limitations

- Default binarization discards expression magnitude, so scoring depends on gene detection rather than abundance
- Universal signature gene lists ship without importance weights; weights must be derived via merge_hubs
- Score_hub assumes non-scaled data; scaled matrices will produce invalid scores

## Evidence

- [methods] importance weighting default: "it weighs the importance of each gene based on the number of connections that gene has in the hub network"
- [readme] binarization default: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] unscaled requirement: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [readme] scanpy mimic: "We can mimic default scanpy.score_genes by removing both of these parameters"
- [readme] example invocation: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
