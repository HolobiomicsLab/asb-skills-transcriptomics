---
name: singscore-rank-based-scoring
description: Use when per-cell gene set scores are needed from a Scanpy AnnData object
  (e.g. h5ad) and cells must be scored on ranks rather than raw counts — especially
  when count sparsity or sequencing-depth variation would destabilize bulk-style scoring.
when_to_use_negative:
- Input data cannot be rank-ordered or the analyst wants an unranked method — singscore
  'requires ranked data' (use unranked methods like mean_z or robust_std instead)
- The data are already aggregated pseudobulk profiles, where the nearest-neighbor
  smoothing step adds nothing and bulk gene set scoring applies directly
- The goal is to score counts with no neighborhood graph available and no cluster/groupby
  column to construct groups, since gssnng NNGs are built per user-defined group
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: gssnng
  role: Implements nearest-neighbor graph smoothing and exposes singscore as a score_method
    in scores_cells.with_gene_sets, writing results to adata.obsm['gssnng_estimate']
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Provides the AnnData container (h5ad) and cluster/groupby labels that gssnng
    operates on, e.g. sc.datasets.pbmc3k_processed()
  repo: https://github.com/IlyaLab/gssnng
- name: AnnData
  role: Data structure holding cells, counts, and obs metadata; smoothed counts are
    stored back as an AnnData-compatible obsm entry
  repo: https://github.com/IlyaLab/gssnng
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:34d8ebf17945e18f07fc80df9e6c0bafc820ed2969262caa2a25ee32e0004b26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# singscore-rank-based-scoring

## Summary

Score single cells for a gene set using singscore — the normalized, median-centered mean rank of gene set members — on nearest-neighbor-smoothed count data (gssnng), producing per-cell signature scores robust to single-cell sparsity and downsampling noise.

## When to use

Use when per-cell gene set scores are needed from a Scanpy AnnData object (e.g. h5ad) and cells must be scored on ranks rather than raw counts — especially when count sparsity or sequencing-depth variation would destabilize bulk-style scoring.

## When NOT to use

- Input data cannot be rank-ordered or the analyst wants an unranked method — singscore 'requires ranked data' (use unranked methods like mean_z or robust_std instead)
- The data are already aggregated pseudobulk profiles, where the nearest-neighbor smoothing step adds nothing and bulk gene set scoring applies directly
- The goal is to score counts with no neighborhood graph available and no cluster/groupby column to construct groups, since gssnng NNGs are built per user-defined group

## Inputs

- Scanpy AnnData object (h5ad) with a neighborhood graph or cluster labels in .obs (e.g. 'louvain')
- Gene set file in GMT format (e.g. cibersort_lm22.gmt), MSigDB nomenclature with gene sets defaulting to 'up'

## Outputs

- Per-cell, per-gene-set singscore values stored in adata.obsm['gssnng_estimate'] (one column per gene set)
- Optionally, activities extracted as an AnnData object via dc.get_acts(adata, obsm_key='gssnng_estimate')

## How to apply

Read the processed AnnData (e.g. sc.datasets.pbmc3k_processed()) and score it with gssnng's scores_cells.with_gene_sets using score_method='singscore', ranked=True, and a .gmt gene set file such as cibersort_lm22.gmt; scores land as a column per gene set in adata.obsm['gssnng_estimate']. Because singscore requires ranked data, ranked=True is mandatory; optionally pass method_params={'normalization':'theoretical'} or {'normalization':'standard'} — the theoretical option standardizes against the theoretical max/minimum ranks for the given gene set as described in the singscore manuscript. Smooth first on the nearest neighbor graph with smooth_mode='connectivity' (close neighbors weighted more) and recompute_neighbors set to the desired K (e.g. 32), recomputed within each groupby group (e.g. 'louvain' clusters), which creates mini-pseudobulk profiles per cell before ranking. This smoothing is what makes scores robust to severe downsampling: in the source task, 32-neighbor smoothed scores kept AUC > 0.95 for distinguishing B cells until fewer than 10% of total counts remained, while unsmoothed scores degraded immediately.

## Related tools

- **gssnng** (Implements nearest-neighbor graph smoothing and exposes singscore as a score_method in scores_cells.with_gene_sets, writing results to adata.obsm['gssnng_estimate']) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Provides the AnnData container (h5ad) and cluster/groupby labels that gssnng operates on, e.g. sc.datasets.pbmc3k_processed()) — https://github.com/IlyaLab/gssnng
- **AnnData** (Data structure holding cells, counts, and obs metadata; smoothed counts are stored back as an AnnData-compatible obsm entry) — https://github.com/IlyaLab/gssnng

## Examples

```
from gssnng import score_cells; score_cells.with_gene_sets(adata, gene_set_file='cibersort_lm22.gmt', groupby='louvain', smooth_mode='connectivity', recompute_neighbors=32, score_method='singscore', ranked=True)
```

## Evaluation signals

- Verify scores appear in adata.obsm['gssnng_estimate'] with one column per gene set from the .gmt file
- Confirm ranked=True was set, since singscore 'requires ranked data'
- Check that the normalization method_params value is one of {'normalization': 'theoretical'} or {'normalization': 'standard'}
- For a robustness test: with smooth_mode='connectivity' and recompute_neighbors=32, the AUC of the signature score separating B cells from non-B cells should stay above 0.95 until the downsampled count fraction drops below 10%
- Compare against smooth_mode='off' as a control — the non-smoothed AUC curve should degrade at higher retained-count fractions than the smoothed curve

## Limitations

- singscore only works with ranked data (ranked=True); it is not applicable to raw, unranked count matrices
- Nearest neighbor graphs are constructed within user-defined groupby groups, so scores depend on the chosen grouping (e.g. cluster or cluster × sample); fewer neighbors or poorly defined groups will change smoothing behavior
- Smoothing shares expression information across neighboring cells, producing mini-pseudobulk profiles rather than purely single-cell measurements — appropriate when robustness to noise is the goal, but not when raw per-cell measurements are required

## Evidence

- [methods] singscore requires ranked data and median-centered normalization: "singscore:      Normalised mean (median centered) ranks (requires ranked data)"
- [intro] Smoothing creates mini-pseudobulk profiles scorable by bulk single-sample methods: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring"
- [abstract] Smoothing on the NNG remediates sparsity and technical noise: [evidence span withheld — anchor status: not_found]
- [readme] connectivity smoothing weights close neighbors more: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
- [readme] singscore method_params normalization options: "singscore:  {'normalization', 'theoretical'}, {'normalization', 'standard'}"
