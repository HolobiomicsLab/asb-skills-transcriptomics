---
name: cross-method-benchmark-aggregation
description: Use when a benchmark must compare pathway-activity/cell-type recovery
  performance of multiple gene set scoring methods across many simulated or real datasets
  into one ranked results table.
when_to_use_negative:
- Input is a spatial transcriptomics dataset where the goal is spatially relevant
  pathway identification (compute.spatial.kld) rather than cross-method cell-type
  recovery benchmarking
- Data are targeted-gene panels with limited gene numbers — GSDensity is explicitly
  not suitable for such data, so a benchmark including it would be invalid
- The analysis goal is pathway scoring speed comparison rather than cell-type recovery
  accuracy (the article benchmarks speed separately among GSDensity, CelliD, AUCell,
  ssGSEA, and VAM)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: GSDensity
  role: Method under benchmarking; run full pipeline (compute.mca, compute.kld, compute.nn.edges
    nn.use=300, run.rwr.list restart=0.75, compute.cell.label) per simulated dataset
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Competing gene set scoring method; generally performs second best in the benchmarking
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: ssGSEA
  role: Competing gene set scoring method scored on each simulated and real dataset
- name: AUCell
  role: Competing gene set scoring method scored on each simulated and real dataset
- name: VAM
  role: Competing gene set scoring method scored on each simulated and real dataset
- name: qingnanl/GSDensity_manuscript_code
  role: Manuscript code repository providing SERGIO simulation scripts (sergio_steady_gen.R,
    run_sergio_code.py), benchmarking functions (method.benchmarking.functions.R,
    method.benchmarking.functions.rd.R), and curated real-world markers
  repo: https://github.com/qingnanl/GSDensity_manuscript_code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:a074edb4ddf859baf9188a04b73b88beaa4e2366502cb3200397b171a7731f5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - AUCell
  - CelliD
  - GSDensity
  - VAM
  - qingnanl/GSDensity_manuscript_code
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# cross-method-benchmark-aggregation

## Summary

Aggregates cell-type recovery AUC and cell typing accuracy across seven gene set scoring methods (GSDensity, ssGSEA, GSVA, AUCell, VAM, scGSEA, CelliD) over 27 SERGIO-simulated and 8 real-world scRNA-seq datasets, producing a single ranked comparison table (benchmark_results.csv) that should reproduce GSDensity first, CelliD typically second.

## When to use

Use when a benchmark must compare pathway-activity/cell-type recovery performance of multiple gene set scoring methods across many simulated or real datasets into one ranked results table.

## When NOT to use

- Input is a spatial transcriptomics dataset where the goal is spatially relevant pathway identification (compute.spatial.kld) rather than cross-method cell-type recovery benchmarking
- Data are targeted-gene panels with limited gene numbers — GSDensity is explicitly not suitable for such data, so a benchmark including it would be invalid
- The analysis goal is pathway scoring speed comparison rather than cell-type recovery accuracy (the article benchmarks speed separately among GSDensity, CelliD, AUCell, ssGSEA, and VAM)

## Inputs

- 27 SERGIO-simulated scRNA-seq count matrices (3 simulation modes x 3 dropout levels, e.g. 5000 genes, 3 cell types, 1500 cells per type)
- 8 real-world scRNA-seq datasets (e.g. pbmc3k.rds) with curated marker files (e.g. Cell_marker_Human.xlsx, PanglaoDB_markers_27_Mar_2020.tsv.gz)
- Gene set collections: GO (1160 GO Biological Process sets), MSigDB Hallmarks, KEGG, BIOCARTA
- Known cell-type labels for simulated datasets and marker-based labels for real datasets

## Outputs

- Per-method AUC and accuracy values for cell-type recovery on each of the 35 datasets
- Aggregated comparison table written to benchmark_results.csv
- Method ranking with GSDensity first and CelliD typically second

## How to apply

