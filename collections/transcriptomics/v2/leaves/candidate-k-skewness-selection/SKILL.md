---
name: candidate-k-skewness-selection
description: Use when running GapClust on a scRNA-seq count matrix to flag rare cell
  clusters (e.g. the 29-cell tuft-cell R2 cluster), and you need the criterion that
  turns per-k gap statistics into candidate rare clusters.
when_to_use_negative:
- When the goal is to characterise major cell types rather than rare ones — the paper
  directs that to toolkits such as Seurat or Scater, not GapClust's gap-statistic
  scan.
- When clusters are abundant and the between-cluster gap signal is expected to be
  negligible — the second-order derivative score is designed for minor clusters separated
  from voluminous neighbours.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: 'Core algorithm: computes nearest-neighbour distances, the second-order derivative
    gap score, and the skewness<k> > 2 candidate filter'
  repo: github.com/fabotao/GapClust
- name: Seurat
  role: Downstream marker testing (FindMarkers-style) to validate candidate rare clusters,
    e.g. Dclk1/Trpm5 enrichment in R2
- name: rflann
  role: Fast nearest-neighbour distance computation feeding the gap-score calculation
  repo: github.com/YeeJeremy/rflann
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:d0209366e587f91d3ff5b86c45986199688df17ae91bfdb148f44eb0ef4b2fa2
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.75
n_claims: 8
---

# candidate-k-skewness-selection

## Summary

GapClust's candidate-k selection step: scan candidate cluster sizes k using nearest-neighbour distances and a second-order derivative gap score, and accept a k as a rare-cluster candidate only when the skewness<k> statistic exceeds 2.

## When to use

Use when running GapClust on a scRNA-seq count matrix to flag rare cell clusters (e.g. the 29-cell tuft-cell R2 cluster), and you need the criterion that turns per-k gap statistics into candidate rare clusters.

## When NOT to use

- When the goal is to characterise major cell types rather than rare ones — the paper directs that to toolkits such as Seurat or Scater, not GapClust's gap-statistic scan.
- When clusters are abundant and the between-cluster gap signal is expected to be negligible — the second-order derivative score is designed for minor clusters separated from voluminous neighbours.

## Inputs

- scRNA-seq count/expression matrix (rows = cells, columns = genes), e.g. pandas/Scanpy/Seurat-compatible input such as the 4491-cell non-irradiated intestine crypt dataset
- set/range of candidate cluster sizes k to scan

## Outputs

- list of candidate rare cluster sizes k with their skewness<k> values (those > 2)
- rare-cluster membership per cell (cell barcode, cluster ID, cluster size), e.g. gapclust_rare_cells.csv

## How to apply

Load the scRNA-seq expression profiles (e.g. a pandas/Scanpy-readable count matrix such as the 4491 non-irradiated intestinal crypt cells from Arshad et al.) and compute nearest-neighbour distances per cell (GapClust uses rflann for this). For each candidate k, form the second-order derivative type distance score that captures how the distance-to-neighbours changes across neighbour ranks; the rationale is that cells in a minor cluster of size n have nearest-neighbour distances outside the cluster far greater than within it, so the gap shows up as a sharp change in the distance curve. Compute the skewness of the score distribution for each candidate k and mark k as a candidate rare-cluster size only if skewness<k> > 2. Iterate over the scanned k range, then extract cells assigned to the flagged candidate clusters and record cluster sizes (expecting, e.g., the 29-cell R2 tuft cluster in the intestine dataset). Rerun GapClust with detected rare clusters removed to surface additional hidden rare populations.

## Related tools

- **GapClust** (Core algorithm: computes nearest-neighbour distances, the second-order derivative gap score, and the skewness<k> > 2 candidate filter) — github.com/fabotao/GapClust
- **Seurat** (Downstream marker testing (FindMarkers-style) to validate candidate rare clusters, e.g. Dclk1/Trpm5 enrichment in R2)
- **rflann** (Fast nearest-neighbour distance computation feeding the gap-score calculation) — github.com/YeeJeremy/rflann

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Every reported candidate k satisfies skewness<k> > 2; no k with skewness ≤ 2 is accepted as a rare cluster.
- Detected rare cluster sizes are small relative to the dataset (e.g. 29 of 4491 cells for the tuft R2 cluster in the intestine dataset).
- Marker-gene check: the candidate cluster's top markers include known rare-cell markers (Dclk1 and Trpm5 for tuft cells), matching Fig. 5a-c.
- Output table gapclust_rare_cells.csv has columns cell barcode, cluster ID, cluster size, marker status, and its R2 rows number exactly 29.
- Rerunning GapClust after removing detected rare clusters can recover additional hidden clusters, indicating the candidate-k filter did not absorb major-cluster cells.

## Limitations

- The article's discussion section reports no explicit parameter settings for GapClust runs (e.g. K range, other thresholds beyond skewness > 2).
- Detailed real-dataset results (including supplementary figures and Supplementary Table 5) are deferred to unlinked supplementary material, limiting direct verification.
- The design assumes a clear gap between the minor cluster and its neighbouring abundant cluster; datasets without such separation may yield few or spurious skewness<k> > 2 candidates.

## Evidence

- [discussion] skewness threshold: "Candidate k can be identified if the skewness<k> > 2."
- [intro] design rationale: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [intro] neighbour-gap observation: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
- [readme] dependency: "rflann >=1.8.4 (GitHub: YeeJeremy/rflann)"
- [discussion] validation markers: "Tuft cells are identified in the intestinal dataset, validated by existing maker genes including Dclk1 and Trpm5"
