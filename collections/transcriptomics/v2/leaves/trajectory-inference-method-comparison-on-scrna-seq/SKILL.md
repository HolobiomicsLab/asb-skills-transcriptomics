---
name: trajectory-inference-method-comparison-on-scrna-seq
description: Decision support for selecting a TI method when a scRNA-seq dataset has
  a known or published branch/cell-type structure and the goal is to determine which
  of several TI methods (e.g., Monocle3 vs Mpath) reconstructs the true trajectory
  before downstream pseudotime analysis.
when_to_use_negative:
- The dataset has no known or reference branch structure against which reconstructed
  trajectories can be validated, so method comparison reduces to subjective judgment.
- Downstream analysis only needs any trajectory and pseudotime already computed —
  e.g., when running scTITANS in 'partial' mode on existing trajectory inference results,
  method benchmarking is unnecessary.
- The input is already a per-cell pseudotime or trajectory object rather than a raw/filtered
  UMI matrix suitable for re-running TI methods.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: Monocle
  role: TI method benchmarked on the normalized CMP matrix
- name: Monocle2
  role: TI method benchmarked on the normalized CMP matrix
- name: Monocle3
  role: TI method benchmarked; also the method used by scTITANS to construct trajectories
    and calculate per-cell pseudotime
  repo: https://github.com/ZJUFanLab/scTITANS
- name: TSCAN
  role: TI method benchmarked on the normalized CMP matrix
- name: SLICER
  role: TI method benchmarked on the normalized CMP matrix
- name: GPfates
  role: TI method benchmarked on the normalized CMP matrix
- name: Slingshot
  role: TI method benchmarked on the normalized CMP matrix
- name: Destiny
  role: TI method benchmarked on the normalized CMP matrix
- name: Mpath
  role: TI method benchmarked; accurately reconstructed the eight CMP branches but
    does not provide a per-cell trajectory, reducing its value for correcting single-cell
    asynchrony
- name: STEMNET
  role: TI method benchmarked on the normalized CMP matrix
- name: scran
  role: Normalization of the filtered UMI matrix via pool-and-deconvolute before applying
    each TI method
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:b0b95022a24701840db9bb5e2e8d5e49641dfe34e7ab55cfd2d204a5aaa90cca
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
  - Destiny
  - GPfates
  - Monocle
  - Monocle2
  - Monocle3
  - Mpath
  - SLICER
  - STEMNET
  - Slingshot
  - TSCAN
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
yes_rate: 0.6
n_claims: 10
---

# trajectory-inference-method-comparison-on-scrna-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple trajectory inference (TI) methods (Monocle, Monocle2, Monocle3, TSCAN, SLICER, GPfates, Slingshot, Destiny, Mpath, STEMNET) on a scRNA-seq dataset with known branch structure, comparing the number and topology of reconstructed branches to select the method that best matches the reference trajectory — as done for the CMP dataset (GSE72857) in the scTITANS study.

## When to use

Decision support for selecting a TI method when a scRNA-seq dataset has a known or published branch/cell-type structure and the goal is to determine which of several TI methods (e.g., Monocle3 vs Mpath) reconstructs the true trajectory before downstream pseudotime analysis.

## When NOT to use

- The dataset has no known or reference branch structure against which reconstructed trajectories can be validated, so method comparison reduces to subjective judgment.
- Downstream analysis only needs any trajectory and pseudotime already computed — e.g., when running scTITANS in 'partial' mode on existing trajectory inference results, method benchmarking is unnecessary.
- The input is already a per-cell pseudotime or trajectory object rather than a raw/filtered UMI matrix suitable for re-running TI methods.

## Inputs

- Raw scRNA-seq UMI count matrix (genes x cells) with GEO accession (e.g., GSE72857 CMP dataset: 2730 cells, 19 cell types, eight branches)
- Reference branch structure and known cell-type ordering for the dataset
- Installed TI methods: Monocle, Monocle2, Monocle3, TSCAN, SLICER, GPfates, Slingshot, Destiny, Mpath, STEMNET; scran for normalization

