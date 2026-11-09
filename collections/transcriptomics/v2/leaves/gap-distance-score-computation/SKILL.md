---
name: gap-distance-score-computation
description: Use when a scRNA-seq count matrix (e.g. 4491 non-irradiated intestinal
  crypt cells) must be scanned for rare cell types, and you need to compute the nearest-neighbour
  distance profile and second-order derivative gap score to nominate candidate rare-cluster
  sizes k.
when_to_use_negative:
- The goal is comprehensive characterisation of all major cell types, not rare ones
  — the article defers that to toolkits such as Seurat and Scater.
- Input is already a dimensionality-reduced feature/cluster table without the per-cell
  expression profiles GapClust needs for nearest-neighbour distances.
- 'Known rare populations must be re-detected rather than removed: rerunning on data
  containing already-identified rare types risks re-flagging them instead of discovering
  hidden clusters.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Implements the nearest-neighbour distance computation and second-order derivative
    gap score used to nominate rare-cluster candidate sizes k (skewness<k> > 2).
  repo: https://github.com/fabotao/GapClust
- name: Seurat
  role: Downstream differential-expression check that the scored rare cluster (e.g.
    R2, 29 cells) is enriched for marker genes such as Dclk1 and Trpm5.
- name: rflann
  role: Required R prerequisite (>= 1.8.4) providing the fast nearest-neighbour search
    underlying the distance profile.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:577908620953af77d511e9ec68fbbb6aec25cfa02538dec29e88d6fd0a307141
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabotao/GapClust
  tools:
  - GapClust
  - Seurat
  - rflann
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gap-distance-score-computation

## Summary

Computation of GapClust's second-order derivative gap-distance score: for each cell, nearest-neighbour distances are scanned across candidate k, and the variation of distance changes exposes minor clusters (e.g. 29-cell tuft cell cluster R2) separated by inter-cluster gaps from voluminous neighbours.

## When to use

Use when a scRNA-seq count matrix (e.g. 4491 non-irradiated intestinal crypt cells) must be scanned for rare cell types, and you need to compute the nearest-neighbour distance profile and second-order derivative gap score to nominate candidate rare-cluster sizes k.

## When NOT to use

- The goal is comprehensive characterisation of all major cell types, not rare ones — the article defers that to toolkits such as Seurat and Scater.
- Input is already a dimensionality-reduced feature/cluster table without the per-cell expression profiles GapClust needs for nearest-neighbour distances.
- Known rare populations must be re-detected rather than removed: rerunning on data containing already-identified rare types risks re-flagging them instead of discovering hidden clusters.

## Inputs

- scRNA-seq gene-by-cell count matrix (e.g. 4491 non-irradiated intestine crypt cells from Arshad et al., pandas/Scanpy-compatible IO)
- range of candidate nearest-neighbour counts k to scan
- R environment with GapClust prerequisites (Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1, R >= 3.4.0)

## Outputs

- per-cell nearest-neighbour distance profile across candidate k
- second-order derivative gap scores with skewness<k> values per candidate cluster size
- rare-cluster membership table (cell barcode, cluster ID, cluster size, marker status), e.g. the 29-cell R2 cluster with Dclk1/Trpm5 enrichment, written to gapclust_rare_cells.csv

## How to apply

Load the scRNA-seq count matrix (e.g. the 4491-cell intestine crypt dataset from Arshad et al.) into an R/Seurat >= 3.1.0 environment with the GapClust prerequisites (rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1). Using rflann nearest-neighbour search, compute for each cell the distances to its nearest neighbours across candidate k, exploiting the design principle that a cell in a minor cluster Cn of size n has far larger distances to neighbours outside the cluster than inside it. For each candidate k compute the second-order derivative type distance score capturing the variation of distance changes, and flag candidate k as a rare-cluster size when skewness<k> > 2. Repeat or iterate: after extracting one rare cluster (e.g. 29 cells), rerun with identified rare cell types removed to surface additional hidden clusters, then verify marker enrichment (Dclk1, Trpm5 for tuft cells) with Seurat FindMarkers-style differential expression.

## Related tools

- **GapClust** (Implements the nearest-neighbour distance computation and second-order derivative gap score used to nominate rare-cluster candidate sizes k (skewness<k> > 2).) — https://github.com/fabotao/GapClust
- **Seurat** (Downstream differential-expression check that the scored rare cluster (e.g. R2, 29 cells) is enriched for marker genes such as Dclk1 and Trpm5.)
- **rflann** (Required R prerequisite (>= 1.8.4) providing the fast nearest-neighbour search underlying the distance profile.)

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- The scored output must contain a candidate rare cluster of 29 cells (R2) in the intestine crypt dataset, matching the tuft-cell identification of Arshad et al.
- Each nominated candidate k must satisfy the skewness<k> > 2 threshold; cluster sizes reported should match GapClust's learned cluster sizes.
- R2 cells must be enriched for the tuft-cell markers Dclk1 and Trpm5 (Seurat/FindMarkers-style DE), consistent with Fig. 5a-c.
- Rerunning GapClust with identified rare cell types removed should surface additional hidden rare clusters, not re-detect the same 29 cells.
- Runtime and memory should stay within GapClust's reported light-weight envelope relative to RaceID/GiniClust/CellSIUS/FiRE on ultra-large datasets.

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported for published analyses, so exact reproduction may require tuning.
- FiRE-style limitation: rare-cluster scoring must discriminate minor cell types from mere outliers; doublet/subsampled benchmarking (293T-Jurkat, 68k PBMC) was used to assess this, but performance degrades near single-outlier regimes.
- Results for many real datasets (mouse tracheal epithelial cells, hippocampus, Drosophila wing disc, etc.) are deferred to unlinked supplementary figures, limiting direct validation of score behaviour on those inputs.

## Evidence

- [intro] design inspired by nearest-neighbour gap: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
- [intro] second-order derivative score: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [discussion] skewness threshold for candidate k: "Candidate k can be identified if the skewness<k> > 2."
- [discussion] rerun to discover hidden clusters: "To discover these hidden clusters, users can rerun GapClust in no time with the identified rare cell types removed"
- [readme] README rationale for gap statistics: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out"
- [readme] README prerequisites: "Seurat >=  3.1.0
rflann >=1.8.4 (GitHub: YeeJeremy/rflann)
irlba >=2.3.3
e1071 >= 1.7-0.1"
