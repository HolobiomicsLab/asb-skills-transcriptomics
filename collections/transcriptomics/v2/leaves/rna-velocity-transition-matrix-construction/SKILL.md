---
name: rna-velocity-transition-matrix-construction
description: Use when spliced/unspliced AnnData with scvelo-computed RNA velocity
  is available and a probabilistic cell-state transition matrix is needed to sample
  differentiation trajectories or estimate root/terminal states.
when_to_use_negative:
- When the underlying RNA velocity estimation itself is faulty (e.g., time-dependent
  degradation rates or spurious directionality as in the hepatocyte zonation dataset)
  — trajectory inference on the resulting matrix is not meaningful
- When root and terminal cell states are not clearly identifiable — the authors do
  not recommend performing trajectory inference with Cytopath in that case
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Method whose Markov chain model is defined by the RNA velocity-derived transition
    probability matrix; also estimates root/terminal states via a Markov random-walk
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Computes the RNA velocity field from spliced/unspliced counts that the transition
    matrix is built from
- name: cytopath-notebooks
  role: Demonstration notebooks applying Cytopath to public scRNA-seq datasets
  repo: https://github.com/aron0093/cytopath-notebooks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:ca66212f9b2fa0ea77736755df3470dcadfccfdf36059d8b1d74ef4c6e0d63ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - cytopath-notebooks
  - fastdtw
  - numpy
  - scikit-learn
  - scvelo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# rna-velocity-transition-matrix-construction

## Summary

Constructs a cell-to-cell transition probability matrix from RNA velocity fields (processed with scvelo) of spliced/unspliced single-cell RNA-seq data, which serves as the Markov chain model backbone for Cytopath's simulation-based trajectory inference.

## When to use

Use when spliced/unspliced AnnData with scvelo-computed RNA velocity is available and a probabilistic cell-state transition matrix is needed to sample differentiation trajectories or estimate root/terminal states.

## When NOT to use

- When the underlying RNA velocity estimation itself is faulty (e.g., time-dependent degradation rates or spurious directionality as in the hepatocyte zonation dataset) — trajectory inference on the resulting matrix is not meaningful
- When root and terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference with Cytopath in that case

## Inputs

- Spliced/unspliced single-cell RNA-seq count matrix as AnnData, preprocessed with scvelo RNA velocity
- Optional user-supplied root and terminal cell labels
- Optional pre-computed cell clustering (Louvain used internally if absent)

## Outputs

- Cell-to-cell transition probability matrix encoding Markov transition probabilities between cell states
- Root and terminal state probabilities per cell (from the Markov random-walk model)
- Terminal region assignments used to group downstream simulations

## How to apply

Load the spliced/unspliced AnnData processed with scvelo and construct the cell-to-cell transition probability matrix from the RNA velocity field, following the Cytopath approach in which transcriptional activity information from RNA velocity defines a Markov chain model. Derive root and terminal states either from user-supplied labels or from a Markov random-walk model that reuses the transition probability matrix itself (as described by La Manno et al.), restricting inference to runs where these states are clearly identifiable. If no cell clustering is supplied, compute one internally with Louvain to determine terminal regions. Once built, this matrix drives Markov sampling of consecutive cell state transitions initialized at root cells, with the number of trajectories either user-defined or estimated unsupervised and no topological constraints imposed. The same matrix can also support undirected simulations initialized at random cells to discover regions of transcriptional state switching.

## Related tools

- **Cytopath** (Method whose Markov chain model is defined by the RNA velocity-derived transition probability matrix; also estimates root/terminal states via a Markov random-walk) — https://github.com/aron0093/cytopath
- **scvelo** (Computes the RNA velocity field from spliced/unspliced counts that the transition matrix is built from)
- **cytopath-notebooks** (Demonstration notebooks applying Cytopath to public scRNA-seq datasets) — https://github.com/aron0093/cytopath-notebooks

## Examples

```
import cytopath; adata = scv.read('endocrinogenesis.h5ad'); cytopath.trajectory(adata, groupby='louvain', target='delta', n_trajectories=3000)
```

## Evaluation signals

- Pseudotime inferred along trajectories built from the matrix shows high Spearman correlation with known ordering of cell types per lineage (as in the developing mouse hippocampus dataset with 18,140 cells) and is robust across multiple runs
- Root cell probability is concentrated (not fuzzy or distributed without discernible pattern) at biologically expected start states; terminal probabilities align with known lineages (e.g., alpha, beta, gamma, delta cells in pancreatic endocrinogenesis)
- Pseudotime is robust to root and terminal probability thresholds within a range of [0.85, 1]
- Simulations initialized at root cells terminate at coherent terminal regions, enabling downstream dynamic time warping alignment per terminal state

## Limitations

- RNA velocity estimation can be unrepresentative of true transcriptional dynamics due to simplifying assumptions (time-invariant rates of transcription, splicing, degradation; single regulatory regime per gene)
- Incorrect velocity estimation leads to spurious root/terminal inference, e.g., spurious root points toward the terminus in the erythroid gastrulation dataset and fuzzy, patternless root probabilities in the hepatocyte zonation dataset
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available

## Evidence

- [results] Markov sampling of consecutive cell state transitions initialized at predefined root states to generate simulated cell state sequences: "In the first step, Markov sampling of consecutive cell state transitions is performed based on the probabilities of the transition matrix, resulting in an ensemble of simulated cell state sequences."
- [results] Root and terminal state estimation from the transition probability matrix: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno et al."
- [results] Internal Louvain clustering for terminal regions: "otherwise, a clustering is computed internally using Louvain."
- [abstract] Markov chain model definition from RNA velocity: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [readme] Velocity-derived Markov model description in README: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
- [discussion] Do not infer when root/terminal states unclear: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
