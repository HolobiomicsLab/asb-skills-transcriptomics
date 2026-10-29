---
name: dentate-gyrus-granule-cell-trajectory-analysis
description: Use when applying CellPath to the dentate-gyrus dataset (GSE95753, P12/P35,
  ~2,930 cells) to extract multiple high-resolution granule-cell trajectories with
  RNA velocity and quantify DE genes along within-population paths.
when_to_use_negative:
- The dataset is not the dentate-gyrus neurogenesis data or lacks spliced/unspliced
  layers needed for scVelo velocity estimation
- The goal is a single fixed-topology trajectory rather than multiple high-resolution
  paths including within-population sub-flows
- Velocity should not be estimated (e.g., no velocity desired); CellPath's paths depend
  on RNA velocity, and its performance is limited by velocity inference accuracy
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellPath
  role: Infers the 14 high-resolution granule-cell trajectories via meta-cell construction
    (k-means), k-nn graph, greedy path selection, and cell insertion
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the RNA velocity matrix at CellPath initialization when preprocess
    is False
- name: statsmodels
  role: Performs differentially expressed gene analysis on path-6 cells, producing
    the table with Camk2a, Adcy1, and Tmsb10
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:10f4cd8cf0e396afa75683399dd1f9a56417261d0e75730b80996b85b0dc0ba1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PeterZZQ/CellPath
  repo_url: https://github.com/PeterZZQ/CellPaths.git
  tools:
  - CellPath
  - scVelo
  - statsmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dentate-gyrus-granule-cell-trajectory-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce CellPath-based trajectory inference on the dentate gyrus neurogenesis dataset (GEO:GSE95753), recovering both the mainstream granule-cell differentiation trajectories and subtle within-population sub-flows such as path 6, then verify maturation dynamics via DE analysis of Camk2a and Adcy1.

## When to use

Use when applying CellPath to the dentate-gyrus dataset (GSE95753, P12/P35, ~2,930 cells) to extract multiple high-resolution granule-cell trajectories with RNA velocity and quantify DE genes along within-population paths.

## When NOT to use

- The dataset is not the dentate-gyrus neurogenesis data or lacks spliced/unspliced layers needed for scVelo velocity estimation
- The goal is a single fixed-topology trajectory rather than multiple high-resolution paths including within-population sub-flows
- Velocity should not be estimated (e.g., no velocity desired); CellPath's paths depend on RNA velocity, and its performance is limited by velocity inference accuracy

## Inputs

- Dentate-gyrus scRNA-seq count matrix (GEO:GSE95753, P12/P35 sample, 2,930 cells) as an AnnData (.h5ad) object
- RNA velocity computed during CellPath initialization via scVelo (preprocess flag)

## Outputs

- cellpath_obj.pseudo_order: pseudotime and branching/path assignment for 14 trajectories, including granule-cell path 0 and within-population path 6
- path6_de_genes.csv: DE gene table for path-6 cells containing rows for Camk2a, Adcy1, and Tmsb10

## How to apply

Load the dentate-gyrus count matrix (GEO:GSE95753, P12/P35 sample, 2,930 cells) into an AnnData object and initialize CellPath with preprocess set so scVelo computes RNA velocity at initialization. Construct meta-cells with k-means clustering by passing flavor = "k-means" in cellpath.meta_cell_construction(), then run cellpath_obj.all_in_one with n_neighs = 10, pruning = False, num_trajs = 14, insertion = True, and prop_insert = 0.50 to build the k-nn meta-cell graph, extract candidate paths, greedily select 14 trajectories, and insert unassigned cells. Retrieve pseudotime and branching assignments from cellpath_obj.pseudo_order and verify the 14 output paths include the granule-cell differentiation path 0 and the within-population path 6. Finally, run differentially expressed gene analysis on path-6 cells with statsmodels and confirm that maturation genes Camk2a and Adcy1 increase along the path, saving the table (also containing Tmsb10) as path6_de_genes.csv.

## Related tools

- **CellPath** (Infers the 14 high-resolution granule-cell trajectories via meta-cell construction (k-means), k-nn graph, greedy path selection, and cell insertion) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the RNA velocity matrix at CellPath initialization when preprocess is False)
- **statsmodels** (Performs differentially expressed gene analysis on path-6 cells, producing the table with Camk2a, Adcy1, and Tmsb10)

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = False); cellpath_obj.meta_cell_construction(flavor="k-means"); cellpath_obj.all_in_one(n_neighs = 10, pruning = False, num_trajs = 14, insertion = True, prop_insert = 0.50)
```

## Evaluation signals

- cellpath_obj.pseudo_order reports exactly 14 output paths as specified by num_trajs = 14
- The 14 paths include both the mainstream granule-cell differentiation path 0 and the within-population path 6
- path6_de_genes.csv exists, parses as a valid CSV table, and contains rows for Camk2a, Adcy1, and Tmsb10
- Camk2a and Adcy1 expression increases along path 6, consistent with a maturation dynamic inside mature granule cells
- Meta-cell construction used flavor = "k-means" and all_in_one used n_neighs = 10, pruning = False, insertion = True, prop_insert = 0.50

## Limitations

- Performance of RNA velocity-based methods including CellPath is limited by the accuracy of RNA velocity inference; challenges in velocity estimation remain unresolved
- CellPath detects both global and local paths but no global view alignment of locally detected paths is implemented yet
- Connecting points between paths have not been implemented, so relationships among the 14 detected paths are not resolved

## Evidence

- [methods] k-means meta-cell construction: "You can specify the clustering algorithm you use with either `flavor = "leiden"` or `flavor = "k-means"` in `cellpath.meta_cell_construction()`"
- [methods] all_in_one parameters: "cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, num_trajs = num_trajs, insertion = True, prop_insert = 0.50)"
- [readme] velocity computed at initialization: "`preprocessing`: the velocity has been calculated and stored in adata or not, if False, the velocity will be calculated during initialization with scvelo"
- [discussion] velocity-based TI accuracy: "Given the noise in the estimated RNA velocity, the performance of all RNA velocity-based methods are affected by accuracy in RNA velocity inference."
