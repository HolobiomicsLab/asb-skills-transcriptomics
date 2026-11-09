---
name: cell-cell-distance-based-communication-analysis
description: Use when you have non-spatial scRNA-seq data plus a limited spatial measurement
  of a few genes, and need to infer cell-cell communication or the spatial distance/range
  of a signaling pathway (e.g. Wg ~100 µm, Dpp ~125 µm).
when_to_use_negative:
- The dataset is excessively large — the article notes 'the computation can become
  intractable when the dataset is excessively large' because cell-cell distance requires
  iterative optimal transport over all cell pairs (only landmark-based approximation
  is available)
- The signaling pathway's ligand/receptor/downstream genes are unknown — communication
  inference relies on known genes involved in specific signaling
- The question requires modeling time delays in communication (ligand diffusion time,
  intracellular cascade reacting time) — the approach explicitly does not consider
  them
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: SpaOTsc
  role: 'Core method: structured optimal transport to map scRNA-seq cells to spatial
    positions, compute cell-cell distances, and infer signaling spatial ranges via
    transport_plan, cell_cell_distance, spatial_signaling_ot, and infer_signal_range_ml'
  repo: github.com/zcang/SpaOTsc
- name: pandas
  role: Load and hold the scRNA-seq expression matrix (e.g. dge_normalized.txt) as
    the sc_data DataFrame passed to SpaOTsc.spatial_sc
- name: numpy
  role: Provide is_dmat, sc_dmat, and cost_matrix arrays consumed by the SpaOTsc transport
    and distance routines
- name: scikit-learn
  role: Random-forest regression backend used by infer_signal_range_ml to predict
    downstream target gene expression with receptor expression as sample weights
- name: Giotto
  role: Downstream spatial transcriptomics pipeline that the SpaOTsc-inferred spatial
    metric/annotated scRNA-seq data can be fed into
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:41ff0a69d69696f4d9fcf9772862aaca3e0494af5c5d21ca16c95f94b3d8d6ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Giotto
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-cell-distance-based-communication-analysis

## Summary

Infers cell-cell communications and spatial signaling ranges by first equipping non-spatial scRNA-seq cells with a spatial cell-cell distance via SpaOTsc structured optimal transport, then optimally transporting ligand-expressing sender cells to receptor-expressing receiver cells and fitting random-forest models over distance neighborhoods.

## When to use

Use when you have non-spatial scRNA-seq data plus a limited spatial measurement of a few genes, and need to infer cell-cell communication or the spatial distance/range of a signaling pathway (e.g. Wg ~100 µm, Dpp ~125 µm).

## When NOT to use

- The dataset is excessively large — the article notes 'the computation can become intractable when the dataset is excessively large' because cell-cell distance requires iterative optimal transport over all cell pairs (only landmark-based approximation is available)
- The signaling pathway's ligand/receptor/downstream genes are unknown — communication inference relies on known genes involved in specific signaling
- The question requires modeling time delays in communication (ligand diffusion time, intracellular cascade reacting time) — the approach explicitly does not consider them

## Inputs

- scRNA-seq expression matrix as a pandas DataFrame (rows = cells, columns = genes), e.g. dge_normalized.txt from Synapse syn15665609 (Dream Single Cell Transcriptomics Challenge)
- numpy distance matrix among spatial locations (is_dmat) from in situ / seqFISH+ spatial data
- numpy dissimilarity matrix within single-cell data (sc_dmat)
- numpy cost matrix between single cells and spatial positions (cost_matrix)
- Curated ligand, receptor, and downstream gene lists (e.g. ligand Wg/Wnt5, receptor fz, downstream genes CycD, dpp from Supplementary Methods section 3.1)

## Outputs

- Optimal transport plan mapping scRNA-seq cells to spatial positions
- Spatial cell-cell distance matrix for all scRNA-seq cells
- Per-signaling-pathway spatial range estimates (η at maximal ligand feature importance, e.g. Wg ≈ 100 µm, Dpp ≈ 125 µm), e.g. written to signaling_ranges.csv
- signal_strengths array from infer_signal_range_ml over the supplied effect_ranges grid

