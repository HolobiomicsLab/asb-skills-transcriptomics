---
name: random-forest-signaling-range-inference
description: Use when you have scRNA-seq data with a spatial cell–cell distance matrix
  (e.g. from a SpaOTsc optimal transport mapping) and known ligand, receptor, and
  downstream gene lists, and you need to estimate the spatial range of a signaling
  pathway in micrometers.
when_to_use_negative:
- You lack a spatial cell–cell distance matrix — the random-forest neighborhoods are
  defined on distances from the SpaOTsc optimal transport mapping, not raw expression
  alone
- The ligand–receptor–downstream gene members of the signaling system are unknown,
  since the approach relies on known genes involved in specific signaling
- You need to model time delays (ligand diffusion or intracellular cascade reaction
  times) in the communication — SpaOTsc does not consider them
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SpaOTsc
  role: Provides the infer_signal_range_ml routine that builds radius-aggregated ligand
    features L̃(η), fits random forest models weighted by receptor expression, and
    returns signaling ranges
  repo: https://github.com/zcang/SpaOTsc
- name: scikit-learn
  role: Random forest regression used inside infer_signal_range_ml to predict downstream
    target gene expression and compute ligand feature importance
- name: pandas
  role: Loads the normalized scRNA-seq expression matrix (dge_normalized.txt) as the
    cell-by-gene DataFrame
- name: numpy
  role: Numerical aggregation of ligand expression over spatial distance neighborhoods
    and handling of the cell–cell distance matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:6d78988eec4d90230fd20e91ae413c215859856cc29e50728ebb3382b5190215
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# random-forest-signaling-range-inference

## Summary

Infers the spatial signaling distance of a ligand–receptor system from single-cell transcriptomics by fitting random-forest models that predict downstream target gene expression from aggregated ligand expression over neighborhoods of varying spatial radius, then selecting the radius at which the ligand feature attains maximum feature importance.

## When to use

Use when you have scRNA-seq data with a spatial cell–cell distance matrix (e.g. from a SpaOTsc optimal transport mapping) and known ligand, receptor, and downstream gene lists, and you need to estimate the spatial range of a signaling pathway in micrometers.

## When NOT to use

- You lack a spatial cell–cell distance matrix — the random-forest neighborhoods are defined on distances from the SpaOTsc optimal transport mapping, not raw expression alone
- The ligand–receptor–downstream gene members of the signaling system are unknown, since the approach relies on known genes involved in specific signaling
- You need to model time delays (ligand diffusion or intracellular cascade reaction times) in the communication — SpaOTsc does not consider them

## Inputs

- Normalized scRNA-seq expression matrix (rows=cells, columns=genes; e.g. dge_normalized.txt)
- Spatial cell–cell distance matrix from the SpaOTsc cell_cell_distance step
- Ligand gene list (e.g. Wg/Wnt5, Dpp)
- Receptor gene list (e.g. fz) used as sample weights
- Downstream target gene list (e.g. CycD, dpp) and genes highly correlated to the target
- Grid of candidate signaling radii effect_ranges (e.g. [10,50,100] µm)

## Outputs

- Feature-importance curves of ligand feature L̃(η) as a function of neighborhood radius η
- Inferred spatial signaling range per ligand in µm (Wg ≈ 100 µm, Dpp ≈ 125 µm)
- signaling_ranges.csv with the inferred ranges for each signaling system

## How to apply

Load the scRNA-seq expression matrix (e.g. dge_normalized.txt from Synapse syn15665609) with pandas plus the ligand, receptor, and downstream gene lists, together with the spatial cell–cell distance matrix from the upstream SpaOTsc cell_cell_distance step. For each signaling system, aggregate ligand expression of cells within neighborhoods of varying radius η defined on the spatial distance matrix to form L̃(η), and combine with genes highly correlated to the downstream target gene as features. Fit random forest models (scikit-learn) predicting the downstream target gene expression, using the receptor expression levels as sample weights, via the SpaOTsc infer_signal_range_ml routine with an explicit effect_ranges grid (e.g. [10,50,100]). Select the η at which the ligand feature L̃(η) attains the highest feature importance and report this η as the spatial signaling distance. In the Drosophila embryo this recovers a Wg signaling range of about 100 µm and a longer Dpp range of 125 µm (Fig 4c,d).

## Related tools

- **SpaOTsc** (Provides the infer_signal_range_ml routine that builds radius-aggregated ligand features L̃(η), fits random forest models weighted by receptor expression, and returns signaling ranges) — https://github.com/zcang/SpaOTsc
- **scikit-learn** (Random forest regression used inside infer_signal_range_ml to predict downstream target gene expression and compute ligand feature importance)
- **pandas** (Loads the normalized scRNA-seq expression matrix (dge_normalized.txt) as the cell-by-gene DataFrame)
- **numpy** (Numerical aggregation of ligand expression over spatial distance neighborhoods and handling of the cell–cell distance matrix)

## Examples

```
signal_strengths,_ = spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])
```

## Evaluation signals

- The recovered signaling ranges reproduce the reported values of approximately 100 µm for Wg and 125 µm for Dpp (Fig 4c,d)
- Each ligand's feature-importance curve over η exhibits a clear maximum; the selected η is the argmax of L̃(η) importance
- The output signaling_ranges.csv contains one inferred range per signaling system with finite positive distances within the supplied effect_ranges grid
- Random forest models must use receptor expression levels as sample weights, so runs without receptor weighting should give different (unreliable) importances

## Limitations

- Computation can become intractable when the dataset is excessively large because cell–cell distance inference requires iterative optimal transport over all cell pairs
- Approximation using only a small number of landmark positions may affect distance accuracy
- The approach does not consider time delay in cell–cell communication, such as ligand diffusion time or intracellular cascade reacting time
- Inference relies on known genes involved in the specific signaling system

## Evidence

- [methods] infer_signal_range_ml with effect_ranges grid: "signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])"
- [intro] estimate the spatial range of signaling with machine learning: [evidence span withheld — anchor status: not_found]
- [methods] distance matrix from optimal transport plan: [evidence span withheld — anchor status: not_found]
- [readme] SpaOTsc random forest dependency: "SpaOTsc relies on optimal transport theory (especially structured ot [1] and unbalanced ot [2]), partial information decomposition [3], and random forest model [4]."
- [discussion] no time delay in communication: "our approach does not consider the time delay that may take place in cell–cell communication"
