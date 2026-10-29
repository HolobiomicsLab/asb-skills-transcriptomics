---
name: single-cell-senescence-signature-scoring
description: Use when scoring single-cell transcriptomes (AnnData, normalized but
  unscaled) for cellular senescence with cell-type-specific signatures, e.g., quantifying
  senescent-cell burden across ages or disease conditions.
when_to_use_negative:
- Data are scaled (e.g., scanpy sc.pp.scale applied) — score_hub requires unscaled
  values since it binarizes by default
- The goal is a plain scanpy-style gene-set score without background correction —
  then use score_hub(binarize=False, importance=False) after normalize_total/log1p,
  or scanpy.score_genes directly; the SenePy defaults would change the meaning of
  the score
- Studying senescence in bulk transcriptomes or spatial data without single-cell AnnData
  input — SenePy scoring requires a single-cell expression object
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SenePy
  role: Provides the hub signature database (load_hubs, get_genes, merge_hubs, search_hubs_by_genes)
    and the score_hub function for binarized, importance-weighted per-cell senescence
    scoring with expression-bin background correction
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container for the single-cell expression matrix and per-cell scores written
    to obs; requires unscaled data
- name: scanpy
  role: Preprocessing (normalize_total, log1p) for non-default scoring mode and cross-validation
    via scanpy.score_genes with binarize=False, importance=False
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:5e26f764e31b493945998f70fc126ac4647fa95cc8a1cd10a8590dae33fb038a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-senescence-signature-scoring

## Summary

Scores individual cells for senescence using SenePy's cell-type-specific hub signatures: load hubs with load_hubs, extract genes via get_genes, then run score_hub with binarization and hub-network importance weighting to produce per-cell senescence scores and identify senescent outlier cells.

## When to use

Use when scoring single-cell transcriptomes (AnnData, normalized but unscaled) for cellular senescence with cell-type-specific signatures, e.g., quantifying senescent-cell burden across ages or disease conditions.

## When NOT to use

- Data are scaled (e.g., scanpy sc.pp.scale applied) — score_hub requires unscaled values since it binarizes by default
- The goal is a plain scanpy-style gene-set score without background correction — then use score_hub(binarize=False, importance=False) after normalize_total/log1p, or scanpy.score_genes directly; the SenePy defaults would change the meaning of the score
- Studying senescence in bulk transcriptomes or spatial data without single-cell AnnData input — SenePy scoring requires a single-cell expression object

## Inputs

- AnnData single-cell gene expression object (.h5ad), raw or normalized/log1p but NOT scaled
- SenePy hub signature retrieved via hubs.get_genes((tissue, cell, hub_num)) or hubs.hubs[(tissue, cell, hub_num)]
- hubs.metadata dataframe for selecting tissue/cell/hub_num
- Cell metadata (donor age, disease status, sample ID) in adata.obs for group comparisons

## Outputs

- Per-cell senescence scores stored in adata.obs['sp_score']
- Senescent outlier calls (score > mu + 3 sigma) per cell
- Per-sample or per-age-group aggregated score/proportion tables (e.g., senescence_scores.csv, covid_epithelial_senescence_scores.csv, tongue_keratinocyte_senescence_outliers.csv)
- Group-comparison statistics (chi-square or two-tailed Mann-Whitney p-values)

## How to apply

Initialize the signature database with sp.load_hubs(species='Mouse' or 'Human'), which loads 75 mouse and 65 human hub signatures from 97 unique cell types, and browse hubs.metadata for tissue, cell type, and hub_num. Extract a signature with hubs.get_genes((tissue, cell, hub_num)) or merge signatures with merge_hubs (optionally with overlap_threshold=2 or calculate_thresh=True for a permutation-based threshold). Score cells with sp.score_hub(adata, signature) using the defaults: data are binarized (making normalization/transformation optional) and each gene is weighted by its number of connections in the hub network; data must NOT be scaled. The score equals weighted signature expression minus expected expression from nctrl_size background genes sampled within expression bins (Methods Eq. 11-15). Define senescent outliers as cells exceeding mu + 3 sigma of the hub score, then test group differences (e.g., chi-square across ages, two-tailed Mann-Whitney across disease samples).

## Related tools

- **SenePy** (Provides the hub signature database (load_hubs, get_genes, merge_hubs, search_hubs_by_genes) and the score_hub function for binarized, importance-weighted per-cell senescence scoring with expression-bin background correction) — https://github.com/jaleesr/senepy
- **AnnData** (Container for the single-cell expression matrix and per-cell scores written to obs; requires unscaled data)
- **scanpy** (Preprocessing (normalize_total, log1p) for non-default scoring mode and cross-validation via scanpy.score_genes with binarize=False, importance=False)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])
```

## Evaluation signals

- Score column exists in adata.obs['sp_score'] for every cell after calling sp.score_hub with default binarize=True and importance weighting
- Outlier fractions across age groups reproduce published chi-square p-values (e.g., tongue keratinocyte type-A/type-B hubs: 7.9e-86 and 7.9e-90)
- Mann-Whitney two-tailed test on per-sample epithelial scores reproduces COVID-19 decedent vs control result at ~p = 0.004
- Cross-check re-implemented scores against direct sp.score_hub output and against scanpy.score_genes (binarize=False, importance=False) mimic mode
- Marker dynamics recovered from thresholded scores match reported values: CDKN2A dynamic in ~26% human / ~32% mouse cell types; CCL4 in 18/50 human cell types; old/young positive-cell ratio distribution matches Fig. 1B

## Limitations

- Data passed to score_hub must not be scaled; with binarize=True normalization/transformation is optional, but scaling breaks the binarization-based scoring
- Universal signature lists in universal_signatures/ (FDR p-value < 0.01) carry no importance weights; weights must be obtained via merge_hubs()
- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging, and genes downregulated in senescent cells are not accounted for (no negative markers)
- High dropout and limited sequencing depth challenge use of classical markers like Cdkn2a/Cdkn1a as sole indicators; transient p16 expression combined with dropout yields low Cdkn2a detectability in single-cell data
- Signature count is finite: 72 mouse and 64 human weighted signatures (methods database states 75 mouse / 65 human from 97 cell types), so novel cell types may lack a matched hub

## Evidence

- [readme] By default, it binarizes the data, which makes normalization and transformation optional.: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] score_hub default importance weighting: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [readme] unscaled data requirement: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [methods] database scope of signatures: "Database of 75 **mouse** and 65 **human** signatures of cellular senescence from 97 unique cell types"
- [abstract] background-corrected scoring rationale: "We utilize 72 mouse and 64 human weighted single-cell transcriptomic signatures of cellular senescence to create the SenePy scoring platform."
- [results] outlier definition and age kinetics: [evidence span withheld — anchor status: not_found]
