---
name: receptor-weighted-sample-fitting
description: Use when inferring the spatial distance of a signaling pathway from scRNA-seq
  data with a SpaOTsc-derived cell–cell distance matrix, and known ligand, receptor,
  and downstream target gene lists are available (e.g. Wg/fz in the Drosophila embryo).
when_to_use_negative:
- The signaling system's ligand, receptor, and downstream genes are unknown — the
  article notes that cell–cell communication inference relies on known genes involved
  in specific signaling
- No spatial cell–cell distance matrix exists yet — this skill requires the upstream
  SpaOTsc transport_plan/cell_cell_distance steps to have been run first
- Dataset is excessively large — the article warns the optimal-transport-based computation
  'can become intractable when the dataset is excessively large'
tools:
- name: SpaOTsc
  role: Provides infer_signal_range_ml, which aggregates ligand expression over distance-matrix
    neighborhoods and fits the receptor-weighted random forest models
  repo: github.com/zcang/SpaOTsc
- name: scikit-learn
  role: Random forest regression engine underlying the sample-weighted fitting of
    downstream target gene expression
- name: pandas
  role: Loads and manipulates the Drosophila scRNA-seq expression matrix (dge_normalized.txt)
    and gene lists
- name: numpy
  role: Array operations for distance-matrix neighborhood aggregation and feature
    construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:0540a0e9d7bf16c632258b2094afa8b62481e961a0bdfbd3354530809b0aea7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# receptor-weighted-sample-fitting

## Summary

Fits random-forest regression models that predict downstream target gene expression from spatially aggregated ligand features, weighting each cell by its receptor expression so that the fit emphasizes cells capable of receiving the signal; used by SpaOTsc's infer_signal_range_ml to estimate spatial signaling ranges.

## When to use

Use when inferring the spatial distance of a signaling pathway from scRNA-seq data with a SpaOTsc-derived cell–cell distance matrix, and known ligand, receptor, and downstream target gene lists are available (e.g. Wg/fz in the Drosophila embryo).

## When NOT to use

- The signaling system's ligand, receptor, and downstream genes are unknown — the article notes that cell–cell communication inference relies on known genes involved in specific signaling
- No spatial cell–cell distance matrix exists yet — this skill requires the upstream SpaOTsc transport_plan/cell_cell_distance steps to have been run first
- Dataset is excessively large — the article warns the optimal-transport-based computation 'can become intractable when the dataset is excessively large'

## Inputs

- Normalized scRNA-seq expression matrix (dge_normalized.txt from Synapse syn15665609) loaded as a pandas DataFrame
- Spatial cell–cell distance matrix from the SpaOTsc cell_cell_distance step (upstream optimal transport plan)
- Ligand list, receptor list, and downstream target gene list for the signaling system (Supplementary Methods section 3.1)
- Effect ranges / neighborhood radii η (e.g. effect_ranges=[10,50,100] µm) defining the neighborhood aggregation

## Outputs

- Signal strength profile: ligand feature importance as a function of neighborhood radius η
- Spatial signaling range per ligand (η with maximal ligand feature importance; e.g. Wg ≈ 100 µm, Dpp ≈ 125 µm)
- signaling_ranges.csv with the inferred ranges for each signaling system

## How to apply

Start from the spatial cell–cell distance matrix computed by SpaOTsc (after transport_plan and cell_cell_distance), and the Drosophila scRNA-seq expression matrix (dge_normalized.txt from Synapse syn15665609) with the ligand, receptor, and downstream gene lists from Supplementary Methods section 3.1. For each signaling system, aggregate ligand expression over cell neighborhoods of varying radius η defined on the cell–cell distance matrix to form L̃(η), and add genes highly correlated with the downstream target gene as features. Fit random forest models (scikit-learn) predicting downstream target expression, using receptor expression levels as sample weights via SpaOTsc's infer_signal_range_ml, so cells with high receptor expression dominate the regression. For each ligand, report the η at which its aggregated feature L̃(η) attains the highest feature importance as the spatial signaling distance. In the Drosophila validation case this recovers ~100 µm for Wg and ~125 µm for Dpp (Fig 4c,d), written to signaling_ranges.csv.

## Related tools

- **SpaOTsc** (Provides infer_signal_range_ml, which aggregates ligand expression over distance-matrix neighborhoods and fits the receptor-weighted random forest models) — github.com/zcang/SpaOTsc
- **scikit-learn** (Random forest regression engine underlying the sample-weighted fitting of downstream target gene expression)
- **pandas** (Loads and manipulates the Drosophila scRNA-seq expression matrix (dge_normalized.txt) and gene lists)
- **numpy** (Array operations for distance-matrix neighborhood aggregation and feature construction)

## Examples

```
signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])
```

## Evaluation signals

- Reproduction of the published Drosophila ranges: Wg ≈ 100 µm and Dpp ≈ 125 µm (Fig 4c,d validation)
- Each ligand's aggregated feature L̃(η) attains a single clear maximum feature importance at one neighborhood radius; that radius is reported as the signaling distance
- Regression samples are weighted by receptor expression — cells with zero/low receptor expression contribute little to the fit
- Output file signaling_ranges.csv contains one inferred range per signaling system (Wg, Dpp) with values in the expected micrometer range
- Model fitting only proceeds after a valid cell–cell distance matrix exists from transport_plan/cell_cell_distance

## Limitations

- The approach does not consider time delay in cell–cell communication (ligand diffusion time, intracellular cascade reacting time)
- Depends on known ligand/receptor/downstream gene lists; cannot discover signaling components de novo
- Computationally intractable for excessively large datasets; cell–cell distance inference requires iterative optimal transport over all cell pairs (approximation possible with a small number of landmark positions)
- No software version of SpaOTsc is stated in the article

## Evidence

- [readme] README usage of infer_signal_range_ml: "signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])"
- [methods] Methods description of the machine-learning signaling range step: "spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])"
- [abstract] Spatial metric prerequisite: "A spatial metric for individual cells in scRNA-seq data is first established based on a map connecting it with the spatial measurements."
- [discussion] Time-delay limitation: "our approach does not consider the time delay that may take place in cell–cell communication"
