---
name: drosophila-embryo-gene-expression-analysis
description: Use when analyzing Drosophila embryo scRNA-seq (Synapse syn15665609)
  with a SpaOTsc-derived cell–cell distance matrix and the goal is to infer which
  gene in one cell regulates a gene in a neighbor cell across spatial distances.
when_to_use_negative:
- No spatial measurements or SpaOTsc-derived cell–cell distance matrix is available
  — PID neighborhoods cannot be constructed without a spatial metric for individual
  cells
- The dataset is excessively large — the article notes 'the computation can become
  intractable when the dataset is excessively large' for optimal-transport-based distance
  inference over all cell pairs
- The goal is signaling between known ligand–receptor pairs with effect ranges (e.g.
  Wnt5–fz at 10/50/100 µm) rather than genome-wide information flow — use spsc.infer_signal_range_ml
  or spsc.spatial_signaling_ot instead
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SpaOTsc
  role: Provides the cell–cell spatial distance matrix and the spatial_grn_range function
    that sweeps intercellular GRN inference across effect ranges (η, e.g. 50 µm)
  repo: https://github.com/zcang/SpaOTsc
- name: dit
  role: Computes partial information decomposition (unique information) for Bayesian-block-binned
    gene expression distributions
- name: numpy
  role: Handles variance ranking of genes and neighborhood-array construction for
    the top 20 highly variable genes
- name: pandas
  role: Loads and manipulates the dge_normalized.txt scRNA-seq DataFrame
- name: matplotlib
  role: Plots information flow versus spatial distance for gene pairs such as Twist–Snail
    (Fig. 5a-style curves)
- name: scipy
  role: Numerical support for distance and distribution computations in the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:0a05760c02da6014686628a92de3f9c95dc1d554f46ab558f39c00598743b3cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - SpaOTsc
  - dit
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# drosophila-embryo-gene-expression-analysis

## Summary

Infers intercellular gene–gene regulatory information flow in the Drosophila embryo by computing partial information decomposition (PID) over spatially arranged scRNA-seq neighborhoods derived from SpaOTsc cell–cell distances, e.g. detecting the Twist→Snail link peaking at ~25 µm.

## When to use

Use when analyzing Drosophila embryo scRNA-seq (Synapse syn15665609) with a SpaOTsc-derived cell–cell distance matrix and the goal is to infer which gene in one cell regulates a gene in a neighbor cell across spatial distances.

## When NOT to use

- No spatial measurements or SpaOTsc-derived cell–cell distance matrix is available — PID neighborhoods cannot be constructed without a spatial metric for individual cells
- The dataset is excessively large — the article notes 'the computation can become intractable when the dataset is excessively large' for optimal-transport-based distance inference over all cell pairs
- The goal is signaling between known ligand–receptor pairs with effect ranges (e.g. Wnt5–fz at 10/50/100 µm) rather than genome-wide information flow — use spsc.infer_signal_range_ml or spsc.spatial_signaling_ot instead

## Inputs

- Drosophila embryo scRNA-seq normalized expression matrix (dge_normalized.txt from Synapse syn15665609, Dream Single Cell Transcriptomics Challenge)
- Spatial cell–cell distance matrix from the SpaOTsc transport step (spsc.transport_plan + spsc.cell_cell_distance)
- Candidate gene list for the regulatory network (e.g. Twist, Snail, fz, CycD, dpp)

## Outputs

- Accumulated unique information u(Gsrc,Gtar,η) per source–target gene pair at each spatial neighborhood radius η
- Intercellular gene-gene regulatory information flow map/GRN from spsc.spatial_grn_range
- Information-flow vs. spatial-distance curves (e.g. Twist–Snail peaking at ~25 µm), plotted with matplotlib

## How to apply

Load dge_normalized.txt (Synapse syn15665609) together with the spatial cell–cell distance matrix produced by the upstream SpaOTsc optimal-transport step. Rank genes by variance and keep the top 20 highly variable genes. For each source–target gene pair, build η-neighborhood observations of the source gene from the distance matrix, represent expression as probability distributions over Bayesian-block-binned bins, and run partial information decomposition with the dit package to accumulate the unique information u(Gsrc,Gtar,η), summing unique information over genes highly intracellularly correlated with the target. Sweep the neighborhood radius η across spatial distances (10, 25, 50, 100 µm) with SpaOTsc's spatial_grn_range, then plot information flow vs. distance per gene pair; the Twist–Snail profile should peak near 25 µm, matching published Fig. 5a.

## Related tools

- **SpaOTsc** (Provides the cell–cell spatial distance matrix and the spatial_grn_range function that sweeps intercellular GRN inference across effect ranges (η, e.g. 50 µm)) — https://github.com/zcang/SpaOTsc
- **dit** (Computes partial information decomposition (unique information) for Bayesian-block-binned gene expression distributions)
- **numpy** (Handles variance ranking of genes and neighborhood-array construction for the top 20 highly variable genes)
- **pandas** (Loads and manipulates the dge_normalized.txt scRNA-seq DataFrame)
- **matplotlib** (Plots information flow versus spatial distance for gene pairs such as Twist–Snail (Fig. 5a-style curves))
- **scipy** (Numerical support for distance and distribution computations in the pipeline)

## Examples

```
from spaotsc import SpaOTsc; spsc.transport_plan(cost_matrix); spsc.cell_cell_distance(use_landmark=True); grn = spsc.spatial_grn_range(['Twist','Snail'], effect_range=25)
```

## Evaluation signals

- The Twist–Snail information-flow distance profile peaks at approximately 25 µm, qualitatively matching published Fig. 5a
- Gene selection is restricted to the top 20 genes ranked by variance of the normalized expression matrix
- Each gene's expression is representable as a valid probability distribution over Bayesian-block-binned expression values (non-negative, normalized)
- The accumulated unique information u(Gsrc,Gtar,η) increases the target's information when summed over genes highly intracellularly correlated with the target
- Sweeping η over {10, 25, 50, 100} µm yields monotonically interpretable information-flow curves per gene pair rather than artifacts at bin edges

## Limitations

- Computation can become intractable when the dataset is excessively large; cell–cell distance inference requires iterative optimal transport over all cell pairs (approximation via a small number of landmark positions)
- The approach does not consider time delay in cell–cell communication, such as ligand diffusion time and intracellular cascade reacting time
- No software version of SpaOTsc is stated in the article, complicating exact reproduction

## Evidence

- [abstract] Intercellular gene-gene information flow via PID: "Using partial information decomposition, we next compute the intercellular gene–gene information flow to estimate the spatial regulations between genes across cells."
- [methods] spatial_grn_range usage: "intercellular_grn=spsc.spatial_grn_range(['Wnt5','fz','CycD','dpp'], effect_range=50)"
- [methods] dit as PID dependency: "SpaOTsc depends on the following packages: pandas, numpy, scipy, networkx, python-igraph, louvain, POT, dit, astropy, scikit-learn, matplotlib, umap-learn."
- [readme] spatial_grn_range in README usage example: "# construct the spatial map of intercellular gene-gene regulatory information flow within a spatial range of 50"
- [discussion] intractability limitation: "the computation can become intractable when the dataset is excessively large"
- [discussion] Drosophila validation dataset: "gene expression reconstruction validation on zebrafish embryo and Drosophila embryo datasets"
