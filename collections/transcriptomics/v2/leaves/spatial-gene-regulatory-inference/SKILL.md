---
name: spatial-gene-regulatory-inference
description: Use when non-spatial scRNA-seq data has been equipped with a spatial
  cell–cell distance matrix (e.g., via SpaOTsc optimal transport) and the goal is
  to infer which gene in one cell affects which gene in a neighbor cell across spatial
  ranges.
when_to_use_negative:
- When no spatial measurements of a small set of genes are available to build the
  cell–cell distance matrix via optimal transport
- When the dataset is excessively large, since cell–cell distance inference over all
  cell pairs can become computationally intractable
- When studying time-resolved signaling where ligand diffusion or intracellular cascade
  time delays must be modeled — this approach does not consider time delay
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpaOTsc
  role: Provides the spatial_grn_range function that constructs the spatial map of
    intercellular gene–gene regulatory information flow within a given effect_range
    (e.g., 50 µm), building on its cell–cell distance matrix
  repo: github.com/zcang/SpaOTsc
- name: dit
  role: Python package used to compute partial information decomposition and the accumulated
    unique information u(Gsrc,Gtar,η) between source and target gene distributions
- name: pandas
  role: Loads and manipulates the normalized scRNA-seq expression matrix (dge_normalized.txt)
    and variance ranking of genes
- name: numpy
  role: Handles η-neighborhood observations and probability distribution construction
    over Bayesian-block-binned expression values
- name: matplotlib
  role: Plots information flow versus spatial distance for gene pairs (e.g., the Twist–Snail
    profile matching Fig. 5a)
- name: scipy
  role: SpaOTsc dependency used in the optimal transport and distance computations
    underlying the gene regulatory map
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:eb180a8e72b88c94bea082f8c81d9c928e1a317c0b642b517fb32c626651e674
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SpaOTsc
  - dit
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# spatial-gene-regulatory-inference

## Summary

Constructs a spatial map of intercellular gene–gene regulatory information flow from scRNA-seq data by quantifying unique information (via partial information decomposition) between a source gene's η-neighborhood expression and a target gene, using SpaOTsc cell–cell distances. Applied to the Drosophila embryo to recover regulations such as Twist affecting Snail across ~25 µm.

## When to use

Use when non-spatial scRNA-seq data has been equipped with a spatial cell–cell distance matrix (e.g., via SpaOTsc optimal transport) and the goal is to infer which gene in one cell affects which gene in a neighbor cell across spatial ranges.

## When NOT to use

- When no spatial measurements of a small set of genes are available to build the cell–cell distance matrix via optimal transport
- When the dataset is excessively large, since cell–cell distance inference over all cell pairs can become computationally intractable
- When studying time-resolved signaling where ligand diffusion or intracellular cascade time delays must be modeled — this approach does not consider time delay

## Inputs

- Normalized scRNA-seq expression matrix (genes × cells, e.g., dge_normalized.txt from Synapse syn15665609)
- Spatial cell–cell distance matrix derived from the SpaOTsc optimal transport plan
- List of candidate genes (e.g., top highly variable genes by variance)
- Set of candidate spatial ranges η (e.g., 10, 25, 50, 100 µm)

## Outputs

- Intercellular gene–gene regulatory information flow map (source–target pairs with unique-information scores)
- Information flow versus spatial distance (η) profiles for each gene pair
- Ranked gene–gene spatial regulatory edges (e.g., Twist → Snail at ~25 µm)

## How to apply

Load the normalized scRNA-seq expression matrix (e.g., dge_normalized.txt from Synapse syn15665609) and the spatial cell–cell distance matrix computed by SpaOTsc's transport_plan/cell_cell_distance step. Rank genes by variance and select a small set of highly variable genes (e.g., top 20). For each gene pair, build η-neighborhood observations of the source gene using the distance matrix, represent expression as probability distributions over Bayesian-block-binned expression values, and run partial information decomposition with the dit package, summing the accumulated unique information u(Gsrc,Gtar,η) over genes highly intracellularly correlated with the target. Sweep the neighborhood radius η across spatial distances (e.g., 10, 25, 50, 100 µm) using SpaOTsc's spatial_grn_range to obtain a distance profile of information flow for each gene pair.

## Related tools

- **SpaOTsc** (Provides the spatial_grn_range function that constructs the spatial map of intercellular gene–gene regulatory information flow within a given effect_range (e.g., 50 µm), building on its cell–cell distance matrix) — github.com/zcang/SpaOTsc
- **dit** (Python package used to compute partial information decomposition and the accumulated unique information u(Gsrc,Gtar,η) between source and target gene distributions)
- **pandas** (Loads and manipulates the normalized scRNA-seq expression matrix (dge_normalized.txt) and variance ranking of genes)
- **numpy** (Handles η-neighborhood observations and probability distribution construction over Bayesian-block-binned expression values)
- **matplotlib** (Plots information flow versus spatial distance for gene pairs (e.g., the Twist–Snail profile matching Fig. 5a))
- **scipy** (SpaOTsc dependency used in the optimal transport and distance computations underlying the gene regulatory map)

## Examples

```
intercellular_grn = spsc.spatial_grn_range(['Twist','Snail'], effect_range=25)
```

## Evaluation signals

- The reproduced Twist–Snail information-flow distance profile peaks at ~25 µm, matching the published Fig. 5a curve
- Sweeping η over 10, 25, 50, and 100 µm yields distinct information-flow values per gene pair, confirming the spatial range sweep executed
- Unique information u(Gsrc,Gtar,η) sums are nonnegative and computed over genes highly intracellularly correlated with the target
- The resulting intercellular GRN is restricted to the selected highly variable genes (e.g., top 20) rather than all genes
- The spatial_grn_range call returns an intercellular GRN object covering the specified effect_range

## Limitations

- Computationally intractable for excessively large datasets; cell–cell distance inference requires iterative optimal transport over all cell pairs
- The approach does not consider time delay in cell–cell communication (ligand diffusion time, intracellular cascade reacting time)
- Approximation is made by using only a small number of landmark positions
- No software version of SpaOTsc is stated in the article

## Evidence

- [abstract] PID over η-neighborhoods: "Using partial information decomposition, we next compute the intercellular gene–gene information flow to estimate the spatial regulations between genes across cells."
- [intro] unique information across cells: "we then use partial information decomposition28,29 to quantify the unique information provided by one gene to another gene across different cells."
- [methods] spatial_grn_range API: "intercellular_grn=spsc.spatial_grn_range(['Wnt5','fz','CycD','dpp'], effect_range=50)"
- [readme] README usage of spatial_grn_range: "construct the spatial map of intercellular gene-gene regulatory information flow within a spatial range of 50"
- [discussion] time delay limitation: "our approach does not consider the time delay that may take place in cell–cell communication"
