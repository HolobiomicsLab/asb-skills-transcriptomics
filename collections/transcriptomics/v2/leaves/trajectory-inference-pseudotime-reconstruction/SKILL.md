---
name: trajectory-inference-pseudotime-reconstruction
description: Use when analyzing time-series scRNA-seq count matrices where cell asynchrony
  must be corrected via trajectory inference and pseudotime to identify genes and
  subclusters that vary over time.
when_to_use_negative:
- Data is not time-series single-cell RNA-seq or lacks a biological ordering along
  which pseudotime is meaningful (e.g., a static case-control bulk comparison such
  as the 'edge' baseline)
- You already have a validated trajectory and pseudotime from another tool and only
  need downstream testing — scTITANS_partial, or the article notes partial mode is
  'much more suitable', is preferable to rerunning full trajectory inference
- Cell cluster information needed for the partial mode's cell_metadata is missing
  and you cannot run full mode's annotation (which requires sp = Human or Mouse and
  a tissue/organ name)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTITANS
  role: Main pipeline that runs trajectory inference and pseudotime-based DEG/subcluster
    identification via scTITANS_full or scTITANS_partial
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Constructs the trajectory graph and calculates pseudotime for each cell inside
    scTITANS
- name: scran
  role: Normalizes the filtered count matrix with the pool-and-deconvolute algorithm
    prior to trajectory inference
- name: scCATCH
  role: Obtains cell type annotations when annotation is requested in full mode (sp
    = Human or Mouse plus tissue)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:7fe7d4aa581de3bf47da011007c46ed409fe3925087cffdd5f6fa12780741632
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# trajectory-inference-pseudotime-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs single-cell trajectories and pseudotime using Monocle3 within the scTITANS pipeline, correcting cell asynchrony in time-series scRNA-seq data before DEG and subcluster identification.

## When to use

Use when analyzing time-series scRNA-seq count matrices where cell asynchrony must be corrected via trajectory inference and pseudotime to identify genes and subclusters that vary over time.

## When NOT to use

- Data is not time-series single-cell RNA-seq or lacks a biological ordering along which pseudotime is meaningful (e.g., a static case-control bulk comparison such as the 'edge' baseline)
- You already have a validated trajectory and pseudotime from another tool and only need downstream testing — scTITANS_partial, or the article notes partial mode is 'much more suitable', is preferable to rerunning full trajectory inference
- Cell cluster information needed for the partial mode's cell_metadata is missing and you cannot run full mode's annotation (which requires sp = Human or Mouse and a tissue/organ name)

## Inputs

- Gene expression matrix file (rows = genes, columns = cells) for time-series scRNA-seq data
- Optional cell metadata file (rows = cells, with cluster/cell type info for partial mode)
- Optional gene metadata file (rows = genes, header must contain gene_short_name)

## Outputs

- SigGenes.fdr0.01.txt — significant DEGs at the chosen fdr level
- SigClusters.fdr0.01.txt — significant cell subclusters at the chosen fdr level
- Pseudotime-ordered trajectory inferred by Monocle3

## How to apply

Preprocess the raw U count matrix first: filter genes detected (>=2 transcripts) in at least 10 cells, discard cells with gene counts outside the 5th-95th percentile and cells with >10% mitochondrial UMIs, then normalize with the scran pool-and-deconvolute algorithm. Run scTITANS in full mode (scTITANS_full) when no prior trajectory exists, supplying the expression matrix, indication ('none', 'filtered', or 'normalized'), annotation flag, optional root cell type, and an fdr level such as 0.01; use scTITANS_partial when trajectory inference is already done. scTITANS delegates trajectory construction and pseudotime calculation to Monocle3, then introduces a time-dependent covariate to evaluate gene and cell-number fluctuations along pseudotime and fits a curve for each gene's dynamic expression. q-values are computed from a bootstrap null distribution, and significant genes are returned in SigGenes.fdr0.01.txt (with SigClusters.fdr0.01.txt for subclusters). Validate by checking the output files exist at the requested fdr level and, where possible, comparing DEG rankings/overlap against supplementary reference tables (e.g., Table S3/S4).

## Related tools

- **scTITANS** (Main pipeline that runs trajectory inference and pseudotime-based DEG/subcluster identification via scTITANS_full or scTITANS_partial) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Constructs the trajectory graph and calculates pseudotime for each cell inside scTITANS)
- **scran** (Normalizes the filtered count matrix with the pool-and-deconvolute algorithm prior to trajectory inference)
- **scCATCH** (Obtains cell type annotations when annotation is requested in full mode (sp = Human or Mouse plus tissue))

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- The run returns two output files named SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt at the requested fdr level; an error message is displayed if significant cluster identification fails
- Gene filtering invariant: only genes present with >=2 transcripts in at least 10 cells are retained as 'detected'
- Cell filtering invariant: no retained cells fall outside the 5th-95th percentile of detected gene counts, and no retained cell has >10% of UMIs assigned to mitochondrial genes
- Normalized matrix was produced with scran's default pool-and-deconvolute algorithm before trajectory construction
- For benchmarking tasks, computed DEG overlap percentages (e.g., top-20 scTITANS vs baseline lists in gene_overlap_top20.csv) match values reported in supplementary Tables S3/S4

## Limitations

- Not all trajectory inference methods succeed on every dataset — in the article's benchmark, only Monocle3 reconstructed the three branches in MEF, and only Mpath handled the eight branches in CMP, so trajectory method choice is dataset-dependent
- Significant cluster identification can fail, in which case scTITANS displays an error message rather than producing SigClusters
- Software version numbers for Monocle3, scran, and scTITANS are not stated, complicating exact reproduction
- In the AD (GSE98969) benchmark, scTITANS' advantage over 'edge' (80% vs 40% literature-verified top-20 DEGs) was not statistically significant (p = 0.30) with low power (0.60), suggesting the p-value may be unreliable

## Evidence
