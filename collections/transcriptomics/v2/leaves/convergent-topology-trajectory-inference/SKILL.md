---
name: convergent-topology-trajectory-inference
description: Use when scRNA-seq data exhibit a convergent differentiation topology
  (e.g., the 157-cell neonatal mouse inner ear dataset, Burns et al., GSE71982) that
  violates the topology-related assumptions of tree-based trajectory methods.
when_to_use_negative:
- When root and terminal cell states are not clearly identifiable — the authors state
  they do not recommend performing trajectory inference with Cytopath in that setting
- When the underlying RNA velocity estimation itself is faulty — trajectory inference
  with Cytopath is not expected to be meaningful in that case
- When the topology can be assumed tree-like and a simpler method such as Slingshot
  suffices
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: velocyto
  role: Generates the spliced/unspliced/ambiguous count matrices from raw reads that
    feed RNA velocity estimation
  repo: https://github.com/velocyto-team/velocyto-notebooks
- name: scvelo
  role: Preprocesses the AnnData object and estimates RNA velocity, which Cytopath
    depends on
- name: Cytopath
  role: Defines the Markov chain model from the velocity field, simulates the trajectory
    ensemble, aligns simulations with dynamic time warping, and assigns pseudotime
  repo: https://github.com/aron0093/cytopath
- name: cytopath-notebooks
  role: Sample notebooks demonstrating Cytopath on publicly available single cell
    RNA sequencing datasets
  repo: https://github.com/aron0093/cytopath-notebooks
- name: Slingshot
  role: Baseline comparator for tree-like topologies, cited as an alternative when
    convergence is absent
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:a31e12dd222f018b11a403ed9723db470cf9f15c29a3657ffc19ebe9d915af26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/velocyto-team/velocyto-notebooks
  tools:
  - Cytopath
  - Slingshot
  - cytopath-notebooks
  - scvelo
  - velocyto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# convergent-topology-trajectory-inference

## Summary

Reconstruct convergent differentiation trajectories and infer pseudotime from scRNA-seq using Cytopath, a simulation-based method that defines a Markov chain model from RNA velocity, simulates an ensemble of trajectories, and builds a consensus trajectory without imposing topological constraints.

## When to use

Use when scRNA-seq data exhibit a convergent differentiation topology (e.g., the 157-cell neonatal mouse inner ear dataset, Burns et al., GSE71982) that violates the topology-related assumptions of tree-based trajectory methods.

## When NOT to use

- When root and terminal cell states are not clearly identifiable — the authors state they do not recommend performing trajectory inference with Cytopath in that setting
- When the underlying RNA velocity estimation itself is faulty — trajectory inference with Cytopath is not expected to be meaningful in that case
- When the topology can be assumed tree-like and a simpler method such as Slingshot suffices

## Inputs

- Raw scRNA-seq reads (FASTQ, e.g. GEO accession GSE71982, 157 cells)
- Spliced/unspliced/ambiguous count matrices from velocyto
- AnnData object (.h5ad) with RNA velocity estimated by scvelo
- Optional pre-computed cell clustering; otherwise Cytopath computes Louvain clustering internally
- Known cell type ordering per lineage for validation

## Outputs

- Consensus differentiation trajectory (or trajectories) per terminal state
- Per-cell pseudotime values from alignment-score-based assignment to trajectory steps
- Per-run and mean Spearman correlation between inferred pseudotime and known cell type ordering across 10 independent runs

## How to apply

Download raw reads and run velocyto to produce spliced/unspliced/ambiguous count matrices, then load them into an AnnData object and use scvelo (>=0.1.25) to preprocess and estimate RNA velocity, since Cytopath depends on scvelo. Run Cytopath on the velocity field: it defines a Markov chain model from the RNA velocity, and root/terminal states can be derived unsupervised from the Markov random-walk model using the transition probability matrix itself — appropriate for convergent topologies. Simulate an ensemble of differentiation trajectories, repeated for 10 independent initialisations; simulations that terminate at a common terminal state are aligned using dynamic time warping, and cell states are averaged across aligned simulations to define consensus trajectory steps. Assign cells to trajectory steps via an alignment score evaluating similarity of each cell's static and velocity profile to each trajectory step, yielding inferred pseudotime per cell. Judge success by computing the Spearman correlation between inferred pseudotime and the known cell type ordering per lineage, requiring high and robust (consistent) correlations across the 10 runs.

## Related tools

- **velocyto** (Generates the spliced/unspliced/ambiguous count matrices from raw reads that feed RNA velocity estimation) — https://github.com/velocyto-team/velocyto-notebooks
- **scvelo** (Preprocesses the AnnData object and estimates RNA velocity, which Cytopath depends on)
- **Cytopath** (Defines the Markov chain model from the velocity field, simulates the trajectory ensemble, aligns simulations with dynamic time warping, and assigns pseudotime) — https://github.com/aron0093/cytopath
- **cytopath-notebooks** (Sample notebooks demonstrating Cytopath on publicly available single cell RNA sequencing datasets) — https://github.com/aron0093/cytopath-notebooks
- **Slingshot** (Baseline comparator for tree-like topologies, cited as an alternative when convergence is absent)

## Examples

```
import scvelo as scv, cytopath; scv.tl.velocity(adata); cytopath.trajectory(adata, clustering='louvain')
```

## Evaluation signals

- Spearman correlation between Cytopath-inferred pseudotime and the known ordering of cell types is high per lineage and robust across the 10 independent runs
- Both lineages of the convergent inner ear topology are reconstructed and converge on their expected terminal states without imposing topological constraints
- Root/terminal state estimation is unsupervised and matches the convergent topology (no spurious root points at the terminus of the expected process, as occurred in the faulty-velocity erythroid gastrulation example)
- Cells are assigned to exactly one consensus trajectory step via the alignment score combining static and velocity profile similarity
- Inferred pseudotime remains stable to root and terminal probability thresholds within a range of [0.85, 1] (reported robust, 'data not shown')

## Limitations

- RNA velocity estimation can be unrepresentative of true transcriptional dynamics due to simplifying assumptions (time-invariant rates, single regulatory regime per gene)
- For the erythroid gastrulation dataset with incorrect velocity, a spurious set of root points was inferred toward the terminus of the expected process and endpoints spuriously in the middle
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available
- Authors do not recommend performing trajectory inference with Cytopath when root and terminal cell states are not clearly identifiable

## Evidence

- [abstract] Markov chain model basis: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [results] Convergent topology motivation: "given its relatively small size of 157 cells as well as a convergent differentiation topology that violates the topology-related assumptions of several methods."
- [results] Cell-to-step assignment: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [results] Unsupervised endpoints: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno"
- [results] Robustness criterion: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs"
- [readme] Convergence support in README: "Cytopath can model complex behaviours like cycling and convergence as well as cooccurring combinations of multiple"
- [discussion] Faulty velocity caveat: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
