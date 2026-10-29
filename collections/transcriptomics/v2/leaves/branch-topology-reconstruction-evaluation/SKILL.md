---
name: branch-topology-reconstruction-evaluation
description: Decision support for selecting a trajectory inference method when a scRNA-seq
  dataset has known branch structure (e.g., 8 branches in the CMP myeloid progenitor
  data) and downstream time-series analysis needs single-cell pseudotime.
when_to_use_negative:
- The dataset has no known reference topology or validated cell-type ordering against
  which reconstructed branches can be compared
- You only need branch topology with no downstream need for per-cell pseudotime (in
  that case Mpath may suffice, unlike scTITANS's asynchrony-correction use case)
- You already have a completed trajectory and only need DEG/subcluster identification
  — use scTITANS 'partial' mode instead of re-running the method comparison
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Monocle3
  role: Performs the trajectory construction and per-cell pseudotime calculation;
    the method that scored for CMP because it yields a trajectory for each single
    cell
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Mpath
  role: Benchmarked TI method; only method that accurately reconstructed the eight
    CMP branches but did not provide per-single-cell trajectories
- name: Monocle
  role: Benchmarked TI method in the ten-method comparison
- name: Monocle2
  role: Benchmarked TI method in the ten-method comparison
- name: TSCAN
  role: Benchmarked TI method in the ten-method comparison
- name: SLICER
  role: Benchmarked TI method in the ten-method comparison
- name: GPfates
  role: Benchmarked TI method in the ten-method comparison
- name: Slingshot
  role: Benchmarked TI method in the ten-method comparison
- name: Destiny
  role: Benchmarked TI method in the ten-method comparison
- name: STEMNET
  role: Benchmarked TI method in the ten-method comparison
- name: scran
  role: Normalizes the raw UMI matrix using the pool-and-deconvolute algorithm before
    TI method comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:675c5c5f46291b2e2a3ef083740561b503f928d862a50d9e60ee0b4554a9edff
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# branch-topology-reconstruction-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematic comparison of trajectory inference (TI) methods on a dataset with known branching structure, evaluating how many and which branches each method reconstructs against known cell-type ordering, and selecting the method that both recovers the topology AND yields per-cell pseudotime for downstream asynchrony correction.

## When to use

Decision support for selecting a trajectory inference method when a scRNA-seq dataset has known branch structure (e.g., 8 branches in the CMP myeloid progenitor data) and downstream time-series analysis needs single-cell pseudotime.

## When NOT to use

- The dataset has no known reference topology or validated cell-type ordering against which reconstructed branches can be compared
- You only need branch topology with no downstream need for per-cell pseudotime (in that case Mpath may suffice, unlike scTITANS's asynchrony-correction use case)
- You already have a completed trajectory and only need DEG/subcluster identification — use scTITANS 'partial' mode instead of re-running the method comparison

## Inputs

- Raw UMI gene expression matrix (rows=genes, columns=cells), e.g., from GEO accession GSE72857 (MARS-seq, FACS-sorted mouse myeloid progenitors)
- Known branch topology and cell-type ordering for the dataset (19 cell types, eight branches for CMP)
- List of candidate TI methods (Monocle, Monocle2, Monocle3, TSCAN, SLICER, GPfates, Slingshot, Destiny, Mpath, STEMNET)
- scran-normalized expression matrix (pool-and-deconvolute)

## Outputs

- Per-method branch reconstruction results (number and topology of reconstructed branches vs. ground truth)
- Comparison table of per-method success/failure (e.g., cmp_ti_comparison.csv)
- Selected TI method with justification (e.g., Monocle3 for CMP, Mpath accurate but lacking per-cell trajectories)

## How to apply

Obtain a dataset with known topology (e.g., CMP GSE72857: 2730 mouse myeloid progenitor cells, 19 cell types, eight branches) and preprocess the raw UMI matrix: keep genes detected with two or more transcripts in at least 10 cells, discard cells with gene counts outside the 5th-95th percentile or >10% mitochondrial UMIs, and normalize with scran's pool-and-deconvolute algorithm. Apply each candidate TI method (Monocle, Monocle2, Monocle3, TSCAN, SLICER, GPfates, Slingshot, Destiny, Mpath, STEMNET in the article) to the normalized matrix. For each method, compare the number and topology of reconstructed branches against the known branch count and cell-type ordering, and record per-method success/failure in a comparison table (e.g., cmp_ti_comparison.csv). The decision point is that topological accuracy alone is insufficient: a method like Mpath that best reconstructs branches but does not assign a trajectory/pseudotime to each single cell cannot correct cell asynchrony, so Monocle3 scored for CMP because it constructs the trajectory and calculates pseudotime per cell.

## Related tools

- **Monocle3** (Performs the trajectory construction and per-cell pseudotime calculation; the method that scored for CMP because it yields a trajectory for each single cell) — https://github.com/ZJUFanLab/scTITANS
- **Mpath** (Benchmarked TI method; only method that accurately reconstructed the eight CMP branches but did not provide per-single-cell trajectories)
- **Monocle** (Benchmarked TI method in the ten-method comparison)
- **Monocle2** (Benchmarked TI method in the ten-method comparison)
- **TSCAN** (Benchmarked TI method in the ten-method comparison)
- **SLICER** (Benchmarked TI method in the ten-method comparison)
- **GPfates** (Benchmarked TI method in the ten-method comparison)
- **Slingshot** (Benchmarked TI method in the ten-method comparison)
- **Destiny** (Benchmarked TI method in the ten-method comparison)
- **STEMNET** (Benchmarked TI method in the ten-method comparison)
- **scran** (Normalizes the raw UMI matrix using the pool-and-deconvolute algorithm before TI method comparison)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- The comparison table must show Mpath as the method that accurately reconstructs the eight CMP branches (consistent with Figure S1c of the paper), while other methods fail to recover the full branch structure
- Each method's reconstructed branch count must be compared against the ground-truth count (e.g., 2 branches for HSMM, 3 for MEF, 8 for CMP)
- The selected method must provide a pseudotime value for each single cell, not just cluster-level trajectories — verify Monocle3 outputs per-cell pseudotime
- Preprocessing invariants hold: genes detected with ≥2 transcripts in ≥10 cells; cells with gene counts within the 5th-95th percentile and ≤10% mitochondrial UMIs; scran pool-and-deconvolute normalization applied
- Benchmark datasets (HSMM GSE52529, MEF GSE67310, CMP GSE72857) yield expected per-method outcomes, e.g., only Monocle3 reconstructed the three MEF branches

## Limitations

- Mpath accurately reconstructs branch topology but does not provide a trajectory for each single cell, limiting its value for correcting single-cell asynchrony in time-series analysis
- Software version numbers for the TI tools (e.g., Monocle3, scran, scTITANS) are not stated in the article
- The benchmark's branch-count criteria are specific to the tested datasets (HSMM 2 branches, MEF 3 branches, CMP 8 branches); results may not transfer to datasets with different topology or different normalization

## Evidence

- [results] Only Mpath accurately reconstructed the eight branches implicated in the database and also the trajectory path: "Only Mpath accurately reconstructed the eight branches implicated in the database and also the trajectory path (Figure S1c)"
- [results] Mpath lacks per-cell trajectories, so Monocle3 scored for CMP: "scTITANS utilizes Monocle3 [13], [35] to construct the trajectory and calculate the pseudotime for each cell."
- [methods] Ten benchmarked TI methods listed: "The selected 10 methods include Monocle[13], Monocle2, Monocle3, TSCAN [26], SLICER [27], GPfates [28], Slingshot [29], Destiny [30], Mpath [31], and STEMNET [32]."
