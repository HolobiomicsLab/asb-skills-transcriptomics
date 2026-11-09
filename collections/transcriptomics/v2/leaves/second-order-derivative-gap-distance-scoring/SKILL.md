---
name: second-order-derivative-gap-distance-scoring
description: Use when a large scRNA-seq expression matrix (e.g. tens of thousands
  of cells) must be scanned for rare cell clusters without clustering the whole dataset,
  and clusters must be ranked by a gap-based distance score (skewness > 2).
when_to_use_negative:
- When the task is comprehensive characterisation of major cell types — use mainstream
  toolkits such as Seurat or Scater, as GapClust targets only rare populations
- When the input is already a clustered/annotated cell-type table rather than a raw
  expression matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: Implements the second-order derivative gap distance scoring and skewness-based
    candidate selection (skewness_k > 2) for rare cell detection
  repo: github.com/fabotao/GapClust
- name: Seurat
  role: Prerequisite R package (>= 3.1.0) for handling the scRNA-seq expression matrix
    that GapClust scores
- name: rflann
  role: Fast nearest-neighbour search (>= 1.8.4) underlying the distance computations
    used by the gap score
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:ba685bf07c16a37d7fc4593ed10716ce98c4a023da738c120f3614ad71c51b3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - Seurat
  - rflann
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# second-order-derivative-gap-distance-scoring

## Summary

GapClust's core statistic: score candidate rare clusters by a second-order derivative type distance score that exploits the inter-cluster gap, ranking minor clusters by skewness so that cells in a rare cluster whose nearest out-of-cluster neighbours are far away stand out. It underpins ultra-fast rare cell detection in large scRNA-seq datasets such as the 68k PBMC dataset.

## When to use

Use when a large scRNA-seq expression matrix (e.g. tens of thousands of cells) must be scanned for rare cell clusters without clustering the whole dataset, and clusters must be ranked by a gap-based distance score (skewness > 2).

## When NOT to use

- When the task is comprehensive characterisation of major cell types — use mainstream toolkits such as Seurat or Scater, as GapClust targets only rare populations
- When the input is already a clustered/annotated cell-type table rather than a raw expression matrix

## Inputs

- Single-cell RNA-seq expression matrix (e.g. the 68k PBMC expression matrix) as processed by Seurat >= 3.1.0
- Optional: list of previously identified rare cell types to remove before a rerun

## Outputs

- Ranked list of candidate rare clusters with skewness scores (candidates where skewness<k> > 2)
- Rare cluster assignments and sizes (e.g. 3-cell CD14+ monocyte cluster R1 and 185-cell CD34+ megakaryocyte cluster R2)
- Marker-gene profiles of detected rare clusters, e.g. saved to rare_cells_68k_pbmc.csv

## How to apply

Run GapClust (github.com/fabotao/GapClust, requires R >= 3.4.0, Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1) on the expression matrix. The method exploits the observation that for a cell in a minor cluster Cn of size n, the distance to its nearest neighbours outside Cn is far greater than to neighbours inside it; a second-order derivative type distance score captures the variation of these distance changes to utilise the gap information. Candidate rare cluster sizes k are identified if the skewness<k> > 2, giving a ranked list of candidates. Annotate top-ranked minor clusters with marker genes (e.g. CD14+ monocytes, CD34+ megakaryocytes in the 68k PBMC dataset) and, to find additional hidden clusters, rerun GapClust with the identified rare cell types removed.

## Related tools

- **GapClust** (Implements the second-order derivative gap distance scoring and skewness-based candidate selection (skewness_k > 2) for rare cell detection) — github.com/fabotao/GapClust
- **Seurat** (Prerequisite R package (>= 3.1.0) for handling the scRNA-seq expression matrix that GapClust scores)
- **rflann** (Fast nearest-neighbour search (>= 1.8.4) underlying the distance computations used by the gap score)

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Candidate list contains only k values whose skewness<k> exceeds the threshold of 2
- Detected rare clusters reproduce reported sizes and markers, e.g. a 3-cell CD14+ monocyte cluster and a 185-cell CD34+ megakaryocyte cluster in the 68k PBMC dataset
- Cross-check against GiniClust and FiRE outputs: known rare clusters should agree where those methods report detections (e.g. the 185-cell megakaryocyte cluster)
- Runtime and memory remain far below RaceID/GiniClust on ultra-large datasets, consistent with GapClust's reported state-of-the-art speed and memory efficiency

## Limitations

- Results for many real datasets are deferred to supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5), so the article body does not report per-dataset details
- No version numbers for compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE) are given, complicating exact benchmark reproduction
- GiniClust was reported to fail in processing very large datasets and RaceID has unrealistic running time, so cross-validation on ultra-large inputs is limited
- Some competing methods do not discriminate outliers from minor cell types (FiRE) or require major cell types to be determined first (CellSIUS), affecting comparability of detected clusters

## Evidence

- [intro] Design rationale: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
- [intro] Score definition: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [discussion] Candidate threshold: "Candidate k can be identified if the skewness<k> > 2."
- [abstract] Application scale: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [readme] README prerequisites: "Seurat >=  3.1.0 rflann >=1.8.4 (GitHub: YeeJeremy/rflann) irlba >=2.3.3 e1071 >= 1.7-0.1"