Score the same gene set collections (GO, Hallmarks, KEGG, BIOCARTA as applicable) on each dataset with all seven methods: run the GSDensity pipeline (compute.mca, compute.kld, compute.nn.edges with nn.use = 300, run.rwr.list with restart = 0.75, compute.cell.label) and the six competitors (ssGSEA, GSVA, AUCell, VAM, scGSEA, CelliD). For each method-dataset pair, fetch the most relevant cells and compute cell-type recovery AUC and cell typing accuracy against known labels — for the 27 SERGIO-simulated datasets (3 simulation modes x 3 dropout levels, e.g. design '5000.gene.3ct', drop.out 'dp1', 3 cell types, 1500 cells per type, 5000 genes) and the 8 real-world datasets (e.g. pbmc3k with marker files). Aggregate AUC and accuracy across all 35 datasets per method into a single table and verify the reported ordering holds: GSDensity consistently ranks first and CelliD generally performs second best. The workflow is executed with the helper functions sergio_analysis_functions.R and method.benchmarking.functions.R from qingnanl/GSDensity_manuscript_code, and real-data benchmarking with method.benchmarking.functions.rd.R via benchmark.real(fname = 'pbmc3k').

## Related tools

- **GSDensity** (Method under benchmarking; run full pipeline (compute.mca, compute.kld, compute.nn.edges nn.use=300, run.rwr.list restart=0.75, compute.cell.label) per simulated dataset) — https://github.com/qingnanl/gsdensity
- **CelliD** (Competing gene set scoring method; generally performs second best in the benchmarking) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **ssGSEA** (Competing gene set scoring method scored on each simulated and real dataset)
- **AUCell** (Competing gene set scoring method scored on each simulated and real dataset)
- **VAM** (Competing gene set scoring method scored on each simulated and real dataset)
- **qingnanl/GSDensity_manuscript_code** (Manuscript code repository providing SERGIO simulation scripts (sergio_steady_gen.R, run_sergio_code.py), benchmarking functions (method.benchmarking.functions.R, method.benchmarking.functions.rd.R), and curated real-world markers) — https://github.com/qingnanl/GSDensity_manuscript_code

## Examples

```
data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)
```

## Evaluation signals

- benchmark_results.csv contains an AUC and accuracy entry for all 7 methods across all 27 simulated and 8 real datasets (35 total)
- Resulting ranking reproduces the reported ordering: GSDensity consistently ranks best and CelliD typically second best
- Simulated datasets cover the full 3 simulation modes x 3 dropout levels factorial design (e.g. drop.out = 'dp1')
- Real-data benchmark runs complete via benchmark.real(fname = 'pbmc3k') with both pbmc3k.rds and pbmc3k.markers.rds present in the working directory
- AUC and accuracy values are comparable in magnitude across methods and computed against the same known cell-type labels

## Limitations

- CelliD depends on a scuttle version with known install issues (fails on macOS M1; the updated Bioconductor CelliD version is required)
- The benchmark ranking depends on the neighborhood size parameter (nn.use = 300); neighborhood size is an important parameter when constructing the cell-gene graph
- GSDensity's MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset), limiting benchmarkable dataset sizes
- The gsdensity stable CRAN install was temporarily unavailable; the GitHub development version and an older 'anticlust' version (e.g. 0.6.1) must be used

## Evidence

- [results] In benchmarking on simulated datasets, GSDensity consistently outperformed the other six gene set scoring methods across all tested conditions.: [evidence span withheld — anchor status: not_found]
- [discussion] using the 27 SERGIO-simulated datasets: "using the 27 SERGIO-simulated datasets (Methods, Supplementary Fig. 21)"
- [discussion] The application of MCA embedding in scRNA-seq studies was first reported in the CelliD12 method, which generally performed the second best in the benchmarking: "The application of MCA embedding in scRNA-seq studies was first reported in the CelliD12 method, which generally performed the second best in the benchmarking"
- [readme] el <- compute.nn.edges(coembed = ce, nn.use = 300): "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [readme] data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1",: "data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", 
                             n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)"
