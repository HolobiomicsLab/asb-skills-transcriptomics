---
name: pathway-activity-scoring-across-methods
description: Use when you need to compare how well competing pathway activity scoring
  methods recover known cell types from scRNA-seq data, or to benchmark GSDensity
  against ssGSEA/GSVA/AUCell/VAM/scGSEA/CelliD on labeled or simulated datasets.
when_to_use_negative:
- The data are targeted gene panels with limited numbers of genes — GSDensity is not
  suitable there and imputation of unmeasured genes would be needed
- The goal is only pathway scoring speed on one dataset rather than cell-type recovery
  benchmarking (a separate five-method speed comparison covers GSDensity, CelliD,
  AUCell, ssGSEA, VAM)
- No known cell-type labels or simulated ground truth exist, so recovery AUC/accuracy
  cannot be computed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: GSDensity
  role: Method under test; full pipeline (compute.mca, compute.kld, compute.nn.edges,
    run.rwr.list, compute.cell.label) for pathway activity scoring and cell fetch
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Competing gene set scoring method; updated Bioconductor version required for
    benchmarking
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: ssGSEA
  role: Competing gene set scoring method benchmarked for cell-type recovery AUC and
    accuracy
- name: AUCell
  role: Competing gene set scoring method benchmarked for cell-type recovery AUC and
    accuracy
- name: VAM
  role: Competing gene set scoring method benchmarked for cell-type recovery AUC and
    accuracy
- name: GSVA
  role: Competing gene set scoring method benchmarked for cell-type recovery AUC and
    accuracy
- name: scGSEA
  role: Competing gene set scoring method benchmarked for cell-type recovery AUC and
    accuracy
- name: qingnanl/GSDensity_manuscript_code
  role: Manuscript code repository with SERGIO simulation scripts and method.benchmarking.functions.R
    / method.benchmarking.functions.rd.R benchmarking functions
  repo: https://github.com/qingnanl/GSDensity_manuscript_code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:f64af53c047b1c651371819bf1a62e0c812924a3c3bc65ba43b0286cf0190205
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - AUCell
  - CelliD
  - GSDensity
  - GSVA
  - VAM
  - qingnanl/GSDensity_manuscript_code
  - scGSEA
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# pathway-activity-scoring-across-methods

## Summary

Benchmark gene set/pathway activity scoring methods (GSDensity, ssGSEA, GSVA, AUCell, VAM, scGSEA, CelliD) for cell-type recovery on SERGIO-simulated and real scRNA-seq datasets, reproducing the GSDensity paper's Fig. 2f-i comparison.

## When to use

Use when you need to compare how well competing pathway activity scoring methods recover known cell types from scRNA-seq data, or to benchmark GSDensity against ssGSEA/GSVA/AUCell/VAM/scGSEA/CelliD on labeled or simulated datasets.

## When NOT to use

- The data are targeted gene panels with limited numbers of genes — GSDensity is not suitable there and imputation of unmeasured genes would be needed
- The goal is only pathway scoring speed on one dataset rather than cell-type recovery benchmarking (a separate five-method speed comparison covers GSDensity, CelliD, AUCell, ssGSEA, VAM)
- No known cell-type labels or simulated ground truth exist, so recovery AUC/accuracy cannot be computed

## Inputs

- 27 SERGIO-simulated scRNA-seq datasets (3 simulation modes x 3 dropout levels, e.g. design '5000.gene.3ct', drop.out 'dp1', 1500 cells per type, 5000 genes)
- 8 real-world scRNA-seq datasets with known cell labels (e.g. pbmc3k.rds plus pbmc3k.markers.rds)
- Marker/gene set collections: GO Biological Process (1160 gene sets), MSigDB Hallmarks, KEGG, BIOCARTA
- Marker files such as Cell_marker_Mouse.xlsx, Cell_marker_Human.xlsx, st4.csv, PanglaoDB_markers_27_Mar_2020.tsv.gz

## Outputs

- benchmark_results.csv with cell-type recovery AUC and cell typing accuracy for all 7 methods across 27 simulated and 8 real datasets
- Method ranking with GSDensity first and CelliD generally second best

