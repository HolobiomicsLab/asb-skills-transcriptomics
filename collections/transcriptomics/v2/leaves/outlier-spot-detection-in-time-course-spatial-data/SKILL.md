---
name: outlier-spot-detection-in-time-course-spatial-data
description: Use when analyzing 10x Visium spatial spots from a tissue time course
  (e.g., myocardial infarction) to flag senescence-high outlier spots, test their
  colocalization with a spatial marker such as Col1a1 fibrosis, and quantify Moran's
  I spatial autocorrelation.
when_to_use_negative:
- Input is single-cell (not spatial) data — plain SenePy hub scoring per cell suffices
  without spot-level outlier detection or Moran's I
- Data are scaled (e.g., scanpy.pp.scale applied) — score_hub requires raw or normalized,
  non-scaled expression
- No spatial coordinates are available, so colocalization and Moran's I spatial autocorrelation
  cannot be computed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_3170
tools:
- name: SenePy
  role: Provides the mouse senescence hub signatures (load_hubs, get_genes) and the
    spot-scoring function score_hub with default binarize=True and importance weighting
    by hub-network connections
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the Visium spot-by-gene count matrix; per-spot scores and outlier
    labels are stored in anndata.obs
- name: scanpy
  role: Loading and preprocessing the Visium AnnData (normalize_total/log1p optional
    because binarize=True by default); spatial colocalization and Moran's I testing
    build on the scanpy-compatible object
- name: Python
  role: Language environment for the senepy/scanny pipeline; senepy is installed via
    pip or conda (python=3.9)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:8b4e3f2402c35e1218658afb30c2802aca9f8f0e4d7e4b1e6980d41c2f0730b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# outlier-spot-detection-in-time-course-spatial-data

## Summary

Detect spatially coherent senescence-outlier spots in Visium spatial transcriptomics by scoring spots against cell-type-specific SenePy senescence hubs, aggregating into a composite senescence burden, and testing colocalization with fibrosis markers and spatial autocorrelation over a post-MI time course.

## When to use

Use when analyzing 10x Visium spatial spots from a tissue time course (e.g., myocardial infarction) to flag senescence-high outlier spots, test their colocalization with a spatial marker such as Col1a1 fibrosis, and quantify Moran's I spatial autocorrelation.

## When NOT to use

- Input is single-cell (not spatial) data — plain SenePy hub scoring per cell suffices without spot-level outlier detection or Moran's I
- Data are scaled (e.g., scanpy.pp.scale applied) — score_hub requires raw or normalized, non-scaled expression
- No spatial coordinates are available, so colocalization and Moran's I spatial autocorrelation cannot be computed

## Inputs

- 10x Visium spatial transcriptomics count matrices from a myocardial-infarction time course, as scanpy-compatible AnnData (normalized or raw, not scaled)
- SenePy mouse hub signatures (hubs.metadata filtered to heart tissue, 8 heart hubs) retrieved via sp.load_hubs(species='Mouse')
- Spatial coordinates of Visium spots and a fibrosis marker channel/expression (Col1a1) for colocalization testing

## Outputs

- Per-spot senescence hub scores and a composite senescence score per spot
- Binary senescence-outlier labels for spots from the upper tail of the score distribution
- Colocalization statistics of outlier spots with Col1a1+ fibrotic regions
- Moran's I spatial autocorrelation values of the composite senescence score per tissue section
- Results table mi_senescence_colocalization_results.csv

## How to apply

Load the mouse hub signatures with sp.load_hubs(species='Mouse') and filter hubs.metadata to heart tissue to retrieve the 8 heart hub gene signatures via hubs.get_genes((tissue, cell, hub_num)). Load the 10x Visium myocardial-infarction time-course count matrices into an AnnData object compatible with scanpy, keeping expression normalized-or-raw but explicitly not scaled. Score every spot against each of the 8 heart hubs with sp.score_hub using default settings, i.e., binarization enabled (which makes normalization and transformation optional) and per-gene importance weighting by the number of hub-network connections each gene has. Aggregate the 8 per-spot hub scores into a composite senescence score and flag senescence-outlier spots from the upper tail of the score distribution across the time course. Overlay the outlier spots onto the spatial map of Col1a1-expressing fibrotic regions, quantify colocalization, and test spatial coherence of the composite score with Moran's I per Visium section; in the source study, senescent-like spots strongly colocalized with Col1a1+ fibrotic regions by 7 days post-MI and showed significant clustering in 8 of 9 samples.

## Related tools

- **SenePy** (Provides the mouse senescence hub signatures (load_hubs, get_genes) and the spot-scoring function score_hub with default binarize=True and importance weighting by hub-network connections) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the Visium spot-by-gene count matrix; per-spot scores and outlier labels are stored in anndata.obs)
- **scanpy** (Loading and preprocessing the Visium AnnData (normalize_total/log1p optional because binarize=True by default); spatial colocalization and Moran's I testing build on the scanpy-compatible object)
- **Python** (Language environment for the senepy/scanny pipeline; senepy is installed via pip or conda (python=3.9))

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); sig = hubs.get_genes(('Heart', 'fibroblast', 1)); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Heart', 'fibroblast', 1)])
```

## Evaluation signals

- Sanity-check that each of the 8 heart hub gene lists retrieved via hubs.get_genes((tissue, cell, hub_num)) matches hubs.metadata sizes for heart hubs
- Verify the AnnData expression matrix is not scaled (score_hub tolerates normalized or raw with default binarization, but fails conceptually on scaled data)
- Outlier spots should fall in the upper tail of the composite score distribution and, as a positive control, should be enriched in Col1a1+ fibrotic regions by the 7-days-post-MI time point
- Moran's I of the composite senescence score should be significant in the majority of sections (in the source study, significant spatial clustering in 8 of 9 samples)
- Per-spot scores, outlier labels, colocalization statistics, and Moran's I values must all be present as columns in mi_senescence_colocalization_results.csv

## Limitations

- Binarization and importance weighting are defaults of score_hub; removing them (binarize=False, importance=False) requires pre-normalized, log-transformed data and changes the score semantics
- SenePy signatures capture upregulated senescence programs only — downregulated genes in senescent cells are not accounted for (no negative markers), so scoring may miss senescent states defined by loss of expression
- Classical single-marker senescence readouts (Cdkn2a/Cdkn1a) have low detectability due to dropout and transient p16 expression, so spot-level scores should not be reduced to a single classical marker
- Genes overlapping the transcriptional shift of organismal aging are discounted by the hub-definition methodology, so scores may exclude age-shift genes

## Evidence

- [readme] Binarization enabled with hub-connection importance weighting: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data"
- [readme] Data must be normalized-or-raw, not scaled: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [readme] Scoring AnnData spots and storing in obs: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
- [methods] Retrieving heart hub signatures by tissue, cell, hub_num: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [abstract] Colocalization of senescent spots with Col1a1 fibrosis by 7 days post-MI (task finding, supported by abstract claim of MI application): "We use SenePy to map the kinetics of senescent cell accumulation in healthy aging as well as multiple disease contexts, including tumorigenesis, inflammation, and myocardial infarction"