## Outputs

- Per-method reconstructed trajectories over the same normalized matrix
- Comparison table of per-method branch-reconstruction success/failure (e.g., cmp_ti_comparison.csv)
- Selected TI method (e.g., Monocle3 for CMP) with rationale, for downstream pseudotime-based analysis such as scTITANS

## How to apply

Download the dataset with known trajectory structure (e.g., CMP GSE72857: 2730 mouse myeloid progenitor cells, 19 cell types, eight branches) from NCBI GEO. Preprocess the raw UMI matrix: filter genes detected (two or more transcripts) in at least 10 cells, discard cells with gene counts outside the 5th and 95th percentile or >10% mitochondrial UMIs, then normalize with the scran pool-and-deconvolute algorithm. Apply each candidate TI method to the same normalized matrix to reconstruct its trajectory. For each method, compare the number and topology of reconstructed branches against the known branch count and cell-type ordering. Record per-method success/failure in a comparison table (e.g., cmp_ti_comparison.csv), noting that a method that reconstructs all branches but lacks per-cell pseudotime (like Mpath on CMP) may be excluded if single-cell pseudotime is required downstream.

## Related tools

- **Monocle** (TI method benchmarked on the normalized CMP matrix)
- **Monocle2** (TI method benchmarked on the normalized CMP matrix)
- **Monocle3** (TI method benchmarked; also the method used by scTITANS to construct trajectories and calculate per-cell pseudotime) — https://github.com/ZJUFanLab/scTITANS
- **TSCAN** (TI method benchmarked on the normalized CMP matrix)
- **SLICER** (TI method benchmarked on the normalized CMP matrix)
- **GPfates** (TI method benchmarked on the normalized CMP matrix)
- **Slingshot** (TI method benchmarked on the normalized CMP matrix)
- **Destiny** (TI method benchmarked on the normalized CMP matrix)
- **Mpath** (TI method benchmarked; accurately reconstructed the eight CMP branches but does not provide a per-cell trajectory, reducing its value for correcting single-cell asynchrony)
- **STEMNET** (TI method benchmarked on the normalized CMP matrix)
- **scran** (Normalization of the filtered UMI matrix via pool-and-deconvolute before applying each TI method)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- The comparison table must show that Mpath is the method that accurately reconstructs the eight CMP branches (consistent with Figure S1c of the paper), while other methods fail to recover the full branch structure.
- The same scran-normalized matrix is fed identically to all ten TI methods, so differences in output are attributable to the TI method, not preprocessing.
- Cell filtering is applied consistently: genes detected (>=2 transcripts) in at least 10 cells kept; cells with gene counts outside the 5th–95th percentile or >10% mitochondrial UMIs discarded.
- The selected method must provide a trajectory/pseudotime for each single cell if the goal is correcting asynchrony for time-series analysis (this is why Monocle3 scored for CMP despite Mpath's better branch reconstruction).
- Sanity check across datasets: only Monocle, Monocle3, GPfates, and Mpath should reconstruct the two HSMM (GSE52529) branches, and only Monocle3 should reconstruct the three MEF (GSE67310) branches.

## Limitations

- Mpath accurately reconstructed the eight CMP branches but does not provide a trajectory for each single cell, which reduces its value for correcting single-cell asynchrony in subsequent time-series analysis.
- GPfates trajectories can be 'apparently much less reasonable' even when the correct number of branches is recovered (HSMM), so branch-count agreement alone is insufficient — topology must also be compared.
- Software version numbers for tools (e.g., Monocle3, scran, scTITANS) are not stated in the article, and no changelog is available for the scTITANS repository, hindering exact reproduction.

## Evidence

- [other] Although Mpath best reconstructed the CMP branches, Mpath did not provide a trajectory for each single cell, which reduces its value in correcting the asynchrony of single cells: [evidence span withheld — anchor status: not_found]
