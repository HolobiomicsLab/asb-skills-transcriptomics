---
name: senescence-signature-scoring-of-spatial-transcriptomics
description: Use when spatial transcriptomics spots or single-cell transcriptomes
  must be scored for senescence using cell-type-specific hub signatures (e.g., mapping
  senescence burden onto Visium sections or with fibrosis markers like Col1a1).
when_to_use_negative:
- The AnnData expression matrix is already scaled — score_hub's default binarization
  assumes raw or normalized (optionally log-transformed) data, not scaled data
- The goal is scoring with plain scanpy.score_genes behavior — you would pass binarize=False,
  importance=False after normalize_total and log1p, which changes the method rather
  than applying this skill's default weighted-binarized scoring
- You only need a universal signature without importance weights — the README warns
  universal signatures have no importance weights; use merge_hubs if weights are needed,
  or a different approach
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: SenePy
  role: Provides the senescence hub signature database (load_hubs, get_genes, merge_hubs)
    and the score_hub function with default binarization and hub-network importance
    weighting used to score each spot
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the Visium count matrix; hub scores are written into adata.obs
    per spot
- name: scanpy
  role: Compatible single-cell analysis framework for loading/preprocessing the AnnData
    object (normalize_total, log1p) if non-binarized scoring is desired
- name: Python 3.9
  role: Runtime environment for the SenePy package (conda create -n senepy python=3.9;
    pip install senepy)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:5ec45994c37a71c5c94898ef38cf6acc22eb08a7addd2a0ff5f12816675e0da5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - Python 3.9
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# senescence-signature-scoring-of-spatial-transcriptomics

## Summary

Scores spatial transcriptomics spots (e.g., 10x Visium) for cellular senescence using SenePy's cell-type-specific weighted senescence hub signatures, then aggregates per-hub scores into a composite senescence burden for downstream spatial colocalization and autocorrelation analyses.

## When to use

Use when spatial transcriptomics spots or single-cell transcriptomes must be scored for senescence using cell-type-specific hub signatures (e.g., mapping senescence burden onto Visium sections or with fibrosis markers like Col1a1).

## When NOT to use

- The AnnData expression matrix is already scaled — score_hub's default binarization assumes raw or normalized (optionally log-transformed) data, not scaled data
- The goal is scoring with plain scanpy.score_genes behavior — you would pass binarize=False, importance=False after normalize_total and log1p, which changes the method rather than applying this skill's default weighted-binarized scoring
- You only need a universal signature without importance weights — the README warns universal signatures have no importance weights; use merge_hubs if weights are needed, or a different approach

## Inputs

- Single-cell or spatial (10x Visium spot-level) gene expression matrix as an AnnData object (raw or normalized, not scaled)
- SenePy hub signatures and metadata (e.g., 8 mouse heart hub signatures retrieved via load_hubs(species='Mouse'))
- Marker gene expression for spatial overlay (e.g., Col1a1 for fibrotic regions)
- Spatial coordinates of spots/tissue sections for autocorrelation testing

## Outputs

- Per-spot/per-cell senescence scores for each hub signature (adata.obs columns)
- Composite senescence burden score and senescence-outlier spot labels
- Colocalization statistics of senescence-outlier spots with marker regions (e.g., Col1a1+ fibrosis)
- Spatial autocorrelation statistics (Moran's I) per tissue section
- Results table (e.g., mi_senescence_colocalization_results.csv)

## How to apply

Load mouse or human hub signatures with senepy's load_hubs(species='Mouse'), filter hubs.metadata to the relevant tissue (e.g., heart), and extract each hub's gene list via hubs.get_genes((tissue, cell, hub_num)). Load the count matrix into an AnnData object compatible with scanpy, keeping expression raw or normalized but NOT scaled. Score every spot against each hub with sp.score_hub using its default settings: binarization enabled (making normalization/transformation optional) and per-gene importance weighting by the number of hub-network connections each gene has; store each score in adata.obs. Aggregate the per-hub scores into a composite senescence score, flag outlier spots from the upper tail of the score distribution, then test spatial coherence (e.g., Moran's I) and colocalization with marker regions such as Col1a1-positive fibrosis. Rationale: SenePy signatures better recapitulate in vivo senescence than in vitro-derived signatures, and default weighting exploits hub-network topology rather than treating all genes equally.

## Related tools

- **SenePy** (Provides the senescence hub signature database (load_hubs, get_genes, merge_hubs) and the score_hub function with default binarization and hub-network importance weighting used to score each spot) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the Visium count matrix; hub scores are written into adata.obs per spot)
- **scanpy** (Compatible single-cell analysis framework for loading/preprocessing the AnnData object (normalize_total, log1p) if non-binarized scoring is desired)
- **Python 3.9** (Runtime environment for the SenePy package (conda create -n senepy python=3.9; pip install senepy))

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); sig = hubs.get_genes(('Heart', 'fibroblast', 1)); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Heart', 'fibroblast', 1)])
```

## Evaluation signals

- Scores are computed with default score_hub settings: binarize=True and importance=True (per-gene weighting by hub-network connection count) — verify no non-default flags were flipped unintentionally
- The input AnnData is not scaled (raw or normalized is acceptable by default)
- Every spot receives a score for each of the selected hubs (e.g., all 8 mouse heart hubs), stored as columns in adata.obs
- Senescence-outlier spots come from the upper tail of the composite score distribution across the time course and their colocalization with Col1a1+ fibrotic regions is quantified, with Moran's I computed per sample (expected significance in ~8 of 9 MI samples per the source finding)
- Output CSV (mi_senescence_colocalization_results.csv) contains per-spot scores, outlier labels, colocalization statistics, and Moran's I values

## Limitations

- Universal signature lists from universal_signatures/ carry no importance weights; weights must come from merge_hubs if needed
- The overlap_threshold in merge_hubs drastically reduces gene numbers when merging only a handful of signatures
- Classical CS markers like Cdkn2a and Cdkn1a are unreliable alone due to high dropout and transient p16 expression in single-cell data — hub signatures are meant to address, not replace with single markers
- SenePy's methodology discounts CS genes overlapping the organismal-aging transcriptional shift and does not account for downregulated genes (no negative markers)

## Evidence

- [readme] score_hub default binarization and importance weighting: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data, which makes normalization and transformation"
- [readme] score_hub usage with AnnData and non-scaled data: [evidence span withheld — anchor status: not_found]
- [methods] get_genes extraction: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [abstract] SenePy MI application and signature count: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [readme] Universal signature weighting caveat: "However, there are no importance weights associated with them. If you want weights, get them from .merge_hubs():"
