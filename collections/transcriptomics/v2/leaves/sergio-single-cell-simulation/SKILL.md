---
name: sergio-single-cell-simulation
description: Use when a gene set scoring or cell-type recovery method needs benchmarking
  on simulated scRNA-seq data with known cell type labels and tunable dropout/noise
  conditions.
when_to_use_negative:
- Targeted gene data with limited numbers of genes, since GSDensity (the downstream
  scoring method benchmarked here) is not suitable for such data
- Analyses where real-world labeled datasets are already available, as simulation
  is only needed to obtain controlled ground truth across dropout conditions
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SERGIO
  role: Simulator that generates synthetic single-cell gene expression data in steady
    state and dynamic modes with tunable noise and dropout for benchmarking
- name: GSDensity
  role: Method under benchmark whose MCA/KLD/RWR pipeline is evaluated on the simulated
    datasets against ground-truth cell labels
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Competing gene set scoring method benchmarked on the SERGIO-simulated datasets
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: AUCell
  role: Competing gene set scoring method benchmarked on the SERGIO-simulated datasets
- name: ssGSEA
  role: Competing gene set scoring method benchmarked on the SERGIO-simulated datasets
- name: VAM
  role: Competing gene set scoring method benchmarked on the SERGIO-simulated datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:54356541d2e71dd7e0d91d804c0cdbfa52bb765952f4b617b11817282e1dbec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - AUCell
  - CelliD
  - GSDensity
  - SERGIO
  - VAM
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# sergio-single-cell-simulation

## Summary

Generates synthetic single-cell RNA-seq datasets with SERGIO under controlled simulation modes (steady state, dynamic), cell-state migration matrices, and dropout/noise levels, to create labeled benchmark data for evaluating gene set scoring methods against known ground-truth cell type labels.

## When to use

Use when a gene set scoring or cell-type recovery method needs benchmarking on simulated scRNA-seq data with known cell type labels and tunable dropout/noise conditions.

## When NOT to use

- Targeted gene data with limited numbers of genes, since GSDensity (the downstream scoring method benchmarked here) is not suitable for such data
- Analyses where real-world labeled datasets are already available, as simulation is only needed to obtain controlled ground truth across dropout conditions

## Inputs

- SERGIO simulation configuration (design string, e.g. '5000.gene.3ct', version number, dropout level, number of cell types, cells per type, number of genes)
- Cell state migration matrices for dynamic mode (bMat_cID7.tab, bMat_cID10.mod.tab)
- Random gene matrix combined with simulated gene matrix (via normalize_sim_rand.R)

## Outputs

- 27 SERGIO-simulated scRNA-seq datasets (3 simulation modes x 3 dropout levels) with known cell type labels
- Per-method cell-type recovery AUC and accuracy values aggregated across simulated datasets

## How to apply

Generate SERGIO input files from R scripts (sergio_steady_gen.R for steady state, sergio_dyn_gen.R for dynamic mode with bMat_cID7.tab / bMat_cID10.mod.tab cell state migration files), run SERGIO via run_sergio_code.py or run_sergio_dyn.1.py, combine the random gene matrix with the simulated gene matrix using normalize_sim_rand.R (dynamic mode only), and add noise with the Add_noise_to_data.ipynb notebook. In the GSDensity benchmark this produced 27 simulated datasets (3 simulation modes x 3 dropout levels) using designs such as '5000.gene.3ct' with 3 cell types, 1500 cells per type, and 5000 genes. Each simulated dataset is then scored with GSDensity and six competing methods (ssGSEA, GSVA, AUCell, VAM, scGSEA, CelliD), and cell-type recovery AUC and accuracy are computed against the known simulated labels. Rationale: simulation provides ground-truth labels that real scRNA-seq data lack, allowing controlled assessment of method performance across sparsity conditions.

## Related tools

- **SERGIO** (Simulator that generates synthetic single-cell gene expression data in steady state and dynamic modes with tunable noise and dropout for benchmarking)
- **GSDensity** (Method under benchmark whose MCA/KLD/RWR pipeline is evaluated on the simulated datasets against ground-truth cell labels) — https://github.com/qingnanl/gsdensity
- **CelliD** (Competing gene set scoring method benchmarked on the SERGIO-simulated datasets) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **AUCell** (Competing gene set scoring method benchmarked on the SERGIO-simulated datasets)
- **ssGSEA** (Competing gene set scoring method benchmarked on the SERGIO-simulated datasets)
- **VAM** (Competing gene set scoring method benchmarked on the SERGIO-simulated datasets)

## Examples

```
data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)
```

## Evaluation signals

- The simulated dataset design matches the expected configuration, e.g. benchmark.sergio(design = "5000.gene.3ct", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)
- The full simulation grid covers 27 datasets (3 simulation modes x 3 dropout levels) as used in the GSDensity benchmark
- Cell-type recovery AUC and accuracy can be computed for all 7 methods against the known simulated labels
- The resulting ranking reproduces the reported ordering: GSDensity consistently ranks first and CelliD generally second

## Limitations

- Simulation is restricted to the tested modes and dropout levels; results may not generalize to other noise structures
- The downstream scoring pipeline (GSDensity) requires large memory (~25 Gb for a 40,000-cell dataset) at the MCA embedding step
- Over- or under-correction of batch effects on simulated or real data may introduce bias requiring careful evaluation

## Evidence

- [discussion] using the 27 SERGIO-simulated datasets (Methods, Supplementary Fig. 21): "using the 27 SERGIO-simulated datasets (Methods, Supplementary Fig. 21)"
- [readme] Simulation scRNA-seq data with SERGIO: sergio_steady_gen.R: generate SERGIO input file (steady state mode): [evidence span withheld — anchor status: external_doc]
- [readme] data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000): [evidence span withheld — anchor status: external_doc]
- [results] GSDensity consistently outperformed the other six gene set scoring methods across all tested conditions.: [evidence span withheld — anchor status: not_found]
- [readme] normalize_sim_rand.R: combine random gene matrix with simulated gene matrix (only dynamic mode): [evidence span withheld — anchor status: external_doc]