## How to apply

Generate 27 SERGIO-simulated datasets (3 simulation modes x 3 dropout levels) using the scripts in qingnanl/GSDensity_manuscript_code (sergio_steady_gen.R, run_sergio_code.py, Add_noise_to_data.ipynb). For GSDensity, run the full pipeline per dataset: co-embed cells and genes with compute.mca, test gene set coordination with compute.kld (n.grids=100, n.times=100), build the nearest-neighbor cell-gene graph with compute.nn.edges (nn.use=300), propagate with run.rwr.list (default restart=0.75), and binarize with compute.cell.label. Score the same gene set collections (GO, Hallmarks, KEGG, BIOCARTA) with ssGSEA, GSVA, AUCell, VAM, scGSEA, and CelliD (using the updated Bioconductor CelliD build) on each of the 27 simulated and 8 real datasets. Fetch the most relevant cells per method and compute cell-type recovery AUC and cell typing accuracy against known labels, then aggregate all 35 datasets per method into benchmark_results.csv and verify GSDensity ranks first and CelliD typically second.

## Related tools

- **GSDensity** (Method under test; full pipeline (compute.mca, compute.kld, compute.nn.edges, run.rwr.list, compute.cell.label) for pathway activity scoring and cell fetch) — https://github.com/qingnanl/gsdensity
- **CelliD** (Competing gene set scoring method; updated Bioconductor version required for benchmarking) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **ssGSEA** (Competing gene set scoring method benchmarked for cell-type recovery AUC and accuracy)
- **AUCell** (Competing gene set scoring method benchmarked for cell-type recovery AUC and accuracy)
- **VAM** (Competing gene set scoring method benchmarked for cell-type recovery AUC and accuracy)
- **GSVA** (Competing gene set scoring method benchmarked for cell-type recovery AUC and accuracy)
- **scGSEA** (Competing gene set scoring method benchmarked for cell-type recovery AUC and accuracy)
- **qingnanl/GSDensity_manuscript_code** (Manuscript code repository with SERGIO simulation scripts and method.benchmarking.functions.R / method.benchmarking.functions.rd.R benchmarking functions) — https://github.com/qingnanl/GSDensity_manuscript_code

## Examples

```
data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)
```

## Evaluation signals

- benchmark_results.csv contains an AUC and accuracy entry for all 7 methods across all 27 simulated and 8 real datasets (35 total)
- Aggregated ranking reproduces the reported ordering: GSDensity ranks best and CelliD generally performs second best
- GSDensity consistently outperforms the other six methods across all tested SERGIO simulation conditions
- SERGIO simulation parameters match the paper design, e.g. benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000) runs without error
- Nearest-neighbor graph uses nn.use = 300 and random walk restart probability defaults to 0.75 as specified in the workflow

## Limitations

- GSDensity is not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- The MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset like hcabm40k)
- Neighborhood size is an important parameter when constructing the cell-gene graph, and results may depend on it
- gsdensity install fails on macOS with M1 due to a CelliD/scuttle dependency; the anticlust dependency must be an older version (e.g., 0.6.1) and the CRAN install was temporarily unavailable

## Evidence

- [results] In benchmarking on simulated datasets, GSDensity consistently outperformed the other six gene set scoring methods across all tested conditions.: [evidence span withheld — anchor status: not_found]
- [discussion] CelliD second best in benchmarking: "The application of MCA embedding in scRNA-seq studies was first reported in the CelliD12 method, which generally performed the second best in the benchmarking"
- [readme] benchmark.sergio invocation: [evidence span withheld — anchor status: external_doc]
- [methods] compute.nn.edges parameter: "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [methods] run.rwr.list signature: "run.rwr.list(el, gene_set_list, cells, restart, parallel)"
- [methods] CelliD version requirement: "this updated version of CellID is needed: https://bioconductor.org/packages/devel/bioc/html/CelliD.html"
- [readme] real-world benchmark function: [evidence span withheld — anchor status: external_doc]
