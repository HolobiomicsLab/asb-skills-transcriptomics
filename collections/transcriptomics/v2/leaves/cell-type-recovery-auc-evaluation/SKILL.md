---
name: cell-type-recovery-auc-evaluation
description: Use when comparing pathway activity scoring methods on scRNA-seq data
  where ground-truth cell labels exist (e.g., SERGIO simulations or annotated datasets),
  to measure cell-type recovery AUC and accuracy.
when_to_use_negative:
- The data are spatial transcriptomics or trajectory analyses with no ground-truth
  cell-type labels — use GSDensity's compute.spatial.kld or trajectory workflow instead
- Only a single method needs to be applied to new data (no benchmarking) — run the
  GSDensity pipeline directly rather than the comparison harness
- Data are targeted gene panels with limited gene numbers — GSDensity is explicitly
  unsuitable for such targeted gene data
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Method under evaluation; computes MCA co-embedding (compute.mca), KLD coordination
    test (compute.kld), nearest-neighbor graph (compute.nn.edges, nn.use = 300), random-walk
    propagation (run.rwr.list, restart = 0.75), and PAL binarization (compute.cell.label)
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Competing gene set scoring method in the benchmark; MCA-based hypergeometric
    test, generally ranked second best
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: ssGSEA
  role: Competing gene set scoring method benchmarked for cell-type recovery
- name: AUCell
  role: Competing gene set scoring method benchmarked for cell-type recovery
- name: VAM
  role: Competing gene set scoring method benchmarked for cell-type recovery
- name: GSDensity_manuscript_code
  role: Provides benchmark.sergio and benchmark.real functions, SERGIO simulation
    scripts, and marker curation files used to run the evaluation
  repo: https://github.com/qingnanl/GSDensity_manuscript_code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:8d230aa65b6b935412516cd68fdbf36eea5b3a2e98fb30474f05f8a37e00f10f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - AUCell
  - CelliD
  - GSDensity
  - GSDensity_manuscript_code
  - VAM
  - ssGSEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# Cell-type recovery AUC evaluation for gene set scoring methods

## Summary

Benchmark framework for evaluating how well gene set/pathway scoring methods (GSDensity vs ssGSEA, GSVA, AUCell, VAM, scGSEA, CelliD) recover known cell types, quantified as AUC and cell typing accuracy over SERGIO-simulated and real single-cell datasets.

## When to use

Use when comparing pathway activity scoring methods on scRNA-seq data where ground-truth cell labels exist (e.g., SERGIO simulations or annotated datasets), to measure cell-type recovery AUC and accuracy.

## When NOT to use

- The data are spatial transcriptomics or trajectory analyses with no ground-truth cell-type labels — use GSDensity's compute.spatial.kld or trajectory workflow instead
- Only a single method needs to be applied to new data (no benchmarking) — run the GSDensity pipeline directly rather than the comparison harness
- Data are targeted gene panels with limited gene numbers — GSDensity is explicitly unsuitable for such targeted gene data

## Inputs

- 27 SERGIO-simulated scRNA-seq datasets (3 simulation modes x 3 dropout levels, e.g. design '5000.gene.3ct', drop.out 'dp1', 3 cell types, 1500 cells/type, 5000 genes)
- 8 real-world scRNA-seq datasets with known cell labels (e.g. pbmc3k.rds with pbmc3k.markers.rds)
- Gene set collections: MSigDB GO Biological Process, Hallmarks, KEGG, BIOCARTA
- Cell-type marker files (Cell_marker_Mouse.xlsx, Cell_marker_Human.xlsx, st4.csv, PanglaoDB_markers_27_Mar_2020.tsv.gz)

## Outputs

- benchmark_results.csv with AUC and accuracy entries for all 7 methods across the 27 simulated and 8 real datasets
- Method ranking reproducing GSDensity first, CelliD typically second

## How to apply

Generate the 27 SERGIO-simulated datasets (3 simulation modes x 3 dropout levels) using the scripts in qingnanl/GSDensity_manuscript_code (sergio_steady_gen.R, run_sergio_code.py, run_sergio_dyn.1.py, Add_noise_to_data.ipynb), then call benchmark.sergio(design, version.n, drop.out, ...) per condition. For GSDensity, run the full pipeline: compute.mca co-embedding, compute.kld coordination test, compute.nn.edges with nn.use = 300, run.rwr.list with restart = 0.75, and compute.cell.label binarization. Score the same gene set collections (GO, Hallmarks, KEGG, BIOCARTA as applicable) with ssGSEA, GSVA, AUCell, VAM, scGSEA, and CelliD, using benchmark.real (e.g., fname = "pbmc3k" with .rds and markers files) for the 8 real-world datasets. Fetch the most relevant cells per method and evaluate cell-type recovery AUC and cell typing accuracy against the known labels, then aggregate across all 35 datasets and confirm the reported ordering (GSDensity first, CelliD typically second).

## Related tools

- **GSDensity** (Method under evaluation; computes MCA co-embedding (compute.mca), KLD coordination test (compute.kld), nearest-neighbor graph (compute.nn.edges, nn.use = 300), random-walk propagation (run.rwr.list, restart = 0.75), and PAL binarization (compute.cell.label)) — https://github.com/qingnanl/gsdensity
- **CelliD** (Competing gene set scoring method in the benchmark; MCA-based hypergeometric test, generally ranked second best) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **ssGSEA** (Competing gene set scoring method benchmarked for cell-type recovery)
- **AUCell** (Competing gene set scoring method benchmarked for cell-type recovery)
- **VAM** (Competing gene set scoring method benchmarked for cell-type recovery)
- **GSDensity_manuscript_code** (Provides benchmark.sergio and benchmark.real functions, SERGIO simulation scripts, and marker curation files used to run the evaluation) — https://github.com/qingnanl/GSDensity_manuscript_code

## Examples

```
data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)
```

## Evaluation signals

- benchmark_results.csv contains an AUC and accuracy entry for all 7 methods across the 27 simulated and 8 real datasets (35 datasets total)
- Aggregated ranking reproduces the reported ordering: GSDensity consistently ranks best and CelliD generally performs second best
- Each SERGIO condition is generated with the correct design parameters (e.g. design '5000.gene.3ct', version.n, drop.out 'dp1', n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)
- GSDensity runs use the published parameters: nn.use = 300 for compute.nn.edges and default restart = 0.75 for run.rwr.list
- Real-data runs consume paired data and marker files (e.g. pbmc3k.rds and pbmc3k.markers.rds) so cell-type labels are available for AUC computation

## Limitations

- MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset like hcabm40k)
- GSDensity is not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- Neighborhood size is an important parameter when constructing the cell-gene graph; results depend on nn.use
- The gsdensity version number is not stated and the CRAN install was temporarily unavailable; a specific updated CelliD build and older anticlust (e.g. 0.6.1) are required
- Installation fails on macOS with M1 chips due to a CelliD dependency (LTLA/scuttle issue)

## Evidence

- [results] In benchmarking on simulated datasets, GSDensity consistently outperformed the other six gene set scoring methods across all tested conditions.: [evidence span withheld — anchor status: not_found]
- [readme] README benchmark.sergio invocation: [evidence span withheld — anchor status: external_doc]
- [readme] README real-data benchmark invocation: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion: CelliD second best: "The application of MCA embedding in scRNA-seq studies was first reported in the CelliD12 method, which generally performed the second best in the benchmarking"
- [discussion] Methods: benchmarking used 27 SERGIO-simulated datasets: "using the 27 SERGIO-simulated datasets (Methods, Supplementary Fig. 21)"
