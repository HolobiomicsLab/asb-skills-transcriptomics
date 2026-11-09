---
name: morans-i-spatial-autocorrelation-testing
description: Use when you have scored spots or cells (e.g., SenePy hub scores) on
  spatially resolved data and need to test whether the score is spatially clustered
  or colocalized with a marker-defined region such as Col1a1+ fibrosis.
when_to_use_negative:
- Data are non-spatial single-cell suspensions — sp.score_hub applies, but Moran's
  I spatial autocorrelation and fibrosis colocalization testing are not meaningful
  without spatial coordinates
- The AnnData expression matrix is already scaled — SenePy scoring assumes raw or
  normalized (not scaled) data because score_hub binarizes by default
- Species is human — this workflow retrieves the 8 mouse heart hub signatures and
  would need the corresponding human hubs instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SenePy
  role: Provides load_hubs, get_genes, and score_hub to retrieve the mouse heart hub
    signatures and compute per-spot senescence scores with default binarization and
    connection-count importance weighting
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Container format for the Visium spot-by-gene count matrices; per-spot SenePy
    scores are written into adata.obs columns
- name: scanpy
  role: Downstream single-cell/spatial analysis framework in which the AnnData Visium
    objects are loaded and processed before spatial statistics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:623efbba64913856417b91f01c775716d9fb199af301bfc1e5bb3f0c03580ef6
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# morans-i-spatial-autocorrelation-testing

## Summary

Tests whether a per-spot composite senescence score derived from SenePy hub signatures on Visium myocardial-infarction sections exhibits significant spatial autocorrelation (Moran's I), and quantifies colocalization of senescence-outlier spots with Col1a1-expressing fibrotic regions across the post-MI time course.

## When to use

Use when you have scored spots or cells (e.g., SenePy hub scores) on spatially resolved data and need to test whether the score is spatially clustered or colocalized with a marker-defined region such as Col1a1+ fibrosis.

## When NOT to use

- Data are non-spatial single-cell suspensions — sp.score_hub applies, but Moran's I spatial autocorrelation and fibrosis colocalization testing are not meaningful without spatial coordinates
- The AnnData expression matrix is already scaled — SenePy scoring assumes raw or normalized (not scaled) data because score_hub binarizes by default
- Species is human — this workflow retrieves the 8 mouse heart hub signatures and would need the corresponding human hubs instead

## Inputs

- 10x Visium myocardial-infarction time-course count matrices loaded into a scanpy-compatible AnnData object (normalized or raw, not scaled)
- Mouse SenePy hub signatures (8 heart hubs) from sp.load_hubs(species='Mouse') filtered by hubs.metadata tissue
- Col1a1 expression values or fibrosis-region spatial annotation per spot

## Outputs

- Per-spot composite senescence scores and senescence-outlier labels for each Visium section
- Colocalization statistics of senescence-outlier spots with Col1a1+ fibrotic areas per time point
- Moran's I spatial autocorrelation values per tissue section
- mi_senescence_colocalization_results.csv containing scores, outlier labels, colocalization and Moran's I values

## How to apply

Load the mouse hub signatures with sp.load_hubs(species='Mouse'), filter hubs.metadata to heart tissue, and retrieve the 8 heart hub gene signatures via hubs.get_genes((tissue, cell, hub_num)). Load the 10x Visium MI time-course count matrices into an AnnData object (normalized or raw, but not scaled, since sp.score_hub binarizes by default), then score every spot against each of the 8 signatures with sp.score_hub using its default importance weighting by hub-network gene connections. Aggregate the 8 per-spot scores into a composite senescence score, flag upper-tail senescence-outlier spots across the time course, overlay them on Col1a1+ fibrotic regions, and quantify colocalization. Finally compute Moran's I on the composite score per tissue section to test spatial coherence, and export per-spot scores, outlier labels, colocalization statistics, and Moran's I values to a results CSV.

## Related tools

- **SenePy** (Provides load_hubs, get_genes, and score_hub to retrieve the mouse heart hub signatures and compute per-spot senescence scores with default binarization and connection-count importance weighting) — https://github.com/jaleesr/senepy
- **AnnData** (Container format for the Visium spot-by-gene count matrices; per-spot SenePy scores are written into adata.obs columns)
- **scanpy** (Downstream single-cell/spatial analysis framework in which the AnnData Visium objects are loaded and processed before spatial statistics)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); sig = hubs.get_genes(('Heart', 'fibroblast', 1)); adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Heart', 'fibroblast', 1)])
```

## Evaluation signals

- Senescence-outlier spots show strong colocalization with Col1a1-expressing fibrotic regions by 7 days post-MI, mirroring the reported finding
- Moran's I indicates significant spatial clustering of the composite senescence score in 8 of 9 Visium samples
- Each of the 8 heart hubs contributes to the composite burden with distinguishable temporal patterns across the MI time course
- score_hub was run with default settings (binarize=True, importance=True) and the input AnnData was not scaled
- Output CSV contains per-spot scores, outlier labels, colocalization statistics, and Moran's I values as specified

## Limitations

- Classical senescence markers such as Cdkn2a and Cdkn1a are unreliable as sole indicators due to dropout and transient p16 expression, which is why hub signatures rather than single markers are scored
- There is a paucity of tissue- and cell-specific senescence markers due to cellular senescence heterogeneity; hub composition may not capture all senescent states
- SenePy signatures do not account for downregulated genes in senescent cells (no negative markers), and the methodology discounts CS genes overlapping the transcriptional shift of organismal aging

## Evidence

- [methods] load_hubs: "Initializes hub object that contains senePy gene signatures and metadata for either Human or Mouse."
- [methods] score_hub: "Function to score single-cell transcriptomes based on a given senescence gene signature."
- [readme] default binarization: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] raw/normalized input requirement: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [readme] importance weighting rationale: "it weighs the importance of each gene based on the number of connections that gene has in the hub network"
- [abstract] MI disease context: "We use SenePy to map the kinetics of senescent cell accumulation in healthy aging as well as multiple disease contexts, including tumorigenesis, inflammation, and myocardial infarction."
