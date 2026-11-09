---
name: trajectory-inference-with-monocle3
description: Use when a time-series scRNA-seq dataset (e.g. UMI counts from Fluidigm
  C1 or 10X) needs branch/pseudotime reconstruction before time-dependent covariate
  DEG or subcluster testing.
when_to_use_negative:
- The input is already trajectory-ordered pseudotime/cluster output from another TI
  method — use scTITANS 'partial' mode instead of re-running Monocle3
- 'The dataset is not time-series or lacks a branching structure that trajectory inference
  can resolve (article scope: DEGs/subclusters varying over time)'
- Cell counts are too low for reliable graph learning after filtering (cf. MRD, where
  RPE/margin/periocular mesenchyme/lens epithelial cells were dropped for low cell
  counts)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: Monocle3
  role: Constructs the trajectory, performs Louvain clustering and UMAP dimension
    reduction, and calculates pseudotime for each cell
- name: scTITANS
  role: Wrapper workflow that consumes the Monocle3 trajectory and pseudotime for
    time-dependent covariate DEG and subcluster testing
  repo: https://github.com/ZJUFanLab/scTITANS
- name: scran
  role: Pool-and-deconvolute normalization of the filtered count matrix prior to trajectory
    inference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:c91e12fa58df3c8ec5a6d989057636635978f0b27491a240412bf92461242a2b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  repo_url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Monocle3
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# trajectory-inference-with-monocle3

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct a single-cell trajectory and per-cell pseudotime with Monocle3 (Louvain clustering, UMAP reduction, trajectory graph learning) as the TI backbone of the scTITANS time-series DEG/subcluster workflow.

## When to use

Use when a time-series scRNA-seq dataset (e.g. UMI counts from Fluidigm C1 or 10X) needs branch/pseudotime reconstruction before time-dependent covariate DEG or subcluster testing.

## When NOT to use

- The input is already trajectory-ordered pseudotime/cluster output from another TI method — use scTITANS 'partial' mode instead of re-running Monocle3
- The dataset is not time-series or lacks a branching structure that trajectory inference can resolve (article scope: DEGs/subclusters varying over time)
- Cell counts are too low for reliable graph learning after filtering (cf. MRD, where RPE/margin/periocular mesenchyme/lens epithelial cells were dropped for low cell counts)

## Inputs

- Gene expression matrix (genes x cells) of raw UMI counts, e.g. from Cell Ranger (10X) or Fluidigm C1 (GEO accession such as GSE52529)
- Optionally cell_metadata (cells x rows with clusters/types) and gene_metadata (with gene_short_name column) for scTITANS partial mode
- Optionally a root cluster/cell type name, or 'none' if unknown

## Outputs

- Monocle3 trajectory graph with branch structure (e.g. two branches for HSMM)
- Per-cell pseudotime ordering used as the time-dependent covariate
- Louvain cluster assignments and UMAP embedding

## How to apply

After scran pool-and-deconvolute normalization of the filtered UMI matrix, run Monocle3 inside scTITANS: cluster cells with the Louvain algorithm, reduce dimensionality with UMAP, learn the trajectory graph, and order cells by pseudotime. The resulting pseudotime is the covariate for scTITANS' subsequent curve-fitting (complete model vs flat null, F statistic on residuals) and bootstrap q-value steps. Method choice matters: benchmarking on HSMM (GSE52529, 271 cells, two branches from Fluidigm C1) showed only Monocle and Monocle3 scored, while GPfates produced a much less reasonable trajectory and Mpath gave centroid-only trajectories; on MEF (GSE67310) only Monocle3 reconstructed the three branches. If no root cell type is known, pass 'none' and let the two-branch topology drive ordering. Validate the fitted graph against known biological branches and cell counts (e.g. discard cells outside the 5th–95th percentile of detected genes or with >10% mitochondrial UMIs beforehand).

## Related tools

- **Monocle3** (Constructs the trajectory, performs Louvain clustering and UMAP dimension reduction, and calculates pseudotime for each cell)
- **scTITANS** (Wrapper workflow that consumes the Monocle3 trajectory and pseudotime for time-dependent covariate DEG and subcluster testing) — https://github.com/ZJUFanLab/scTITANS
- **scran** (Pool-and-deconvolute normalization of the filtered count matrix prior to trajectory inference)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- The reconstructed branch count matches known biology (e.g. two branches for HSMM's 271 cells, three branches for MEF's 605 cells)
- Cells outside the 5th–95th percentile of detected genes or with >10% mitochondrial UMIs were removed before inference
- scTITANS_full call with fdr_level = 0.01 and no root cell type returns SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt without error from the cluster-identification step
- Pseudotime ordering is monotonic along the learned graph and consistent with sampling timepoints
- GPfates/Mpath alternatives, if tried, should be compared qualitatively — GPfates trajectories on HSMM were 'much less reasonable'

## Limitations

- Software version numbers for Monocle3, scran, and scTITANS are not stated in the article and no changelog exists for the repository
- Mpath provides trajectories only for cluster centroids, not single cells; GPfates trajectories may be qualitatively unreasonable on two-branch datasets like HSMM
- Without a known root cell type, ordering direction must be inferred from the graph ('root_type' = 'none')

## Evidence
