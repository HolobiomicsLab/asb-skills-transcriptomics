---
name: supplementary-figure-reproduction-from-manuscript-code
description: Use when a manuscript ships a code/data repository (GitHub + Zenodo)
  and a target supplementary figure must be reproduced by executing the authors' scripts
  and comparing outputs to the paper's reported values.
when_to_use_negative:
- The goal is to re-evaluate method predictive performance (AUROC/AUPRC on perturbation
  experiments) via the decoupleRBench benchmark rather than reproduce a specific figure's
  quantitative output.
- No authors' code/data release exists for the manuscript, so the figure can only
  be reimplemented from the methods text rather than reproduced from released scripts.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: decoupleR
  role: R/Bioconductor implementation whose 11 methods are timed in the scalability
    benchmark
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupler-py
  role: Python implementation whose methods are timed for the R-vs-Python runtime
    comparison
  repo: https://github.com/saezlab/decoupler-py
- name: decoupleRBench
  role: R package built by the authors to benchmark decoupleR; provides the benchmark
    pipeline machinery
  repo: https://github.com/saezlab/decoupleRBench
- name: decoupleR_manuscript
  role: Repository holding the reproduction scripts (get_data.R, get_simmulated_data.R,
    run_scale_bench.R, supp_fig_7.R)
  repo: https://github.com/saezlab/decoupleR_manuscript
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:17c6f97eb72ccc9fa5eec371fb467415a795432a20796e184ef3a319e135c04f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  - decoupleR_manuscript
  - decoupler-py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# supplementary-figure-reproduction-from-manuscript-code

## Summary

Reproduces a supplementary figure from a published manuscript by re-running the authors' released analysis code end-to-end (data download, processing scripts, benchmark execution) and tabulating the results in the paper's own reporting units, here the decoupleR runtime benchmark behind Supplementary Fig. S7.

## When to use

Use when a manuscript ships a code/data repository (GitHub + Zenodo) and a target supplementary figure must be reproduced by executing the authors' scripts and comparing outputs to the paper's reported values.

## When NOT to use

- The goal is to re-evaluate method predictive performance (AUROC/AUPRC on perturbation experiments) via the decoupleRBench benchmark rather than reproduce a specific figure's quantitative output.
- No authors' code/data release exists for the manuscript, so the figure can only be reimplemented from the methods text rather than reproduced from released scripts.

## Inputs

- Data from the Zenodo record https://zenodo.org/record/5645208 referenced by the manuscript
- decoupleR_manuscript repository code (R/ and run_scale_bench.R scripts)
- decoupleRBench R package (installed via devtools::install_github('saezlab/decoupleRBench'))
- Simulated omics count matrix produced by R/process/get_simmulated_data.R

## Outputs

- runtime_table.csv with median per-sample-and-regulator runtimes per method per language implementation
- Comparison table reproducing the structure of Supplementary Fig. S7

## How to apply

Retrieve the data with the manuscript's retrieval script, then run the specific analysis pipeline for the target figure — for the decoupleR scalability analysis this means generating the simulated omics dataset with 'Rscript R/process/get_simmulated_data.R' from the decoupleR_manuscript repository, then executing 'bash run_scale_bench.R' to time each of decoupleR's 11 methods in both the R (Bioconductor) and Python implementations on the simulated matrices. Follow the paper's reporting convention exactly: compute the median runtime per sample and regulator (ms/sample/regulator) on the reference hardware (Intel(R) Core(TM) i7-8550U CPU @ 1.80 GHz), and assemble a comparison table (runtime_table.csv) mirroring the structure of Supplementary Fig. S7. Validate by checking that computed medians are on the order of the published values (1.44 ms in R, 0.44 ms in Python) rather than matching exactly, since absolute timings are hardware-dependent. Where the manuscript relies on a separate benchmark package (decoupleRBench, installed via devtools::install_github('saezlab/decoupleRBench')), install it before running the pipeline.

## Related tools

- **decoupleR** (R/Bioconductor implementation whose 11 methods are timed in the scalability benchmark) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupler-py** (Python implementation whose methods are timed for the R-vs-Python runtime comparison) — https://github.com/saezlab/decoupler-py
- **decoupleRBench** (R package built by the authors to benchmark decoupleR; provides the benchmark pipeline machinery) — https://github.com/saezlab/decoupleRBench
- **decoupleR_manuscript** (Repository holding the reproduction scripts (get_data.R, get_simmulated_data.R, run_scale_bench.R, supp_fig_7.R)) — https://github.com/saezlab/decoupleR_manuscript

## Examples

```
Rscript R/process/get_simmulated_data.R; bash run_scale_bench.R
```

## Evaluation signals

- Computed median runtime across the 11 methods is on the order of 1.44 ms per sample and regulator in R and 0.44 ms in Python (allowing hardware-dependent deviation).
- runtime_table.csv reproduces the structure of Supplementary Fig. S7 (per-method, per-language runtime comparison).
- All reported timings use the paper's convention of ms/sample/regulator on an Intel(R) Core(TM) i7-8550U CPU @ 1.80 GHz.
- The simulated dataset was generated successfully by R/process/get_simmulated_data.R before the benchmark was run.
- Zenodo data retrieval (Rscript R/process/get_data.R) completed so the benchmark has all required inputs.

## Limitations

- Absolute runtime values are hardware-dependent; expect only order-of-magnitude agreement with the published 1.44 ms (R) / 0.44 ms (Python) medians on CPUs other than the i7-8550U.
- The scalability analysis is explicitly flagged as long-running ('This might take a while') in the manuscript README.
- The Python decoupler package has since moved to the scverse organization and the R version is deprecated, so current releases may not match the manuscript's timing behavior.

## Evidence

- [abstract] The code to reproduce the results is in GitHub (https://github.com/saezlab/decoupleR_manuscript): "The code to reproduce the results is in GitHub (https://github.com/saezlab/decoupleR_manuscript)"
- [readme] README analysis commands: "# Run scalablity analysis (This might take a while)
Rscript R/process/get_simmulated_data.R
bash run_scale_bench.R"
- [readme] README data retrieval: "To retrieve all the necessary data, please run:
```
Rscript R/process/get_data.R
```"
- [readme] decoupleRBench role: [evidence span withheld — anchor status: not_found]
