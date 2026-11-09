---
name: time-series-scrnaseq-differential-expression-analysis
description: Use when raw UMI count matrices from time-course scRNA-seq experiments
  (e.g., GSE52529 HSMM, GSE67310 MEF, GSE126954 CEED) need DEG and differential subcluster
  identification along pseudotime rather than per-timepoint comparisons.
when_to_use_negative:
- The data are not time-series scRNA-seq (no underlying temporal/biological progression
  to order along pseudotime) — trajectory-based DEG identification is not the right
  scope.
- Trajectory inference and preprocessing have already been completed by another pipeline
  with no need to redo TI — use the partial mode only with cell_metadata and gene_metadata
  files, not scTITANS_full on raw data.
- Only cluster-centroid trajectories are needed — the article notes Mpath provided
  trajectories only for cluster centroids rather than single cells, which is a different
  analysis goal.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTITANS
  role: 'core method: corrects cell asynchrony via pseudotime and identifies time-varying
    DEGs and cell subclusters via time-dependent covariate analysis; run as scTITANS_full
    or scTITANS_partial'
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: constructs the trajectory (Louvain clustering, UMAP, trajectory graph) and
    calculates pseudotime for each cell inside scTITANS
- name: scran
  role: normalizes the filtered matrix using the default pool-and-deconvolute normalization
    algorithm before TI analysis
- name: scCATCH
  role: obtains cell type annotations when cell annotation is requested (annotation=T
    in scTITANS_full)
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:249a576b109d2702839cc47a331caf28fade833bf7342e20a2a2e3e7c674a013
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
  - scCATCH
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# time-series-scrnaseq-differential-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify genes and cell subclusters that vary over time in time-series single-cell RNA-seq data by correcting cell asynchrony: construct a trajectory and pseudotime (Monocle3) and apply a time-dependent covariate analysis to gene expression and cell subcluster counts along pseudotime (scTITANS).

## When to use

Use when raw UMI count matrices from time-course scRNA-seq experiments (e.g., GSE52529 HSMM, GSE67310 MEF, GSE126954 CEED) need DEG and differential subcluster identification along pseudotime rather than per-timepoint comparisons.

## When NOT to use

- The data are not time-series scRNA-seq (no underlying temporal/biological progression to order along pseudotime) — trajectory-based DEG identification is not the right scope.
- Trajectory inference and preprocessing have already been completed by another pipeline with no need to redo TI — use the partial mode only with cell_metadata and gene_metadata files, not scTITANS_full on raw data.
- Only cluster-centroid trajectories are needed — the article notes Mpath provided trajectories only for cluster centroids rather than single cells, which is a different analysis goal.

## Inputs

- raw gene-by-cell UMI count matrix (gene expression matrix, rows = genes, columns = cells), e.g. from Cell Ranger (10X Genomics) output
- optional cell metadata file (rows = cells, with cluster/cell type info for partial mode)
- optional gene metadata file (rows = genes, header must contain gene_short_name, for partial mode)

## Outputs

- SigGenes.fdr0.01.txt — significant DEGs at the chosen fdr level (e.g. fdr level 0.01)
- SigClusters.fdr0.01.txt — significant cell subclusters varying along pseudotime
- constructed trajectory and per-cell pseudotime (from Monocle3)

## How to apply

Preprocess the raw U count matrix in scTITANS: filter genes with zero counts, keep genes detected (>=2 transcripts) in at least 10 cells, discard cells whose detected-gene counts fall outside the 5th-95th percentile or with >10% mitochondrial UMIs, then normalize with the scran pool-and-deconvolute algorithm. Run trajectory inference with Monocle3 (Louvain clustering, UMAP, trajectory graph learning, pseudotime ordering); for the 271-cell two-branch HSMM dataset Monocle3 was one of only two methods that scored. For each gene fit a curve of dynamic expression along pseudotime (complete model) versus a flat null model and compute the F statistic on residuals, deriving the null distribution by bootstrapping residuals resampled from the alternative fit; convert p-values to q-values and apply the same time-dependent covariate analysis to cell subcluster counts across pseudotime bins. Run in 'full' mode (scTITANS_full) with no root cell type and fdr_level=0.01 for raw matrices, or 'partial' mode (scTITANS_partial) when preprocessing and TI are already done; both modes return SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt.

## Related tools

- **scTITANS** (core method: corrects cell asynchrony via pseudotime and identifies time-varying DEGs and cell subclusters via time-dependent covariate analysis; run as scTITANS_full or scTITANS_partial) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (constructs the trajectory (Louvain clustering, UMAP, trajectory graph) and calculates pseudotime for each cell inside scTITANS)
- **scran** (normalizes the filtered matrix using the default pool-and-deconvolute normalization algorithm before TI analysis)
- **scCATCH** (obtains cell type annotations when cell annotation is requested (annotation=T in scTITANS_full))

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- Running scTITANS_full with fdr_level=0.01 and root_type='none' must return both SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt without error messages from the cluster-identification step.
- On the HSMM dataset (GSE52529, 271 cells), Monocle3's reconstructed trajectory must show the two known branches; on MEF (GSE67310) it must show the three branches.
- Preprocessing invariants: genes present with two or more transcripts in at least 10 cells retained; cells outside the 5th-95th percentile of detected genes or with >10% mitochondrial UMIs removed; matrix normalized by scran.
- For the AD (GSE98969) benchmark, top-20 DEG lists can be cross-checked against literature-verified genes and bulk RNA-seq baselines (e.g. edge: 80% vs 40% overlap, Fisher exact p=0.30).
- For the CEED (GSE126954) validation, GO biological-process enrichment of the top 20 DEGs (e.g. via ToppGene) should support the literature-verified functions (11 of top 12 genes confirmed).

## Limitations

- Trajectory inference method choice is dataset-dependent: on HSMM only Monocle and Monocle3 scored (GPfates' trajectory was much less reasonable, Mpath gave cluster-centroid trajectories only); on MEF only Monocle3 reconstructed the three branches; on CMP only Mpath accurately reconstructed the eight branches.
- Software version numbers for scTITANS, Monocle3 and scran are not stated in the article and no changelog is available for the repository.
- If something wrong happens in identifying significant clusters, an error information is displayed and the SigClusters output may be missing.

## Evidence
