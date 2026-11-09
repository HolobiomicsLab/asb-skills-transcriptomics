---
name: single-cell-benchmark-evaluation
description: Use when inferred trajectories (e.g. MST-based trajectories from clustered
  single-cell data) must be scored against known ground-truth topologies from the
  dynverse Zenodo benchmark deposit (10.5281/zenodo.1443566).
when_to_use_negative:
- The data have no ground-truth trajectory (real datasets without a reference) — dynverse
  metrics require a known benchmark topology
- The trajectory of interest is disconnected or contains cycles, which Totem cannot
  generate and the tree-based benchmark scoring assumes
- Multi-root (converging) trajectories where dynplot cannot generate correct pseudotime
  without manual work-around
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Totem
  role: R package that generates the CLARA clusterings and MST trajectory candidates
    being benchmarked
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: constructs the Minimum Spanning Tree trajectory backbone from the selected
    clustering before scoring
- name: SingleCellExperiment
  role: container format for the benchmark datasets fed into PrepareTotem
- name: dyndimred
  role: provides LMDS 5-dimensional embedding used to prepare the data prior to clustering
    and scoring
- name: Seurat
  role: LogNormalize normalization of benchmark datasets before trajectory inference
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:0bfda510eddee56906263e0dcf055f0267158cf96533d6a6f4b99377b8c0f16c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - Seurat
  - SingleCellExperiment
  - Slingshot
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-benchmark-evaluation

## Summary

Evaluate inferred single-cell trajectories against dynverse benchmark ground truth using the dynverse metric suite, recording overall scores in [0, 1] per dataset and configuration for reproducible method benchmarking.

## When to use

Use when inferred trajectories (e.g. MST-based trajectories from clustered single-cell data) must be scored against known ground-truth topologies from the dynverse Zenodo benchmark deposit (10.5281/zenodo.1443566).

## When NOT to use

- The data have no ground-truth trajectory (real datasets without a reference) — dynverse metrics require a known benchmark topology
- The trajectory of interest is disconnected or contains cycles, which Totem cannot generate and the tree-based benchmark scoring assumes
- Multi-root (converging) trajectories where dynplot cannot generate correct pseudotime without manual work-around

## Inputs

- Benchmark single-cell datasets with ground-truth trajectories from the dynverse Zenodo deposit (10.5281/zenodo.1443566), as SingleCellExperiment objects
- Inferred trajectories (MST backbones built with Slingshot from selected clusterings)
- Simulated benchmark data such as binary_tree_1 for sanity checking

## Outputs

- Per-run dynverse overall scores (range [0, 1]) tabulated per dataset, parameter configuration, and seed (e.g. benchmark_overall_scores.csv)
- Aggregated max/mean overall-score curves as a function of the number of evaluated trajectories (top-n clusterings)
- Comparison tables/plots of scores across clustering-selection criteria (Totem connectivity+VRC vs ASW vs VRC vs random)

## How to apply

Load the benchmark datasets from the dynverse Zenodo deposit (10.5281/zenodo.1443566) as SingleCellExperiment objects and prepare them with PrepareTotem (Seurat LogNormalize normalization, 5-dimensional LMDS embedding). For each inferred trajectory — e.g. MSTs built with Slingshot from selected CLARA k-medoids clusterings filtered by min.cluster.size = 5 — compute the dynverse metrics against the benchmark ground truth and record the overall score. Aggregate scores across configurations (e.g. 10 random seeds per parameter setting) and report both maximum and mean overall scores when sweeping over a top-n set of ranked clusterings. Validation requires one score entry per dataset, configuration, and seed, with all datasets covered and scores within [0, 1].

## Related tools

- **Totem** (R package that generates the CLARA clusterings and MST trajectory candidates being benchmarked) — https://github.com/elolab/Totem
- **Slingshot** (constructs the Minimum Spanning Tree trajectory backbone from the selected clustering before scoring)
- **SingleCellExperiment** (container format for the benchmark datasets fed into PrepareTotem)
- **dyndimred** (provides LMDS 5-dimensional embedding used to prepare the data prior to clustering and scoring)
- **Seurat** (LogNormalize normalization of benchmark datasets before trajectory inference)

## Examples

```
remotes::install_github("elolab/Totem"); sce <- PrepareTotem(binary_tree_1, normalization_method = "Seurat_LogNormalize", ndim = 5)
```

## Evaluation signals

- Output table contains one overall-score entry per dataset, clustering-count value, k-range upper limit, and seed
- All benchmark datasets (e.g. the 10 dynverse Zenodo datasets) are covered with no missing configurations
- All score values fall within the dynverse score range [0, 1]
- Score-versus-n curves are complete for all four selection criteria (connectivity+VRC, ASW, VRC, random) with no runtime errors
- Reproduced curves match the paper's behavior: connectivity+VRC attains the reported maximum/mean dynverse overall scores

## Limitations

- k-medoids (CLARA) may be unstable with high-dimensional PCA input (e.g. 30 principal components), producing 1-2 cell clusters that degrade downstream scores
- k-medoids can fail on extremely imbalanced datasets combining rare and prevalent cell types, requiring manual cluster merging before evaluation
- dynplot will not generate correct pseudotime for multi-root (converging) trajectories, complicating benchmark visualization

## Evidence

- [discussion] Single-cell -omics datasets containing a trajectory (2.0.0) [Data set]. Zenodo.: "Single-cell -omics datasets containing a trajectory (2.0.0) [Data set]. Zenodo. [https://doi.org/10.5281/zenodo.1443566]"
- [supplementary] Totem's robustness improved as the number of clusterings increased: [evidence span withheld — anchor status: not_found]
- [supplementary] Evaluate each inferred trajectory with dynverse metrics: [evidence span withheld — anchor status: not_found]
- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [readme] Totem uses Slingshot to find linear curves through the lineages of the tree.: [evidence span withheld — anchor status: external_doc]
