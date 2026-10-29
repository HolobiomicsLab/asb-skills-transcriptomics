---
name: rank-based-gene-set-scoring
description: Use when input is a scRNA-seq AnnData (.h5ad) count matrix and per-cell
  gene set activity scores are needed using rank-based scorers such as ssGSEA, singscore,
  RBO, or summed_up ranks, especially where sparsity/noise motivates neighbor-graph
  smoothing first.
when_to_use_negative:
- When the goal is differential activity at the group/sample level rather than per-cell
  scores — aggregated pseudobulk methods may be preferable
- When gene sets are defined by up/down directionality but the .gmt file does not
  encode _UP/_DN nomenclature — ranked scorers will treat all genes as up since genesets
  default to up
- When counts are already normalized bulk expression profiles, where the sparsity-driven
  smoothing of the nearest neighbor graph adds little
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Core package performing nearest neighbor graph smoothing and rank-based gene
    set scoring on Scanpy AnnData objects
  repo: github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the .h5ad AnnData object and manages the cell-by-gene matrix and obs
    groupings used for graph construction
- name: AnnData
  role: Data container for cells and scores; results are written to adata.obsm['gssnng_estimate']
- name: decoupleR
  role: API style for scoring cells with OmniPath gene sets and extracting activities
    via dc.get_acts
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:37c7c813c4974deeb7d0cef0766cabd1b6eb194f9144a4935bdb1a854b1500ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Python
  - Scanpy
  - decoupleR
  - gssnng
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# rank-based-gene-set-scoring

## Summary

Scores single-cell gene expression against gene sets using rank-based single-sample methods (singscore, ssGSEA, rank_biased_overlap, summed_up with ranked=True), optionally on nearest-neighbor-smoothed counts from gssnng, to quantify per-cell gene set activity in Scanpy AnnData objects.

## When to use

Use when input is a scRNA-seq AnnData (.h5ad) count matrix and per-cell gene set activity scores are needed using rank-based scorers such as ssGSEA, singscore, RBO, or summed_up ranks, especially where sparsity/noise motivates neighbor-graph smoothing first.

## When NOT to use

- When the goal is differential activity at the group/sample level rather than per-cell scores — aggregated pseudobulk methods may be preferable
- When gene sets are defined by up/down directionality but the .gmt file does not encode _UP/_DN nomenclature — ranked scorers will treat all genes as up since genesets default to up
- When counts are already normalized bulk expression profiles, where the sparsity-driven smoothing of the nearest neighbor graph adds little

## Inputs

- Scanpy AnnData object (.h5ad) containing scRNA-seq count matrix and obs metadata (e.g., treatment/group labels)
- Gene set file in .gmt format (one gene set per line, MSigDB nomenclature; _UP/_DN nomenclature for UP/DN signatures)
- Optional decoupler/OmniPath model retrieved via the decoupler API

## Outputs

- Per-cell gene set score vector/matrix stored in adata.obsm['gssnng_estimate'] (one column per gene set)
- Activities as an AnnData object via dc.get_acts(adata, obsm_key='gssnng_estimate')
- Summary tables of scores/AUCs (e.g., CSV of AUC per smoothing condition)

## How to apply

Read the AnnData object with scanpy (h5ad file) and supply gene sets as a .gmt file (MSigDB nomenclature, genesets default to up; _UP/_DN nomenclature supported for UP/DN signatures). For rank-based methods, set ranked=True so counts are rank ordered per cell before scoring; choose the scorer via score_method ('ssGSEA' with method_params={'omega': 0.75}, 'singscore' with theoretical or standard normalization, 'rank_biased_overlap' with rbo_depth, or 'summed_up' summing ranks). To reduce single-cell sparsity, smooth counts on the nearest neighbor graph first — either within score_cells.with_gene_sets via smooth_mode ('adjacency' weights all neighbors equally, 'connectivity' weights close neighbors more, 'off' for none) with recompute_neighbors set per groupby group, or via a separate smoothing step that creates mini-pseudobulk profiles per cell before re-ranking and re-scoring. Scores are returned per cell, with each gene set appearing as a column in adata.obsm['gssnng_estimate']; activities can be extracted as AnnData with dc.get_acts.

## Related tools

- **gssnng** (Core package performing nearest neighbor graph smoothing and rank-based gene set scoring on Scanpy AnnData objects) — github.com/IlyaLab/gssnng
- **Scanpy** (Reads the .h5ad AnnData object and manages the cell-by-gene matrix and obs groupings used for graph construction)
- **AnnData** (Data container for cells and scores; results are written to adata.obsm['gssnng_estimate'])
- **decoupleR** (API style for scoring cells with OmniPath gene sets and extracting activities via dc.get_acts)

## Examples

```
q = sc.read_h5ad('phagocytes.h5ad'); score_cells.with_gene_sets(q, gene_set_file='lps_sig.gmt', groupby='lps_stat', smooth_mode='connectivity', recompute_neighbors=8, score_method='ssGSEA', method_params={'omega': 0.75}, ranked=True)
```

## Evaluation signals

- Scores appear as a column per gene set in adata.obsm['gssnng_estimate'] after scoring
- Downstream discrimination improves with smoothing: e.g., LPS summed_up AUC rises from ~0.95 unsmoothed to ~1.0 smoothed; injury signature AUC improves from 0.62 to 0.71 (RBO) and 0.69 to 0.8 (ssGSEA)
- ranked=True was set for methods that require ranked data (singscore, ssGSEA, rank_biased_overlap, summed_up on ranks)
- smooth_mode and recompute_neighbors settings match the intended graph: connectivity weighting with neighbors recomputed within each groupby group (e.g., K in {4, 8, 32})
- Up/down signatures are correctly encoded in the .gmt via _UP/_DN nomenclature before scoring

## Limitations

- Single-cell sparsity and technical noise degrade gene set scores designed for bulk data, motivating smoothing — unsmoothed rank scores may be noisy
- Scorer-specific parameters must be tuned: ssGSEA omega (0.75), RBO rbo_depth, singscore normalization (theoretical vs standard)
- Some methods recommend unranked data (robust_std, mean_z), so rank-based scoring is not universally applicable across gssnng methods

## Evidence

- [readme] ssGSEA and ranked scoring methods: "ssGSEA:         Single sample GSEA based on ranked data."
- [readme] rank-based overlap definition: "rank_biased_overlap:  RBO, Weighted average of agreement between sorted ranks and gene set."
- [intro] smoothing rationale: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
- [abstract] sparsity motivation: "However, the sparsity and technical noise of single-cell expression measures create difficulties for these methods"
- [methods] score output location: "3. Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [readme] smooth_mode semantics: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
