---
name: weighted-binarized-gene-set-scoring
description: Use when scoring single-cell or spatial-transcriptomics data (AnnData/scanpy)
  against SenePy senescence hub signatures, e.g. to flag senescent spots in Visium
  myocardial-infarction sections using mouse heart hub signatures.
when_to_use_negative:
- If the AnnData expression matrix is already scaled (score_hub expects unscaled raw
  or normalized data)
- If you specifically want scanpy.score_genes-like behavior on continuous normalized
  values — then pass binarize=False, importance=False rather than relying on defaults
- If using universal signatures from Universal_mouse.txt / Universal_human.txt, which
  have no importance weights — derive weights from merge_hubs() instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SenePy
  role: Provides load_hubs, get_genes and score_hub with default binarize=True and
    importance weighting by hub-network gene connections
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the expression matrix; scores are stored in .obs
- name: scanpy
  role: Compatible pipeline framework; score_hub defaults mimic binarized scoring
    so scanpy.pp.normalize_total / sc.pp.log1p are optional
- name: Python
  role: Language runtime for the open-source SenePy package (pip install senepy, Python
    3.9 environment)
  repo: https://github.com/jaleesr/senepy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:13f56f50e7aed84cf3d0b01cd58a1fa01083e1d502d8b32b239a143d96f92390
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# weighted-binarized-gene-set-scoring

## Summary

SenePy's default signature-scoring mode, which binarizes single-cell/spot expression and weights each gene by its number of connections in the senescence hub network, so normalization and log-transformation become optional. It converts cell-type-specific senescence signatures (75 mouse, 65 human hubs) into per-cell scores for AnnData objects.

## When to use

Use when scoring single-cell or spatial-transcriptomics data (AnnData/scanpy) against SenePy senescence hub signatures, e.g. to flag senescent spots in Visium myocardial-infarction sections using mouse heart hub signatures.

## When NOT to use

- If the AnnData expression matrix is already scaled (score_hub expects unscaled raw or normalized data)
- If you specifically want scanpy.score_genes-like behavior on continuous normalized values — then pass binarize=False, importance=False rather than relying on defaults
- If using universal signatures from Universal_mouse.txt / Universal_human.txt, which have no importance weights — derive weights from merge_hubs() instead

## Inputs

- AnnData single-cell or spatial (Visium) gene-expression matrix, raw or normalized but not scaled
- SenePy hub object from sp.load_hubs (75 mouse / 65 human signatures with metadata)
- Signature key tuple (tissue, cell, hub_num), e.g. ('Heart', 'fibroblast', 1)

## Outputs

- Per-cell/per-spot senescence score column written to adata.obs (e.g. adata.obs['sp_score'])
- Optional composite senescence burden and outlier labels for downstream colocalization and Moran's I analyses

## How to apply

Load the hub object with sp.load_hubs(species='Mouse') (or 'Human') and retrieve signatures via hubs.get_genes((tissue, cell, hub_num)), filtering hubs.metadata by tissue (e.g. 'Heart'). Ensure the AnnData object holds raw or normalized (optionally log1p-transformed) expression but is NOT scaled, because the default score_hub binarizes the data, making normalization and transformation optional. Call sp.score_hub(adata, hubs.hubs[(tissue, cell, hub_num)]) with defaults so each gene is weighted by its number of connections in the hub network; write the result into adata.obs. To mimic scanpy.score_genes behavior instead, pass binarize=False, importance=False after normalize_total and log1p. Composite burden across multiple heart hubs can then be aggregated per spot and thresholded from the upper tail for downstream spatial analyses (e.g. Col1a1 colocalization, Moran's I).

## Related tools

- **SenePy** (Provides load_hubs, get_genes and score_hub with default binarize=True and importance weighting by hub-network gene connections) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the expression matrix; scores are stored in .obs)
- **scanpy** (Compatible pipeline framework; score_hub defaults mimic binarized scoring so scanpy.pp.normalize_total / sc.pp.log1p are optional)
- **Python** (Language runtime for the open-source SenePy package (pip install senepy, Python 3.9 environment)) — https://github.com/jaleesr/senepy

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Scores are present in adata.obs and finite (no NaNs) for all cells/spots scored
- Input AnnData was not scaled — expression remains raw or normalized/log1p only
- Defaults preserved: score_hub called with binarize enabled and per-gene importance weighting by hub-network connection count
- Signature keys match hubs.metadata entries for the target tissue (e.g. Heart), and retrieved gene lists come from hubs.get_genes((tissue, cell, hub_num))
- Aggregate senescence burden flags a small upper-tail subset of spots whose colocalization with Col1a1 fibrotic regions and Moran's I significance (8 of 9 samples clustered post-MI) can be reproduced

## Limitations

- score_hub's defaults assume binarized expression, so downstream users expecting continuous, normalization-dependent scores must explicitly set binarize=False and importance=False
- Universal signature text files carry no importance weights; weights only come from merge_hubs()
- SenePy does not account for genes downregulated in senescent cells (no negative markers) and discounts genes overlapping the organismal-aging transcriptional shift

## Evidence

- [readme] score_hub contains multiple options that can affect the scoring method. By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network.: "score_hub contains multiple options that can affect the scoring method. By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [readme] By default, it binarizes the data, which makes normalization and transformation optional.: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] Example scoring and saving as a new column in anndata.obs. With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled: "Example scoring and saving as a new column in anndata.obs. With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [readme] We mimic default scanpy.score_genes by removing both of these parameters: "We can mimic default scanpy.score_genes by removing both of these parameters"
- [readme] However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs(): "However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs()"
- [methods] score_hub  [section=methods; evidence='Function to score single-cell transcriptomes based on a given senescence gene signature.']: "Function to score single-cell transcriptomes based on a given senescence gene signature."
