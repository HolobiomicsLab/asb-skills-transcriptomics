---
name: single-cell-rnaseq-data-loading-and-qc
description: Use when starting from a 10x Chromium scRNA-seq matrix processed with
  Cell Ranger v6.0.1 and it must be prepared as input for CellOracle GRN inference/simulation
  or Capybara cell identity classification.
when_to_use_negative:
- Input is raw FASTQ from the 10x Chromium platform that has not yet been processed
  by the Cell Ranger v6.0.1 pipeline
- Analysis goal is bulk transcriptome deconvolution only — Capybara's bulk-QP step
  expects bulk references mined from ARCHS4, but this skill targets single-cell matrix
  loading
- No sample-specific GRN/Links object exists for the condition, since CellOracle simulation
  requires per-state network configurations
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Python framework whose Oracle objects load the single-cell matrix and sample-specific
    Links (GRN) prior to in silico KO/OE simulation
  repo: https://github.com/morris-lab/CellOracle
- name: Cell Ranger 6.0.1
  role: Upstream preprocessing of 10x Chromium data that produces the count matrices
    this skill ingests
- name: Capybara
  role: R package that receives the loaded matrix plus metadata to score cell identity
    as a continuum at single-cell resolution
  repo: https://github.com/morris-lab/Capybara
- name: igraph
  role: R library dependency checked during CellOracle installation that supports
    network handling
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:64fb5a0459d4d2f52813d9be310942614ad5576e011da06d502a2c25eafdb011
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - Capybara
  - Cell Ranger
  - Cell Ranger 6.0.1
  - CellOracle
  - igraph
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-rnaseq-data-loading-and-qc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load Cell Ranger-processed single-cell RNA-seq count matrices (e.g. day-20 Perturb-seq of MEF-to-iEP reprogramming) into CellOracle Oracle objects and Capybara-compatible matrices for downstream GRN simulation and identity scoring, with QC of sample-specific GRN Links.

## When to use

Use when starting from a 10x Chromium scRNA-seq matrix processed with Cell Ranger v6.0.1 and it must be prepared as input for CellOracle GRN inference/simulation or Capybara cell identity classification.

## When NOT to use

- Input is raw FASTQ from the 10x Chromium platform that has not yet been processed by the Cell Ranger v6.0.1 pipeline
- Analysis goal is bulk transcriptome deconvolution only — Capybara's bulk-QP step expects bulk references mined from ARCHS4, but this skill targets single-cell matrix loading
- No sample-specific GRN/Links object exists for the condition, since CellOracle simulation requires per-state network configurations

## Inputs

- Cell Ranger v6.0.1-processed 10x Chromium single-cell count matrix (genes x cells), e.g. day-20 Perturb-seq scRNA-seq
- Sample-specific CellOracle Links (GRN) objects for each state (reprogramming, long-term-iEP)
- Cell-level metadata (cell type / annotation and barcode columns) for Capybara classification
- Single-cell reference profiles for fibroblast and iEP identity scoring

## Outputs

- CellOracle Oracle objects with loaded count data and per-sample GRN Links, ready for in silico perturbation
- Capybara-format gene-by-cell count matrix plus metadata for quadratic-programming classification
- Per-cell and per-condition identity score summaries (e.g. identity_scores.csv)

## How to apply

Start from the Cell Ranger v6.0.1 output matrix generated from 10x Chromium data (e.g. the day-20 Perturb-seq scRNA-seq matrix), together with sample-specific CellOracle Links (GRN) objects for each state, such as the reprogramming and long-term-iEP conditions. Load the count matrix (genes as rows, cells as columns) plus cell metadata and construct CellOracle Oracle objects, attaching the inferred GRN Links per sample. For Capybara-based identity scoring, keep the matrix in raw-count form with an accompanying metadata file whose columns encode cell type and barcode, following the Capybara data layout. QC/decision points: confirm the R igraph dependency is installed for CellOracle (tested with Python 3.8 and 3.10), confirm Capybara's Bioconductor dependencies (limma, impute, AnnotationDbi, GO.db, preprocessCore), and verify each state has a matching Links/GRN configuration before running in silico perturbation. Correct application yields an Oracle object per condition and a cell-by-gene matrix ready for identity scoring, aggregatable to per-condition summaries such as identity_scores.csv.

## Related tools

- **CellOracle** (Python framework whose Oracle objects load the single-cell matrix and sample-specific Links (GRN) prior to in silico KO/OE simulation) — https://github.com/morris-lab/CellOracle
- **Cell Ranger 6.0.1** (Upstream preprocessing of 10x Chromium data that produces the count matrices this skill ingests)
- **Capybara** (R package that receives the loaded matrix plus metadata to score cell identity as a continuum at single-cell resolution) — https://github.com/morris-lab/Capybara
- **igraph** (R library dependency checked during CellOracle installation that supports network handling) — https://github.com/igraph/rigraph

## Examples

```
library(devtools); devtools::install_github("morris-lab/Capybara"); library(Capybara)
```

## Evaluation signals

- Each condition's count matrix loads into an Oracle object with a matching sample-specific Links (GRN) object; a missing per-state GRN is a load failure
- Gene-by-cell matrix dimensions and barcodes are preserved from the Cell Ranger v6.0.1 output (no duplicated barcodes, genes as rows)
- Capybara input satisfies its schema: metadata with cell.type and barcode columns, matrix in raw counts
- Loaded data supports the expected downstream sanity checks: simulated Fos KO displaces cells away from the iEP attractor and Hnf4α-Foxa1 + Fos + Yap1 yields higher iEP identity scores than the cocktail lacking Fos/Yap1
- Dependency checks pass: CellOracle reports 'checking R library installation: igraph -> OK' and Capybara's Bioconductor packages (limma, impute, AnnotationDbi, GO.db, preprocessCore) install

## Limitations

- Matrix must originate from a compatible upstream pipeline — Cell Ranger v6.0.1 on 10x Chromium data — and from the organism matching the Base-GRN (Mouse sci-ATAC-seq Atlas for mouse data)
- Capybara depends on R >= 3.5.0, and its MCA/QP workflow may differ across versions due to threshold-selection variations as noted in the reproducibility notebooks
- CellOracle requires per-state GRN configurations; a single global network is insufficient for state-specific simulation

## Evidence

- [readme] Capybara README purpose: [evidence span withheld — anchor status: external_doc]
