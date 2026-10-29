---
name: single-cell-trajectory-reconstruction
description: Use when scRNA-seq data span multiple stable and transitional cell states
  and the goal is to reconstruct trajectories between any two chosen subpopulations
  (e.g., HSPC-to-erythroid/GMP, B-cell/mono-DC-to-HSPC reprogramming) that linear
  or single-branch tools cannot resolve.
when_to_use_negative:
- The question only requires a simple linear pseudotime ordering of one continuous
  process, where tools like Monocle 1 or Waterfall suffice and CellRouter's subpopulation-aware
  flow network adds unnecessary complexity
- Trajectory structure is undefined and neither community detection nor prior annotations
  (cluster labels, sorted populations) can define source and target subpopulations
- The data are not single-cell (e.g., bulk RNA-seq or a precomputed feature table),
  since CellRouter requires single cells representing multiple cell states
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CellRouter
  role: 'Primary platform: learns data structure, identifies subpopulations, and reconstructs
    multi-state trajectories with a flow-network algorithm'
  repo: https://github.com/edroaldo/cellrouter
- name: Monocle 2
  role: Comparison method using default DDRTree branching trajectory; used to benchmark
    lineage assignment (e.g., megakaryocyte/erythrocyte bifurcation in GSE70245)
- name: Diffusion Pseudotime (DPT)
  role: Comparison method producing diffusion-map pseudotime orderings for smoothness
    benchmarking
- name: Wishbone
  role: Comparison method for trajectory inference in the smoothness benchmark
- name: Waterfall
  role: Comparison method for trajectory inference in the smoothness benchmark
- name: igraph
  role: Community detection used to identify subpopulations on the learned data structure
- name: mclust
  role: Model-based clustering combined with community detection for subpopulation
    identification
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:88f1988e9bd5e064c62129ec061e75730c0599f8d7f7782fcf7148d4e97004c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Diffusion Pseudotime (DPT)
  - Monocle 2
  - R
  - Rtsne
  - Waterfall
  - Wishbone
  - cccd
  - ggplot2
  - igraph
  - mclust
  - pheatmap
  - reshape
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# single-cell-trajectory-reconstruction

## Summary

Reconstruct high-resolution multi-state cell-state transition trajectories from single-cell RNA-seq data using CellRouter's flow-network algorithm, which is subpopulation-aware and requires no a priori knowledge of the number of branches or fates.

## When to use

Use when scRNA-seq data span multiple stable and transitional cell states and the goal is to reconstruct trajectories between any two chosen subpopulations (e.g., HSPC-to-erythroid/GMP, B-cell/mono-DC-to-HSPC reprogramming) that linear or single-branch tools cannot resolve.

## When NOT to use

- The question only requires a simple linear pseudotime ordering of one continuous process, where tools like Monocle 1 or Waterfall suffice and CellRouter's subpopulation-aware flow network adds unnecessary complexity
- Trajectory structure is undefined and neither community detection nor prior annotations (cluster labels, sorted populations) can define source and target subpopulations
- The data are not single-cell (e.g., bulk RNA-seq or a precomputed feature table), since CellRouter requires single cells representing multiple cell states

## Inputs

- Single-cell RNA-seq expression matrix (raw counts or UMI, e.g., GSE75478, GSE70245, GSE72857) in R
- Cell subpopulation annotations or cluster labels (optional; can be user-defined metadata such as sorted populations)
- Path to CellRouter Java libraries directory (libdir) and CellRouter_Class.R

## Outputs

- Subpopulation assignments from community detection (igraph/mclust)
- Multi-state trajectory/branch assignment per cell between chosen source and target subpopulations
- Trajectory-ranked gene lists, e.g., overexpression and knockdown reprogramming candidates (reprogramming_candidates.csv)
- Gene regulatory network of putative regulatory relationships
- Comparison tables/figures versus other algorithms, e.g., algorithm_comparison.csv, lineage_comparison.csv/png

## How to apply

