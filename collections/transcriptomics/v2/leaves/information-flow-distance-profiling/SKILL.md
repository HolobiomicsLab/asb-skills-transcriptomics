---
name: information-flow-distance-profiling
description: Use when scRNA-seq data has been equipped with a spatial cell–cell distance
  matrix and the goal is to infer which gene in one cell affects which gene in a neighboring
  cell, and at which spatial range.
when_to_use_negative:
- Input lacks any spatial measurements or a cell–cell distance matrix — the η-neighborhoods
  cannot be built without spatial context
- 'The dataset is excessively large: the article notes ''the computation can become
  intractable when the dataset is excessively large'' and recommends landmark approximations
  instead'
- The question concerns time-resolved signaling cascades — the approach does not consider
  time delay (ligand diffusion or intracellular cascade reacting time)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: SpaOTsc
  role: Provides spatial_grn_range to sweep neighborhood radii and construct the spatial
    map of intercellular gene–gene regulatory information flow
  repo: github.com/zcang/SpaOTsc
- name: dit
  role: Computes partial information decomposition (unique information) between Bayesian-block-binned
    expression distributions of source and target genes
- name: numpy
  role: Array handling for expression matrices, distance matrices, and neighborhood
    construction
- name: pandas
  role: Loads and manipulates the normalized scRNA-seq DataFrame (dge_normalized.txt)
- name: matplotlib
  role: Plots information flow versus spatial distance for gene pairs (e.g. the Twist–Snail
    profile matching Fig. 5a)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:71ededfa506ae1b7aaa8058f4f2f45c9f32d610635da23779dece4f3e885e5e9
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# information-flow-distance-profiling

## Summary

Profiles intercellular gene–gene regulatory information flow as a function of spatial distance between cells, using partial information decomposition on scRNA-seq data paired with SpaOTsc-derived cell–cell distances (e.g. Twist→Snail in the Drosophila embryo peaking at ~25 µm).

## When to use

Use when scRNA-seq data has been equipped with a spatial cell–cell distance matrix and the goal is to infer which gene in one cell affects which gene in a neighboring cell, and at which spatial range.

## When NOT to use

- Input lacks any spatial measurements or a cell–cell distance matrix — the η-neighborhoods cannot be built without spatial context
- The dataset is excessively large: the article notes 'the computation can become intractable when the dataset is excessively large' and recommends landmark approximations instead
- The question concerns time-resolved signaling cascades — the approach does not consider time delay (ligand diffusion or intracellular cascade reacting time)

## Inputs

- Normalized scRNA-seq expression matrix (rows=cells, columns=genes), e.g. dge_normalized.txt
- Spatial cell–cell distance matrix computed from the SpaOTsc optimal transport plan
- List/ranked set of highly variable genes (e.g. top 20 by variance)
- Neighborhood radius grid η (e.g. effect_ranges=[10,25,50,100] µm)

## Outputs

- Accumulated unique information u(Gsrc,Gtar,η) per source–target gene pair per neighborhood radius
- Intercellular gene–gene regulatory network (intercellular_grn) linking genes across cells at a given spatial range
- Information-flow-versus-distance curve per gene pair (e.g. Twist–Snail peaking at ~25 µm)

## How to apply

Load the normalized scRNA-seq expression matrix (e.g. dge_normalized.txt from Synapse syn15665609) and the cell–cell distance matrix from the upstream SpaOTsc transport step, then rank genes by variance and retain the top ~20 highly variable genes. For each source–target gene pair, build η-neighborhood observations of the source gene using the distance matrix and represent each gene's expression as a probability distribution over Bayesian-block-binned expression values. Run partial information decomposition with the dit package to compute the accumulated unique information u(Gsrc,Gtar,η), summing unique information over genes highly intracellularly correlated with the target. Sweep η across spatial distances (e.g. 10, 25, 50, 100 µm) using SpaOTsc's spatial_grn_range function to obtain the information-flow-versus-distance profile, whose peak indicates the regulatory spatial range.

## Related tools

- **SpaOTsc** (Provides spatial_grn_range to sweep neighborhood radii and construct the spatial map of intercellular gene–gene regulatory information flow) — github.com/zcang/SpaOTsc
- **dit** (Computes partial information decomposition (unique information) between Bayesian-block-binned expression distributions of source and target genes)
- **numpy** (Array handling for expression matrices, distance matrices, and neighborhood construction)
- **pandas** (Loads and manipulates the normalized scRNA-seq DataFrame (dge_normalized.txt))
- **matplotlib** (Plots information flow versus spatial distance for gene pairs (e.g. the Twist–Snail profile matching Fig. 5a))

## Examples

```
intercellular_grn = spsc.spatial_grn_range(['Twist','Snail'], effect_range=25)
```

## Evaluation signals

- The reproduced Twist–Snail information-flow distance profile peaks at ~25 µm, qualitatively matching published Fig. 5a
- Information flow is only computed for the selected highly variable genes (e.g. top 20 by variance), not the full gene set
- u(Gsrc,Gtar,η) values are non-negative and accumulate over genes highly intracellularly correlated with the target
- Distinct peaks in flow-versus-distance curves indicate gene pairs with finite signaling spatial range; flat/monotone curves suggest no spatially constrained regulation
- spatial_grn_range returns a valid intercellular GRN object for each tested effect_range (e.g. 50 µm)

## Limitations

- Computationally intractable for excessively large datasets; landmark-position approximation is used at the cost of accuracy
- Time delay in cell–cell communication (ligand diffusion time, intracellular cascade reacting time) is not modeled
- No software version of SpaOTsc stated in the publication

## Evidence

- [abstract] Using partial information decomposition, we next compute the intercellular gene–gene information flow to estimate the spatial regulations between genes across cells.: "Using partial information decomposition, we next compute the intercellular gene–gene information flow to estimate the spatial regulations between genes across cells."
- [readme] intercellular_grn=spsc.spatial_grn_range(['Wnt5','fz','CycD','dpp'], effect_range=50): "intercellular_grn=spsc.spatial_grn_range(['Wnt5','fz','CycD','dpp'], effect_range=50)"
- [intro] we then use partial information decomposition28,29 to quantify the unique information provided by one gene to another gene across different cells.: "we then use partial information decomposition28,29 to quantify the unique information provided by one gene to another gene across different cells."
- [discussion] the computation can become intractable when the dataset is excessively large: "the computation can become intractable when the dataset is excessively large"