## How to apply

First build a SpaOTsc spatial_sc object from the scRNA-seq DataFrame (e.g. dge_normalized.txt from Synapse syn15665609), an interspatial distance matrix is_dmat, a single-cell dissimilarity matrix sc_dmat, and a cost_matrix; call transport_plan(cost_matrix) to obtain the optimal transport map and cell_cell_distance(use_landmark=True) to compute the spatial cell-cell distance. For each signaling system (e.g. Wg or Dpp), aggregate ligand expression over neighborhoods of varying radius η on the cell-cell distance matrix to form L̃(η), adding genes highly correlated with the downstream target as features. Fit random-forest models (scikit-learn) predicting downstream target gene expression with receptor expression as sample weights via infer_signal_range_ml, supplying a grid of candidate effect_ranges (e.g. [10,50,100]); select the η with the highest ligand feature importance as the signaling spatial distance. The rationale is that the optimal transport plan couples sender and receiver distributions so that communication inference respects both expression similarity and spatial geometry, and the feature-importance peak over η localizes the effective signaling range.

## Related tools

- **SpaOTsc** (Core method: structured optimal transport to map scRNA-seq cells to spatial positions, compute cell-cell distances, and infer signaling spatial ranges via transport_plan, cell_cell_distance, spatial_signaling_ot, and infer_signal_range_ml) — github.com/zcang/SpaOTsc
- **pandas** (Load and hold the scRNA-seq expression matrix (e.g. dge_normalized.txt) as the sc_data DataFrame passed to SpaOTsc.spatial_sc)
- **numpy** (Provide is_dmat, sc_dmat, and cost_matrix arrays consumed by the SpaOTsc transport and distance routines)
- **scikit-learn** (Random-forest regression backend used by infer_signal_range_ml to predict downstream target gene expression with receptor expression as sample weights)
- **Giotto** (Downstream spatial transcriptomics pipeline that the SpaOTsc-inferred spatial metric/annotated scRNA-seq data can be fed into)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix); spsc.cell_cell_distance(use_landmark=True); signal_strengths,_ = spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])
```

## Evaluation signals

- Recovered signaling ranges reproduce reported values of approximately 100 µm for Wg and 125 µm for Dpp in the Drosophila embryo (Fig 4c,d)
- For each ligand, the η selected as the signaling distance is the one at which the aggregated ligand feature L̃(η) attains the highest random-forest feature importance
- The transport_plan call produces a valid optimal transport plan before cell_cell_distance is computed (cell_cell_distance depends on γ*)
- Random-forest fits show receptor expression acting as meaningful sample weights (downstream target prediction improves relative to unweighted fits)

## Limitations

- Computationally intractable for excessively large datasets; cell-cell distance inference requires iterative optimal transport over all cell pairs, with only landmark-position approximation available
- The approach does not consider time delay in cell-cell communication (ligand diffusion time, intracellular cascade reacting time)
- Cell-cell communication inference relies on known genes involved in the specific signaling pathway
- No SpaOTsc software version is stated in the article

## Evidence

- [abstract] Optimal transport maps senders to receivers: [evidence span withheld — anchor status: not_found]
- [abstract] Spatial metric establishment: "A spatial metric for individual cells in scRNA-seq data is first established based on a map connecting it with the spatial measurements."
- [readme] ML signaling range API: "signal_strengths,_=spsc.infer_signal_range_ml(['Wnt5'],['fz'],['CycD','dpp'], effect_ranges=[10,50,100])"
- [readme] Cell-cell distance API: "spsc.cell_cell_distance(use_landmark=True)"
- [methods] Distance computed from transport plan: [evidence span withheld — anchor status: not_found]
- [methods] Dependency stack: "SpaOTsc depends on the following packages: pandas, numpy, scipy, networkx, python-igraph, louvain, POT, dit, astropy, scikit-learn, matplotlib, umap-learn."
