---
name: fibrosis-colocalization-analysis
description: Use when analyzing spatial transcriptomics (e.g., Visium) sections from
  tissue undergoing fibrosis (e.g., post-myocardial-infarction heart) to determine
  whether senescence-burden spots colocalize with fibrotic regions and are spatially
  clustered.
when_to_use_negative:
- Input is single-cell (not spatial) transcriptomics with no spot coordinates — colocalization
  and Moran's I cannot be computed; use standard SenePy single-cell scoring instead
- Data are already scaled — score_hub expects normalized-or-raw expression that is
  not scaled, so scaled AnnData must be reloaded or re-derived
- Non-mouse spatial data where no matching species hub signatures exist (use the Human
  hubs or merge_hubs-derived universal signature instead of the mouse heart hubs)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides load_hubs/get_genes to retrieve mouse heart senescence hub signatures
    and score_hub to score each Visium spot (default binarization and gene-importance
    weighting)
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Holds the Visium count matrix and per-spot scores in adata.obs; SenePy integrates
    with AnnData and scanpy
- name: scanpy
  role: AnnData/scanpy-compatible workflow for the spatial count matrices (normalize_total/log1p
    only if binarize=False is used)
- name: Python
  role: Runtime environment for SenePy (Python 3.9 conda environment recommended;
    pip install senepy)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:2164e63e73313398108ca5f20012a886dcc1c23023163b18ea5123c804343e0e
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
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fibrosis-colocalization-analysis

## Summary

Scores spatial transcriptomics spots (e.g., 10x Visium post-MI heart sections) for cellular senescence using SenePy's cell-type-specific mouse heart hub signatures, then tests whether senescence-high spots colocalize with fibrotic marker expression (Col1a1) and show spatial autocorrelation (Moran's I).

## When to use

Use when analyzing spatial transcriptomics (e.g., Visium) sections from tissue undergoing fibrosis (e.g., post-myocardial-infarction heart) to determine whether senescence-burden spots colocalize with fibrotic regions and are spatially clustered.

## When NOT to use

- Input is single-cell (not spatial) transcriptomics with no spot coordinates — colocalization and Moran's I cannot be computed; use standard SenePy single-cell scoring instead
- Data are already scaled — score_hub expects normalized-or-raw expression that is not scaled, so scaled AnnData must be reloaded or re-derived
- Non-mouse spatial data where no matching species hub signatures exist (use the Human hubs or merge_hubs-derived universal signature instead of the mouse heart hubs)

## Inputs

- 10x Visium spatial transcriptomics count matrices (post-MI mouse heart time course) loadable into an AnnData object
- SenePy mouse heart hub senescence signatures retrieved from hubs.metadata (8 heart hubs)
- Fibrosis marker gene (Col1a1) expression per spatial spot
- Spatial coordinates of spots for Moran's I autocorrelation testing

## Outputs

- Per-spot senescence hub scores and composite senescence score
- Senescence-outlier spot labels (upper tail of score distribution)
- Colocalization statistics of senescence-outlier spots with Col1a1+ fibrotic areas
- Moran's I spatial autocorrelation values per Visium sample
- Results table mi_senescence_colocalization_results.csv

## How to apply

Load the mouse hub signatures with sp.load_hubs(species='Mouse'), filter hubs.metadata to the heart tissue, and retrieve the 8 heart hub gene signatures via hubs.get_genes((tissue, cell, hub_num)). Load the 10x Visium myocardial-infarction time-course count matrices into an AnnData object compatible with scanpy, keeping data normalized-or-raw but never scaled, since sp.score_hub binarizes by default and weights each gene by its number of hub-network connections. Score every spot against each of the 8 heart hubs with sp.score_hub using defaults (binarize=True, importance weighting on), then aggregate the 8 per-spot scores into a composite senescence score and flag senescence-outlier spots from the upper tail of the score distribution across the time course. Overlay outlier spots on the spatial map of Col1a1-expressing fibrotic regions to quantify colocalization, and test spatial coherence of the composite score with Moran's I across the Visium sections (expect significant clustering in most samples, e.g., 8 of 9 post-MI samples). Write per-spot scores, outlier labels, colocalization statistics, and Moran's I values to a results table such as mi_senescence_colocalization_results.csv.

## Related tools

- **SenePy** (Provides load_hubs/get_genes to retrieve mouse heart senescence hub signatures and score_hub to score each Visium spot (default binarization and gene-importance weighting)) — https://github.com/jaleesr/senepy
- **AnnData** (Holds the Visium count matrix and per-spot scores in adata.obs; SenePy integrates with AnnData and scanpy)
- **scanpy** (AnnData/scanpy-compatible workflow for the spatial count matrices (normalize_total/log1p only if binarize=False is used))
- **Python** (Runtime environment for SenePy (Python 3.9 conda environment recommended; pip install senepy))

## Examples

```
adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Heart', 'fibroblast', 1)])
```

## Evaluation signals

- Each of the 8 mouse heart hub signatures is retrievable via hubs.get_genes((tissue, cell, hub_num)) and scores are stored per spot in adata.obs
- AnnData expression is normalized-or-raw but NOT scaled before sp.score_hub with default settings (binarization makes normalization optional)
- Senescence-outlier spots are drawn from the upper tail of the composite score distribution across the time course, not an arbitrary fixed cutoff
- Colocalization of outlier spots with Col1a1+ fibrotic regions increases by 7 days post-MI, matching the reported strong colocalization finding
- Moran's I indicates significant spatial clustering of the composite senescence score in the majority of samples (reported in 8 of 9 samples), and results are written to mi_senescence_colocalization_results.csv

## Limitations

- If binarize=False and importance=False are used to mimic scanpy.score_genes, the data must first be normalized (scanpy.pp.normalize_total) and log transformed (sc.pp.log1p); scaled data are always inappropriate
- Merged/universal signatures (e.g., from merge_hubs with overlap_threshold or calculate_thresh) carry no importance weights unless weights are taken from merge_hubs output
- SenePy signatures do not account for genes downregulated in senescent cells (no negative markers), and hub derivation discounts genes overlapping the transcriptional shift of organismal aging

## Evidence

- [readme] With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled:"
- [readme] Default gene importance weighting by hub-network connections: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [methods] Default binarization makes normalization and transformation optional: "By default, it binarizes the data, which makes normalization and transformation optional."
- [methods] Signature extraction via get_genes keyed on tissue, cell, hub_num: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [abstract] SenePy applied to myocardial infarction among disease contexts: "We use SenePy to map the kinetics of senescent cell accumulation in healthy aging as well as multiple disease contexts, including tumorigenesis, inflammation, and myocardial infarction."
