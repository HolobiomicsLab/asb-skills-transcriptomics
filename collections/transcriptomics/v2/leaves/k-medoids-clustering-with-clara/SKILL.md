---
name: k-medoids-clustering-with-clara
description: Use when the input is a preprocessed SingleCellExperiment with an embedding
  and the goal is tree-shaped trajectory inference where clustering quality — not
  a single clustering — determines the milestone topology.
when_to_use_negative:
- Dataset contains extremely rare and extremely prevalent cell types simultaneously
  — k-medoids may fail and clusters need manual merging
- Goal is a trajectory with cycles or disconnected components — Totem cannot generate
  these
- A reliable ground-truth clustering already exists and only the MST backbone is needed
  (e.g. Slingshot with known labels)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: R package that runs the CLARA k-medoids ensemble, applies min.cluster.size
    filtering, and ranks clusterings by Calinski-Harabasz score and cell connectivity
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree backbone from each selected clustering
- name: dyndimred
  role: Provides the 5-dimensional LMDS embedding on which CLARA clustering is performed
- name: SingleCellExperiment
  role: Container class for the prepared single-cell data before clustering
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:7a7541924610a26d22a349a416ea0d5391f21b92903d4182ae45ccb88846ae69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - BiocManager
  - R
  - SingleCellExperiment
  - Slingshot
  - TinGa
  - Totem
  - cowplot
  - dyndimred
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# k-medoids clustering with CLARA

## Summary

Generates a large ensemble of k-medoids (CLARA) clusterings over a range of k on an embedded single-cell dataset, so that downstream tree-topology selection (MST via Slingshot, cell connectivity, Calinski-Harabasz score) can pick a biologically meaningful trajectory backbone.

## When to use

Use when the input is a preprocessed SingleCellExperiment with an embedding and the goal is tree-shaped trajectory inference where clustering quality — not a single clustering — determines the milestone topology.

## When NOT to use

- Dataset contains extremely rare and extremely prevalent cell types simultaneously — k-medoids may fail and clusters need manual merging
- Goal is a trajectory with cycles or disconnected components — Totem cannot generate these
- A reliable ground-truth clustering already exists and only the MST backbone is needed (e.g. Slingshot with known labels)

## Inputs

- SingleCellExperiment object with normalized expression matrix (e.g. dynbenchmark datasets from Zenodo doi:10.5281/zenodo.1443566)
- 5-dimensional LMDS embedding of the cells (via dyndimred)
- Optional prior clustering labels to bias clustering selection
- Parameter settings: number of CLARA clusterings (default 10k), k-range upper limit, min.cluster.size (default 5)

## Outputs

- Ensemble of candidate clusterings with MST-derived tree topologies
- Top-ranked clustering selected by Calinski-Harabasz score and cell connectivity
- Cell-connectivity profile over the embedding indicating likely leaves (end points) and branch points
- Milestone network edge list of the inferred trajectory (e.g. milestone_networks.csv)

## How to apply

Wrap the expression data in a SingleCellExperiment object, prepare it with PrepareTotem, and embed with 5-dimensional landmark MDS (LMDS) from dyndimred rather than PCA, since CLARA misbehaves on high-dimensional PCA (e.g. 30 components) input. Run the k-medoids (CLARA) algorithm a large number of times (default 10,000; robustness plateaus above ~2500 clusterings), sampling k from a user-set range (upper limits 5–40 were swept in the robustness analysis). Filter out clusterings containing clusters smaller than min.cluster.size (default 5 cells), then rank the surviving clusterings with the Calinski-Harabasz score and the cell-connectivity measure; an optional prior clustering can direct selection via ARI similarity. The selected clustering feeds Slingshot's MST construction to yield the trajectory topology.

## Related tools

- **Totem** (R package that runs the CLARA k-medoids ensemble, applies min.cluster.size filtering, and ranks clusterings by Calinski-Harabasz score and cell connectivity) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree backbone from each selected clustering)
- **dyndimred** (Provides the 5-dimensional LMDS embedding on which CLARA clustering is performed)
- **SingleCellExperiment** (Container class for the prepared single-cell data before clustering)

## Examples

```
remotes::install_github("elolab/Totem"); prepared <- Totem::PrepareTotem(sce, dimred_method = "lmds", ndim = 5)
```

## Evaluation signals

- Every retained clustering satisfies min.cluster.size (≥5 cells per cluster); clusterings with too-small clusters are discarded
- Robustness check: overall dynverse scores aggregated over 10 random seeds per configuration show small variation once the number of clusterings exceeds ~2500
- Inferred milestone network matches the expected tree topology, with cell-connectivity peaks coinciding with true end points and the branching point (as for multifurcating_4 and mouse thymus T cells)
- Top clustering is selected by Calinski-Harabasz score combined with cell connectivity, not by a single run of CLARA
- Output table covers all dataset × n_clusterings × k-limit × seed combinations, with scores in [0, 1]

## Limitations

- k-medoids is unstable on high-dimensional PCA input (e.g. 30 principal components), producing 1–2 cell clusters; use LMDS instead
- On extremely imbalanced datasets with both very rare and very prevalent cell types, k-medoids may not work properly and manual cluster merging is required
- Default 10,000 CLARA runs are computationally heavy; performance plateaus above 2500 clusterings

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] CLARA ensemble generation: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] default number of runs: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [intro] min.cluster.size filter: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [discussion] PCA failure mode: "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
- [readme] package purpose: [evidence span withheld — anchor status: external_doc]