Load the scRNA-seq expression matrix (raw counts/UMI are accepted in the May 2018 version) into R, source the CellRouter class with the 'libdir' variable pointing to the Java libraries inside the CellRouter folder, and install the required R packages (reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd, plus Vennerable). Optionally run quality control and normalization, then learn the structure of the data and identify subpopulations by combining community detection (igraph/mclust); user-defined cluster labels or sorted-population metadata may be supplied instead of de novo clustering. Reconstruct a gene regulatory network from the single cells to define putative regulatory relationships, then run CellRouter's flow-network trajectory algorithm from a chosen source subpopulation to any target subpopulation (e.g., subpopulation 20/CMP to subpopulations 13/erythroid and 9/GMP in the Paul et al. GSE72857 myeloid data). Score genes and regulators along each trajectory to identify transition-specific genes and overexpression/knockdown reprogramming candidates, and compare branch assignments against alternative tools such as Monocle 2, DPT, Wishbone, or Waterfall when benchmarking.

## Related tools

- **CellRouter** (Primary platform: learns data structure, identifies subpopulations, and reconstructs multi-state trajectories with a flow-network algorithm) — https://github.com/edroaldo/cellrouter
- **Monocle 2** (Comparison method using default DDRTree branching trajectory; used to benchmark lineage assignment (e.g., megakaryocyte/erythrocyte bifurcation in GSE70245))
- **Diffusion Pseudotime (DPT)** (Comparison method producing diffusion-map pseudotime orderings for smoothness benchmarking)
- **Wishbone** (Comparison method for trajectory inference in the smoothness benchmark)
- **Waterfall** (Comparison method for trajectory inference in the smoothness benchmark)
- **igraph** (Community detection used to identify subpopulations on the learned data structure)
- **mclust** (Model-based clustering combined with community detection for subpopulation identification)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- reprogramming_candidates.csv exists and contains both overexpression and knockdown ranked lists, recovering 11 of the 67 published reprogramming factors among overexpression candidates
- Expected knockdown candidates appear in the correct lineage lists: DNMT1/H3F3A/JARID2/KMT2D for B-cell-to-HSPC and DNMT1/H2AFZ/TRIM28/KMT2D/H2AFV for mono/DC-to-HSPC
- algorithm_comparison.csv contains a row for each of the six algorithm variants (CellRouter, Monocle 2, DPT, Wishbone, Waterfall, CellRouter-with-Paul-labels) with significantly correlated gene counts and lag-1 autocorrelations populated, and CellRouter shows more significant correlations and higher lag-1 autocorrelation of branch markers (Klf1, Car2, Cited4; Mpo, Prtn3, Ctsg)
- CellRouter resolves distinct trajectories to subpopulations 19 (megakaryocyte) and 5 (erythrocyte) on GSE70245 whereas Monocle 2 places both lineages on a single branch in lineage_comparison.csv
- Correct subpopulations are confirmed before trajectory construction (e.g., subpopulation 19 megakaryocyte, subpopulation 5 erythrocyte, subpopulation 20 CMP, subpopulations 13 erythroid and 9 GMP)

## Limitations

- CellRouter was tested only on Unix-based systems (strongly recommended Unix or Mac; not tested on Windows)
- Current trajectory algorithms, including those compared here, are less robust in reconstructing trajectories from early states towards intermediate or transitory cell states, so benchmarking against them is advisable
- Functional characterization of reprogramming candidate genes (e.g., for generating engraftable HSC-like cells) extends beyond the published scope and requires separate validation

## Evidence

- [abstract] CellRouter, a multifaceted single-cell analysis platform that identifies complex cell-state transition trajectories by using flow networks: "Here, we present CellRouter, a multifaceted single-cell analysis platform that identifies complex cell-state transition trajectories by using flow networks"
- [intro] CellRouter requires no a priori knowledge of trajectory structure, such as number of cell fates or branches: "CellRouter requires no a priori knowledge of trajectory structure, such as number of cell fates or branches"
- [methods] Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [readme] source the CellRouter class and make sure you provide the correct directory for the Java libraries: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [readme] Raw count/UMI data can be provided as input.: "1. Raw count/UMI data can be provided as input."
- [methods] High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
