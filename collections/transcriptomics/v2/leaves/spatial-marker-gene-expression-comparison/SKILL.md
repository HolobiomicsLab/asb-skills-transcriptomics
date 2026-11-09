---
name: spatial-marker-gene-expression-comparison
description: Use when STEM (or another SC-ST mapper) has assigned cells/labels to
  ST spots and you need to confirm localization of a rare cell type by testing marker-gene
  expression differences between localized and non-localized spots.
when_to_use_negative:
- When no spatial localization of the cell type has been established yet — the comparison
  presupposes a prior SC-ST mapping that defines 'localized' vs. 'other' spots
- When the marker genes are not detectably expressed in the ST data (high dropout
  may preclude a meaningful spot-level comparison)
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Provides the SC-ST mapping and pseudo-spatial coordinates that define which
    spots are considered localized for the cell type being tested
  repo: https://github.com/WhirlFirst/STEM
- name: scanpy
  role: Used to normalize (sc.pp.normalize_total), log-transform (sc.pp.log1p) and
    QC (sc.pp.calculate_qc_metrics) the ST expression data prior to the marker comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:b519f858cac7eed9a0129910bdad7308e20cdeee4ff685e093097be57494fe8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - STEM
  - scSTEM
  - scanpy
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spatial-marker-gene-expression-comparison

## Summary

Compares expression of marker genes between spatially localized and other spots in an ST dataset after SC-ST mapping, to validate whether a rare cell type (e.g. plasmacytoid dendritic cells) is genuinely enriched at its predicted spatial location.

## When to use

Use when STEM (or another SC-ST mapper) has assigned cells/labels to ST spots and you need to confirm localization of a rare cell type by testing marker-gene expression differences between localized and non-localized spots.

## When NOT to use

- When no spatial localization of the cell type has been established yet — the comparison presupposes a prior SC-ST mapping that defines 'localized' vs. 'other' spots
- When the marker genes are not detectably expressed in the ST data (high dropout may preclude a meaningful spot-level comparison)

## Inputs

- ST expression matrix (normalized/log-transformed, e.g. scanpy AnnData) with spot-level coordinates
- Cell-type labels or localization calls indicating which spots contain the cell type of interest (e.g. pDC-localized spots from the STEM mapping)
- List of marker genes for the cell type (e.g. BST2, NRP1, JCHAIN for pDCs)

## Outputs

- Marker-gene comparison table (expression of each marker in localized vs. other spots with significance, P < 0.01)
- CSV of the per-spot/per-gene comparison values

## How to apply

After mapping single cells to ST spots (e.g. via STEM's netsc2st matrix and all_coord to obtain pseudo-spatial coordinates), identify the spots where the cell type of interest is localized (e.g. pDC-localized spots) and the remaining spots. For marker genes of that cell type — in the hSCC analysis these were the pDC/immune markers BST2, NRP1, and JCHAIN — compute expression per spot and compare the two spot groups statistically, requiring significance at P < 0.01. Save the marker-gene comparison table (e.g. as CSV) as the quantitative evidence that the mapped localization is consistent with the marker-expression signature. This step validates that the SC-ST mapping recovered biologically meaningful spatial niches rather than arbitrary assignments.

## Related tools

- **STEM** (Provides the SC-ST mapping and pseudo-spatial coordinates that define which spots are considered localized for the cell type being tested) — https://github.com/WhirlFirst/STEM
- **scanpy** (Used to normalize (sc.pp.normalize_total), log-transform (sc.pp.log1p) and QC (sc.pp.calculate_qc_metrics) the ST expression data prior to the marker comparison)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(stadata); sc.pp.log1p(stadata); # then compare BST2/NRP1/JCHAIN expression between pDC-localized and other spots at P < 0.01
```

## Evaluation signals

- Marker genes (e.g. BST2, NRP1, JCHAIN) show significantly higher expression in localized spots than other spots with P < 0.01
- Comparison table contains one row per marker gene with a test statistic and P-value for the localized-vs-other contrast
- The spot sets being compared partition the ST spots consistently with the STEM-derived localization (no spot double-counted or omitted)
- Reproducing the hSCC donor analysis should reproduce the original study's pDC-localization-supported marker pattern

## Limitations

- Statistical significance of marker expression does not by itself prove single-cell localization; it depends on the quality of the upstream SC-ST mapping (STEM embeddings and mapping matrices)
- The test was demonstrated on the hSCC dataset with pDC markers; transfer to other tissues requires choosing tissue-appropriate marker genes
- ST spot-level marker expression mixes multiple cell types, so confounding from other co-localized cell types can inflate or deflate the contrast

## Evidence

- [methods] pDC marker comparison step: [evidence span withheld — anchor status: not_found]
- [abstract] STEM mapping rationale: "It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space"
- [readme] pseudo-spatial coordinate assignment: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
