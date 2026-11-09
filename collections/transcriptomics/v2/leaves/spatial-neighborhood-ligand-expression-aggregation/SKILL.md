---
name: spatial-neighborhood-ligand-expression-aggregation
description: Use when scRNA-seq cells have been equipped with a SpaOTsc-derived spatial
  cell-cell distance matrix and you need to quantify at what spatial distance a known
  ligand explains downstream target gene expression (signaling range inference).
when_to_use_negative:
- No spatial cell-cell distance matrix is available yet — the aggregation requires
  the SpaOTsc optimal-transport-derived cell-cell distance, so run transport_plan/cell_cell_distance
  first
- The signaling pathway's ligand/receptor/downstream genes are unknown — the approach
  relies on known genes involved in specific signaling
- The input is spatial transcriptomics data with native coordinates and only short-range
  signaling distances are needed — direct distance-based approaches may be more appropriate
  than transporting scRNA-seq into space
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SpaOTsc
  role: Provides the optimal-transport cell-cell distance matrix and the infer_signal_range_ml
    routine that fits the radius-dependent ligand-aggregation random forest models
  repo: github.com/zcang/SpaOTsc
- name: pandas
  role: Loads and manipulates the dge_normalized.txt scRNA-seq expression matrix as
    a DataFrame
- name: numpy
  role: Array operations for neighborhood aggregation of ligand expression over the
    cell-cell distance matrix
- name: scikit-learn
  role: Random forest regression backend used by infer_signal_range_ml to predict
    downstream target expression with ligand features L̃(η)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:042ad4022ca4663bcd87836e4e6ef437a5387051da78118ceafdcb9bd063bfa6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# spatial-neighborhood-ligand-expression-aggregation

## Summary

Aggregates ligand expression within spatial cell-cell neighborhoods of varying radius η (computed from the SpaOTsc optimal-transport cell-cell distance matrix) to form radius-dependent ligand features L̃(η), which are then fed to a receptor-weighted random forest to infer the spatial range of a signaling pathway (e.g., ~100 µm for Wg and ~125 µm for Dpp in the Drosophila embryo).

## When to use

Use when scRNA-seq cells have been equipped with a SpaOTsc-derived spatial cell-cell distance matrix and you need to quantify at what spatial distance a known ligand best explains downstream target gene expression (signaling range inference).

## When NOT to use

- No spatial cell-cell distance matrix is available yet — the aggregation requires the SpaOTsc optimal-transport-derived cell-cell distance, so run transport_plan/cell_cell_distance first
- The signaling pathway's ligand/receptor/downstream genes are unknown — the approach relies on known genes involved in specific signaling
- The input is spatial transcriptomics data with native coordinates and only short-range signaling distances are needed — direct distance-based approaches may be more appropriate than transporting scRNA-seq into space

## Inputs

- Normalized scRNA-seq expression matrix (dge_normalized.txt, Synapse syn15665609) as a pandas DataFrame with cells as rows and genes as columns
- Spatial cell-cell distance matrix produced by SpaOTsc cell_cell_distance() (optimal transport plan)
- Curated ligand, receptor, and downstream target gene lists (Supplementary Methods section 3.1)
- Candidate neighborhood radii / effect_ranges grid (e.g., [10, 50, 100] in the API example)

## Outputs

- Radius-resolved ligand feature importances L̃(η) from the random forest models
- Inferred spatial signaling range per ligand (η with maximal ligand feature importance), e.g., ~100 µm for Wg and ~125 µm for Dpp
- signaling_ranges.csv with the inferred ranges for Wg and Dpp

## How to apply

Load the Drosophila scRNA-seq expression matrix (dge_normalized.txt from Synapse syn15665609) with pandas and the ligand, receptor, and downstream gene lists from Supplementary Methods section 3.1. Using the spatial cell-cell distance matrix from the upstream cell_cell_distance step, aggregate ligand expression of cells within neighborhoods of varying radius η to form L̃(η), and augment features with genes highly correlated to the downstream target gene. Fit random forest regressors via scikit-learn through the SpaOTsc infer_signal_range_ml routine, using receptor expression levels as sample weights to reflect receiving-cell competence. For each ligand, select the η at which the ligand feature L̃(η) attains the highest feature importance and report that η as the signaling distance; write results (e.g., Wg ~100 µm, Dpp ~125 µm) to signaling_ranges.csv.

## Related tools

- **SpaOTsc** (Provides the optimal-transport cell-cell distance matrix and the infer_signal_range_ml routine that fits the radius-dependent ligand-aggregation random forest models) — github.com/zcang/SpaOTsc
- **pandas** (Loads and manipulates the dge_normalized.txt scRNA-seq expression matrix as a DataFrame)
- **numpy** (Array operations for neighborhood aggregation of ligand expression over the cell-cell distance matrix)
- **scikit-learn** (Random forest regression backend used by infer_signal_range_ml to predict downstream target expression with ligand features L̃(η))

## Examples

```
signal_strengths,_=spsc.infer_signal_range_ml(['Wg'],['fz2'],['CycD','dpp'], effect_ranges=[10,50,100])
```

## Evaluation signals

- Reproduced signaling ranges match the reported values: approximately 100 µm for Wg and 125 µm for Dpp in the Drosophila embryo (Fig 4c,d)
- The selected signaling radius corresponds to the η at which the aggregated ligand feature L̃(η) attains the highest random forest feature importance, with importance declining at larger η
- Random forest prediction of the downstream target gene improves when receptor expression levels are used as sample weights, indicating receiving-cell specificity
- Output signaling_ranges.csv contains one inferred distance per queried ligand with physically plausible values on the effect_ranges grid

## Limitations

- Computationally intractable for excessively large datasets; cell-cell distance inference requires iterative optimal transport over all cell pairs (approximated with landmark positions)
- The approach does not consider time delay in cell-cell communication, such as ligand diffusion time and intracellular cascade reacting time
- Inference relies on known genes involved in the specific signaling pathway (curated ligand/receptor/downstream lists)
- The signaling range is restricted to the discrete effect_ranges grid supplied to infer_signal_range_ml

## Evidence

- [readme] signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100]): "signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])"
- [results] SpaOTsc infers a Wg signaling spatial range of about 100 µm and a longer Dpp signaling spatial range of 125 µm: [evidence span withheld — anchor status: not_found]
- [methods] the scRNA-seq data and the spatial data (in situ hybridization) were downloaded from the Dream Single cell Transcriptomics Challenge through Synapse ID (syn15665609): "the scRNA-seq data and the spatial data (in situ hybridization) were downloaded from the Dream Single cell Transcriptomics Challenge through Synapse ID (syn15665609)10."
- [abstract] The cell–cell communications are then obtained by “optimally transporting” signal senders to target signal receivers in space.: "The cell–cell communications are then obtained by “optimally transporting” signal senders to target signal receivers in space."
- [discussion] our approach does not consider the time delay that may take place in cell–cell communication: "our approach does not consider the time delay that may take place in cell–cell communication"
