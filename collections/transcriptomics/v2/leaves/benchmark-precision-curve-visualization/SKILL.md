---
name: benchmark-precision-curve-visualization
description: Use when you have ranked driver-regulator lists (with influence scores)
  from multiple methods and curated ground-truth gene sets (e.g. GO:0045165, GO:0019827,
  GO:0007492) and need to compute/plot top-k precision curves for benchmarking.
when_to_use_negative:
- 'The analysis goal is to run CEFCON itself on a new lineage: CEFCON can only handle
  one cell lineage at a time and cannot process multiple fate trajectories with bifurcations
  simultaneously'
- Input is raw scRNA-seq data rather than pre-computed ranked driver-regulator lists
  with influence scores — use the GRN-inference skill instead
- No ground-truth gene sets (GO terms or curated driver genes) are available for the
  cell type or lineage of interest
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: CEFCON
  role: Source of the driver regulator rankings under evaluation; its influence scores
    produce the ranked lists benchmarked against baselines
  repo: https://github.com/WPZgithub/CEFCON
- name: Python
  role: Environment for computing top-k precision by intersecting ranked top-k driver
    lists with GO ground-truth sets
- name: matplotlib
  role: Plotting library (>=3.5.3) used to render the top-k precision curves analogous
    to Fig. 4a-h
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:65499118e62a0acc0614745acdae3c2d7e92377a7973b9b420642f629cf60617
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benchmark-precision-curve-visualization

## Summary

Reproduce and visualize top-k (k=1–20) precision benchmarks for driver regulator identification, comparing a graph-neural-network method (CEFCON) against baselines such as VIPER, ANANSE, SCENIC, CellRouter, and CellOracle on mESC and hESC scRNA-seq datasets using GO-term ground-truth gene sets.

## When to use

Use when you have ranked driver-regulator lists (with influence scores) from multiple methods and curated ground-truth gene sets (e.g. GO:0045165, GO:0019827, GO:0007492) and need to compute/plot top-k precision curves for benchmarking.

## When NOT to use

- The analysis goal is to run CEFCON itself on a new lineage: CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Input is raw scRNA-seq data rather than pre-computed ranked driver-regulator lists with influence scores — use the GRN-inference skill instead
- No ground-truth gene sets (GO terms or curated driver genes) are available for the cell type or lineage of interest

## Inputs

- Ranked driver-regulator lists with influence scores for CEFCON and baseline methods (VIPER, ANANSE, SCENIC, CellRouter, CellOracle) from the Zenodo record
- Ground-truth positive gene sets from GO terms GO:0045165, GO:0019827, GO:0007492
- Literature-curated driver gene sets
- Python environment (3.9–3.10) with matplotlib (>=3.5.3), numpy, pandas

## Outputs

- Top-k precision table (k=1..20) for all six methods against all four ground-truth gene sets
- Top-k precision curve plots analogous to Fig. 4a-h
- Verified top-20 driver regulator ranking (e.g. hESC: NANOG rank 1, SOX2 rank 2, POU5F1 rank 5)

## How to apply

Download the driver regulator ranking lists for all methods from the Zenodo record (zenodo.org/record/7564872) and assemble ground-truth positive gene sets from GO terms GO:0045165 (cell fate commitment), GO:0019827 (stem cell population maintenance), GO:0007492 (endoderm development), plus literature-curated driver genes. In Python, for each method and each ground-truth set, intersect the ranked top-k driver list (k=1..20) with the positive set to compute top-k precision, then aggregate all methods and all four ground-truth sets into a single precision table. Plot the top-k precision curves with matplotlib (analogous to Fig. 4a-h in the paper). Validate the CEFCON hESC ranking by confirming NANOG at rank 1, SOX2 at rank 2, and POU5F1 at rank 5 within the top 20 driver regulators; the environment requires Python 3.9–3.10 with matplotlib >= 3.5.3 for visualization.

## Related tools

- **CEFCON** (Source of the driver regulator rankings under evaluation; its influence scores produce the ranked lists benchmarked against baselines) — https://github.com/WPZgithub/CEFCON
- **Python** (Environment for computing top-k precision by intersecting ranked top-k driver lists with GO ground-truth sets)
- **matplotlib** (Plotting library (>=3.5.3) used to render the top-k precision curves analogous to Fig. 4a-h)

## Examples

```
import pandas as pd, matplotlib.pyplot as plt; prec = [len(set(df.gene.head(k)) & truth)/k for k in range(1,21)]; plt.plot(range(1,21), prec)
```

## Evaluation signals

- The reproduced CEFCON hESC top-20 ranking places NANOG at rank 1, SOX2 at rank 2, and POU5F1 at rank 5
- The precision table contains entries for all six methods (CEFCON plus VIPER, ANANSE, SCENIC, CellRouter, CellOracle) across all four ground-truth gene sets
- Precision values are monotone-consistent with ranking: top-k precision for CEFCON matches or exceeds all baselines at essentially every k in 1..20
- Plotted curves visually reproduce the pattern of Fig. 4a-h, with the highest-ranked genes almost perfectly identified for all four ground-truth sets

## Limitations

- Benchmark validity depends on ground-truth gene sets; label datasets serving as ground-truths are often lacking, especially for specific cell types or lineages
- Supervised frameworks such as GENELink may yield superior results but require ground-truth labels that are unavailable here
- Results are specific to the mESC and hESC datasets and the four chosen GO-term/curated ground-truth sets; rankings may differ on other datasets

## Evidence

- [abstract] Benchmarking tests demonstrated CEFCON's superiority over baseline methods in GRN construction, driver regulator identification, and gene module identification: "Extensive benchmarking tests consistently demonstrated the superiority of CEFCON in GRN construction, driver regulator identification, and gene module identification over baseline methods"
- [methods] Zenodo data availability: "All the input data mentioned in the paper can be downloaded from [here](https://zenodo.org/record/7564872)"
- [readme] matplotlib as optional visualization dependency: "Optional (for performance evaluation, visualization and other analyses)"
- [readme] Python version requirement: "CEFCON was originally tested on Ubuntu 20.04 with Python (3.9~3.10)."
