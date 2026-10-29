---
name: geneset-signature-scoring
description: Use when per-cell gene set scores from sparse scRNA-seq count matrices
  are too noisy for downstream tasks (e.g. AUC discrimination of cell types), and
  you want scores computed on neighbor-smoothed data in a Scanpy AnnData workflow.
when_to_use_negative:
- When smooth_mode='off' is acceptable, i.e. the count matrix is not sparse/noisy
  and bulk-style scoring without neighborhood smoothing suffices — smoothing changes
  per-cell values into neighbor-averaged profiles
- When the input is bulk RNA-seq rather than single-cell AnnData; gssnng is built
  for scRNA-seq and nearest neighbor graphs of cells
- When using a scoring method that does not match the data mode — e.g. singscore and
  ssGSEA require ranked data, while mean_z and robust_std recommend unranked counts
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: gssnng
  role: Core package that smooths counts on the nearest neighbor graph and computes
    per-cell gene set scores; installed via pip install gssnng or from GitHub
  repo: github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the AnnData h5ad input and supplies datasets (e.g. pbmc3k_processed)
    and the obs columns used for groupby
- name: AnnData
  role: Data container format; smoothed counts and gssnng estimates (obsm['gssnng_estimate'])
    are returned as AnnData-compatible structures
- name: decoupleR
  role: Alternative API for scoring cells with OmniPath/decoupler-style gene sets
    and extracting activities via dc.get_acts
- name: Python
  role: Runtime language; gssnng is installed from the Python package index (PyPI)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:ebb710f938fc2827545a102953cdd11c5793053bbb99091d595b583fc0c68509
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Python
  - Scanpy
  - decoupleR
  - gssnng
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# geneset-signature-scoring

## Summary

Score single-cell RNA-seq cells against gene set signatures (e.g. 'B cells naive' from cibersort_lm22.gmt) using gssnng, which smooths counts on the nearest neighbor graph to create mini-pseudobulk profiles before applying bulk-style single-sample scoring methods such as singscore.

## When to use

Use when per-cell gene set scores from sparse scRNA-seq count matrices are too noisy for downstream tasks (e.g. AUC discrimination of cell types), and you want scores computed on neighbor-smoothed data in a Scanpy AnnData workflow.

## When NOT to use

- When smooth_mode='off' is acceptable, i.e. the count matrix is not sparse/noisy and bulk-style scoring without neighborhood smoothing suffices — smoothing changes per-cell values into neighbor-averaged profiles
- When the input is bulk RNA-seq rather than single-cell AnnData; gssnng is built for scRNA-seq and nearest neighbor graphs of cells
- When using a scoring method that does not match the data mode — e.g. singscore and ssGSEA require ranked data, while mean_z and robust_std recommend unranked counts

## Inputs

- Scanpy AnnData object (h5ad file) with processed single-cell counts
- Gene set file in .gmt format (e.g. cibersort_lm22.gmt), one gene set per line, MSigDB nomenclature
- Optional obs column for groupby (e.g. 'louvain' cluster labels)
- Optional Decoupler/OmniPath-style model from the decoupler API as an alternative gene set source

## Outputs

- Per-cell gene set scores stored as columns in adata.obsm['gssnng_estimate']
- Activities as an AnnData object via dc.get_acts(adata, obsm_key='gssnng_estimate')
- An AnnData object with smoothed counts (when smooth_mode is 'adjacency' or 'connectivity')

## How to apply

Load the processed dataset (e.g. sc.datasets.pbmc3k_processed()) as a Scanpy AnnData object and supply gene sets as a .gmt file (e.g. cibersort_lm22.gmt, MSigDB nomenclature where gene sets default to up). Call scores_cells.with_gene_sets with a score_method such as 'singscore' (requires ranked=True), a groupby column like 'louvain' so nearest neighbor graphs are built within groups, and smooth_mode='connectivity' to weight close neighbors more (or 'adjacency' for equal weights, or 'off' as an unsmoothed control). Set recompute_neighbors to a value like 32 so neighbors are rebuilt within each group; each cell's neighbors form a mini-pseudobulk expression profile that stabilizes scores against sparsity and technical noise. Scores appear per gene set as columns in adata.obsm['gssnng_estimate'] and can be extracted as an AnnData of activities with dc.get_acts(adata, obsm_key='gssnng_estimate').

## Related tools

- **gssnng** (Core package that smooths counts on the nearest neighbor graph and computes per-cell gene set scores; installed via pip install gssnng or from GitHub) — github.com/IlyaLab/gssnng
- **Scanpy** (Reads the AnnData h5ad input and supplies datasets (e.g. pbmc3k_processed) and the obs columns used for groupby)
- **AnnData** (Data container format; smoothed counts and gssnng estimates (obsm['gssnng_estimate']) are returned as AnnData-compatible structures)
- **decoupleR** (Alternative API for scoring cells with OmniPath/decoupler-style gene sets and extracting activities via dc.get_acts)
- **Python** (Runtime language; gssnng is installed from the Python package index (PyPI))

## Examples

```
scores_cells.with_gene_sets(adata, gene_set_file='cibersort_lm22.gmt', groupby='louvain', smooth_mode='connectivity', recompute_neighbors=32, score_method='singscore', ranked=True)
```

## Evaluation signals

- Scores appear as expected schema: one column per gene set in adata.obsm['gssnng_estimate']
- Smoothed scores are more stable than non-smoothed controls under downsampling: e.g. in the pbmc3k task, 32-neighbor connectivity-smoothed 'B cells naive' AUC stays above 0.95 until fewer than 10% of total counts remain, while non-smoothed AUC drops immediately
- geneset_overlap diagnostic reports the number or fraction of genes expressed past a threshold for each gene set, confirming sufficient signature-gene coverage
- When smooth_mode='off', scores should reproduce the unsmoothed control behavior, verifying the smoothing step is the only difference
- Grouped computation: NNG construction and smoothing run per groupby category (e.g. per 'louvain' cluster), so results should not mix cells across groups

## Limitations

- Smoothed scores are neighbor-averaged mini-pseudobulk profiles, not purely single-cell measurements; interpretation must account for shared information across transcriptomic neighborhoods
- Source card missing signals: no discussion section content found in the source document, so caveats from discussion are unavailable
- Score methods impose data-mode requirements: singscore and ssGSEA need ranked data; mean_z and robust_std recommend unranked; mismatched settings will distort scores

## Evidence

- [abstract] sparsity and technical noise of single-cell expression measures create difficulties for these methods: [evidence span withheld — anchor status: not_found]
- [intro] mini-pseudobulk profiles: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
- [methods] obsm output location: "3. Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [readme] smooth_mode semantics: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
- [readme] singscore requires ranked data: "**singscore**:      Normalised mean (median centered) ranks (requires ranked data)"
- [intro] scverse/AnnData compatibility: [evidence span withheld — anchor status: not_found]
