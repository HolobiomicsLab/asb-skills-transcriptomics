---
name: single-cell-grn-driver-ranking-analysis
description: Use when evaluating whether a method's ranked driver regulator list recovers
  known cell-fate regulators (e.g., NANOG, SOX2, POU5F1) against GO ground-truth sets
  in scRNA-seq benchmarking.
when_to_use_negative:
- The goal is GRN edge inference quality rather than driver regulator ranking — use
  GRN construction benchmarks instead
- The input dataset contains multiple fate trajectories with bifurcations that must
  be processed simultaneously — CEFCON handles only one cell lineage at a time
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: CEFCON
  role: Source of the driver regulator ranking (via gene influence score and driver_regulators
    step) being benchmarked against five baselines
  repo: https://github.com/WPZgithub/CEFCON
- name: Python
  role: Implementation language for computing top-k precision against ground-truth
    gene sets and for running CEFCON
  repo: https://github.com/WPZgithub/CEFCON
- name: matplotlib
  role: Optional dependency used to plot the aggregated top-k precision curves analogous
    to Fig. 4a-h
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:ed768a3f78fe21d533f9b724eb70dbd6507a1725fc93576b6e1ef0ca8f6ed503
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-grn-driver-ranking-analysis

## Summary

Reproduce and benchmark CEFCON's driver regulator identification by computing top-k (k=1-20) precision of ranked driver lists from CEFCON and five baselines (VIPER, ANANSE, SCENIC, CellRouter, CellOracle) against GO term and literature-curated ground-truth gene sets on the mESC and hESC datasets.

## When to use

Use when evaluating whether a method's ranked driver regulator list recovers known cell-fate regulators (e.g., NANOG, SOX2, POU5F1) against GO ground-truth sets in scRNA-seq benchmarking.

## When NOT to use

- The goal is GRN edge inference quality rather than driver regulator ranking — use GRN construction benchmarks instead
- The input dataset contains multiple fate trajectories with bifurcations that must be processed simultaneously — CEFCON handles only one cell lineage at a time

## Inputs

- Driver regulator ranking lists with influence scores for CEFCON, VIPER, ANANSE, SCENIC, CellRouter, and CellOracle (hESC dataset, from Zenodo record 7564872)
- Ground-truth positive gene sets from GO terms GO:0045165, GO:0019827, GO:0007492
- Literature-curated driver gene sets

## Outputs

- Top-k precision table (k=1..20) for all six methods against all four ground-truth gene sets
- Top-k precision curve plots (analogous to Fig. 4a-h) rendered with matplotlib
- Verified top-20 driver regulator ranking for CEFCON on hESC

## How to apply

Download the driver regulator ranking lists with influence scores for CEFCON and the five baselines from the Zenodo record (zenodo.org/record/7564872) for the hESC dataset. Assemble ground-truth positive sets from GO terms GO:0045165 (cell fate commitment), GO:0019827 (stem cell population maintenance), GO:0007492 (endoderm development) plus literature-curated driver genes. In Python, compute top-k precision for k=1..20 per method per ground-truth set by intersecting each ranked top-k list with the positive set. Verify CEFCON's reported hESC ranks: NANOG at rank 1, SOX2 at rank 2, POU5F1 at rank 5 within the top 20. Aggregate precision curves into a single table and plot with matplotlib, analogous to Fig. 4a-h.

## Related tools

- **CEFCON** (Source of the driver regulator ranking (via gene influence score and driver_regulators step) being benchmarked against five baselines) — https://github.com/WPZgithub/CEFCON
- **Python** (Implementation language for computing top-k precision against ground-truth gene sets and for running CEFCON) — https://github.com/WPZgithub/CEFCON
- **matplotlib** (Optional dependency used to plot the aggregated top-k precision curves analogous to Fig. 4a-h)

## Examples

```
cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8); cefcon_results.gene_influence_score(); cefcon_results.driver_regulators()
```

## Evaluation signals

- Reproduced CEFCON hESC ranking places NANOG at rank 1, SOX2 at rank 2, and POU5F1 at rank 5 within the top-20 driver regulators
- Top-k precision table contains values for all six methods (CEFCON, VIPER, ANANSE, SCENIC, CellRouter, CellOracle) and all four ground-truth sets
- CEFCON's precision curves match or exceed all five baselines, with highest-ranked genes almost perfectly identified for all four ground-truth gene sets
- All input ranking lists load with influence scores and cover the same gene universe as the hESC dataset (1,000 highly variable genes)

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- CEFCON only considers intracellular factors; extrinsic factors such as cell-cell communication and spatial location are not modeled
- Ground-truth label datasets for supervised alternatives are often lacking, especially for specific cell types or lineages

## Evidence

- [results] CEFCON achieved superior performance on driver regulator identification over existing methods (VIPER, ANANSE, SCENIC, CellRouter, CellOracle): [evidence span withheld — anchor status: not_found]
- [methods] identify driver regulators via cefcon_results.driver_regulators(): "# Identify dirver regulators
cefcon_results.driver_regulators()"
- [methods] gene influence score computation precedes driver regulator ranking: "cefcon_results.gene_influence_score()"
- [methods] All the input data mentioned in the paper can be downloaded from the Zenodo record: "All the input data mentioned in the paper can be downloaded from [here](https://zenodo.org/record/7564872)"
