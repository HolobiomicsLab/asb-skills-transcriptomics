---
name: transcription-factor-binding-site-matrix-construction
description: Use when preparing FateCompass inputs from single-cell transcriptomics
  (e.g., a pre-processed AnnData from Dyngen simulation or pancreatic endocrine differentiation)
  and you must supply the TF/motif-vs-expressed-gene binding-site matrix needed for
  TF activity inference.
when_to_use_negative:
- You lack a defined set of TFs/motifs and promoter-region annotations for the organism,
  so computationally predicted regulatory sites cannot be generated
- Your goal is to capture distal (long-range) enhancer regulation — FateCompass considers
  only TF-promoter interactions, not distal regulatory sites
- The analysis goal is method benchmarking of fate-probability tools alone, with no
  need to infer TF activities from a regulatory-site matrix
tools:
- name: FateCompass
  role: Consumes the binding-site matrix (TFs/motifs as columns, expressed genes as
    rows) as its second required input to compute TF activities via the linear model
    of gene regulation; module provided in fatecompass.py
  repo: github.com/sarajimenez/fatecompass
- name: Dyngen
  role: Simulates the ground-truth in silico dataset (1000 cells, 1035 genes, 130
    TFs) from which expressed-gene rows of the binding-site matrix are derived during
    validation
  repo: dynverse/dyngen
- name: Scanpy
  role: Suggested for the basic pre-processing of the single-cell counts into an AnnData
    object whose expressed genes define the matrix rows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:4352fb9e83a1038a7aea74f931f66b0343ac80b2f220ce72e23df8842e2c3937
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - Dyngen
  - FateCompass
  - Python
  - Scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transcription-factor-binding-site-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the transcription-factor binding-site matrix that FateCompass requires as input: rows are expressed genes, columns are TFs or motifs, encoding computationally predicted regulatory sites used by the linear model of gene regulation to infer TF activities.

## When to use

Use when preparing FateCompass inputs from single-cell transcriptomics (e.g., a pre-processed AnnData from Dyngen simulation or pancreatic endocrine differentiation) and you must supply the TF/motif-vs-expressed-gene binding-site matrix needed for TF activity inference.

## When NOT to use

- You lack a defined set of TFs/motifs and promoter-region annotations for the organism, so computationally predicted regulatory sites cannot be generated
- Your goal is to capture distal (long-range) enhancer regulation — FateCompass considers only TF-promoter interactions, not distal regulatory sites
- The analysis goal is method benchmarking of fate-probability tools alone, with no need to infer TF activities from a regulatory-site matrix

## Inputs

- Pre-processed AnnData object of scRNA-seq counts with cell-type ('cluster') annotations (e.g., Dyngen simulated counts: 1000 cells, 1035 genes, 130 TFs)
- Set of transcription factors / motifs to include as matrix columns (e.g., the 130 Dyngen TFs)
- Annotation of promoter regions of expressed genes for TF binding-site (motif) prediction

## Outputs

- TF binding-site matrix with TFs/motifs as columns and expressed genes as rows (FateCompass input ii)
- Downstream per-cell TF activities and differential motif activity results inferred via the linear model of gene regulation

## How to apply

First pre-process the single-cell count matrix (e.g., Dyngen simulation of 1000 cells and 1035 genes, of which 130 are TFs) into an AnnData object with 'cluster' annotations, using Scanpy/scVelo for basic pre-processing as the FateCompass README suggests. Then build the binding-site matrix so that columns are TFs or motifs and rows are the expressed genes, based on computationally predicted regulatory sites in the promoter regions of target genes. This matrix is a required second input to the fatecompass.py module, which models observed gene expression as a linear combination of regulatory sites and TF activities. Validate the construction by confirming the gene rows match the AnnData's expressed genes and that downstream differential motif activity analysis recovers expected fate drivers (in the Dyngen ground-truth case, B11 for sEndC and B14 for sEndD).

## Related tools

- **FateCompass** (Consumes the binding-site matrix (TFs/motifs as columns, expressed genes as rows) as its second required input to compute TF activities via the linear model of gene regulation; module provided in fatecompass.py) — github.com/sarajimenez/fatecompass
- **Dyngen** (Simulates the ground-truth in silico dataset (1000 cells, 1035 genes, 130 TFs) from which expressed-gene rows of the binding-site matrix are derived during validation) — dynverse/dyngen
- **Scanpy** (Suggested for the basic pre-processing of the single-cell counts into an AnnData object whose expressed genes define the matrix rows)

## Examples

```
conda env create -f fatecompass.yml
```

## Evaluation signals

- Matrix rows exactly match the expressed genes in the pre-processed AnnData object; columns correspond to TFs/motifs (130 TFs for the Dyngen dataset)
- Running FateCompass in differentiation-potential mode on the matrix yields fate probability estimates for sEndC and sEndD consistent with ground truth
- Differential motif activity analysis on the inferred TF activities recovers B11 as the sEndC driver and B14 as the sEndD driver, matching the reported Dyngen validation
- Results (fate probabilities and driver list) saved to fatecompass_dyngen_results.csv match the reported ground-truth validation results

## Limitations

- TF activities are estimated using computationally predicted binding sites only in promoter regions; long-range (distal) regulatory interactions are not considered
- Reliance on computationally predicted regulatory sites introduces a bias on the structure of the gene-regulatory network
- Incorporating scATAC-seq-derived binding sites and appropriate regularization terms into the linear model is not straightforward

## Evidence
