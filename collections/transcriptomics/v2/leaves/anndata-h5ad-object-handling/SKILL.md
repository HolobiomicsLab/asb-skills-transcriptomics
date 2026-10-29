---
name: anndata-h5ad-object-handling
description: Use when building or persisting a single-cell count matrix (e.g. 10x
  PBMC 3k counts or Splatter-simulated 10000 genes x 3000 cells) as an AnnData object
  for downstream scanpy/Cytocipher analysis, or when writing/reading .h5ad files.
when_to_use_negative:
- The data is not a cell x gene single-cell count matrix with per-cell annotations
  (AnnData is designed for scRNA-seq-style tabular omics)
- The object still holds raw counts when a Cytocipher call is planned — Cytocipher
  expects log-cpm normalised data with existing cluster labels, so normalise first
  instead of passing raw AnnData
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AnnData
  role: Container format holding counts, gene metadata, and cluster/enrichment annotations
    passed to Cytocipher
  repo: https://github.com/BradBalderson/Cytocipher
- name: scanpy
  role: Loads and preprocesses the AnnData (e.g. scanpy.datasets.pbmc3k) via the standard
    log-cpm workflow before cluster labelling
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: Cytocipher
  role: scverse-compatible consumer of the AnnData object that adds marker, enrichment-score,
    and merged-cluster annotations in place
  repo: https://github.com/BradBalderson/Cytocipher
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:bf183db294e1d6cf1cf576cee0c10bd7e0ed124cd961373f00bb2aa523591656
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Cytocipher
  - Splatter
  - scanpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# anndata-h5ad-object-handling

## Summary

Wrangle single-cell RNA-seq data as AnnData objects (counts, gene metadata, cluster labels, enrichment-score outputs) and persist them as .h5ad files, as required by the scverse/scanpy/Cytocipher ecosystem.

## When to use

Use when building or persisting a single-cell count matrix (e.g. 10x PBMC 3k counts or Splatter-simulated 10000 genes x 3000 cells) as an AnnData object for downstream scanpy/Cytocipher analysis, or when writing/reading .h5ad files.

## When NOT to use

- The data is not a cell x gene single-cell count matrix with per-cell annotations (AnnData is designed for scRNA-seq-style tabular omics)
- The object still holds raw counts when a Cytocipher call is planned — Cytocipher expects log-cpm normalised data with existing cluster labels, so normalise first instead of passing raw AnnData

## Inputs

- Raw single-cell count matrix (e.g. scanpy pbmc3k dataset raw counts or Splatter-simulated 10000 genes x 3000 cells matrix)
- Gene metadata (e.g. 15-gene DE pool / group-assigned gene sets for simulated data)
- Per-cell cluster labels (e.g. Leiden 'leiden' column, or simulated group labels)
- Existing .h5ad file to read back (e.g. simulated_pbmc3k_splat.h5ad)

## Outputs

- AnnData object with log-cpm normalised expression and a tentative cluster-label column (e.g. 'leiden')
- .h5ad file (e.g. simulated_pbmc3k_splat.h5ad) containing simulated counts, gene metadata, and group labels
- AnnData augmented in place by Cytocipher with marker genes, enrichment scores, and merged cluster labels ('leiden_merged')

## How to apply

Construct an AnnData object holding the raw or simulated count matrix together with gene metadata and per-cell labels (e.g. group or 'leiden' cluster assignments) — for example, a Splatter-simulated 10000 genes x 3000 cells matrix with group labels written to simulated_pbmc3k_splat.h5ad. Before passing the object to Cytocipher, process it via the scanpy standard workflow (as in the PBMC 3k tutorial) to obtain log-cpm normalised data with tentative cluster labels, ideally at high Leiden resolution so over-clustering exists for Cytocipher to merge. Cytocipher then augments the same AnnData in place — cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters add marker genes, enrichment scores, and 'leiden_merged' labels keyed on the 'leiden' column. Judgement of correctness: the AnnData must be log-cpm normalised with a tentative cluster label column before Cytocipher calls, and enrich_heatmap must display cell-by-cluster enrichment scores for both the original and merged cluster keys.

## Related tools

- **AnnData** (Container format holding counts, gene metadata, and cluster/enrichment annotations passed to Cytocipher) — https://github.com/BradBalderson/Cytocipher
- **scanpy** (Loads and preprocesses the AnnData (e.g. scanpy.datasets.pbmc3k) via the standard log-cpm workflow before cluster labelling) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **Cytocipher** (scverse-compatible consumer of the AnnData object that adds marker, enrichment-score, and merged-cluster annotations in place) — https://github.com/BradBalderson/Cytocipher

## Examples

```
import scanpy as sc; import cytocipher as cc; data = sc.datasets.pbmc3k(); sc.pp.normalize_total(data); sc.pp.log1p(data); sc.tl.leiden(data, resolution=1.5, key_added='leiden'); cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden')
```

## Evaluation signals

- AnnData shape matches expectation (e.g. 10000 genes x 3000 cells for the simulated Splatter dataset)
- Expression matrix is log-cpm normalised before Cytocipher calls, per the README expected-input contract
- A tentative cluster-label column (e.g. 'leiden') exists in obs with high resolution / over-clustering
- .h5ad round-trip succeeds and preserves counts, gene metadata, and group labels (simulated_pbmc3k_splat.h5ad)
- cc.pl.enrich_heatmap renders a cell-by-cluster enrichment-score heatmap for 'leiden' and 'leiden_merged' keys without key errors

## Limitations

- Cytocipher requires high Leiden resolution (over-clustering) to work as intended; a coarsely clustered AnnData reduces the merging benefit
- The article's discussion section is absent, so limitations of the AnnData handling workflow (e.g. h5ad schema versioning) are not documented in context
- Only the PBMC 3k tutorial-style input is validated in the README; other preprocessing regimes are not documented

## Evidence

- [readme] expected input AnnData: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)"
- [readme] log-cpm with cluster labels: "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high"
- [methods] article methods AnnData: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [supplementary] source card h5ad output: [evidence span withheld — anchor status: not_found]
- [abstract] scverse compatibility: "we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters"
