---
name: promoter-binding-site-matrix-processing
description: Use when running FateCompass or an ISMARA-style linear model of gene
  regulation on scRNA-seq and you must build/load the TF binding-site matrix and align
  it to expressed genes in an adata object.
when_to_use_negative:
- Your regulatory sites come from distal enhancers or long-range interactions (e.g.
  scATAC-seq maps) — FateCompass only considers binding sites in promoter regions
- You want a dynamic model of TF activity over time and already have validated motif
  activities (the matrix alone does not model dynamics; it feeds the linear model)
- Input gene identifiers in the binding-site matrix cannot be mapped to the adata
  gene names — alignment will silently drop genes
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Consumes the binding-site matrix together with the adata to infer TF activities
    via the linear model of gene regulation with data-diffusion regularization; module
    provided in fatecompass.py
  repo: github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Recommended for basic pre-processing of the single-cell data object (adata)
    whose expressed genes define the matrix rows
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Python
  role: Language in which FateCompass is fully implemented; pandas loads the binding-site
    matrix and matplotlib plots the MSE-vs-t curve
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:e7e32b8ca2e8d5fbb4546bacea6cd6b9379bac2e8f6f4f4111c53a28765e7172
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - Python
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# promoter-binding-site-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares and validates the TF binding-site matrix — the FateCompass input whose columns are TFs/motifs and whose rows are expressed genes with promoter binding sites — so it aligns with a pre-processed adata object before TF activities are inferred with the linear model of gene regulation.

## When to use

Use when running FateCompass or an ISMARA-style linear model of gene regulation on scRNA-seq and you must build/load the TF binding-site matrix and align it to expressed genes in an adata object.

## When NOT to use

- Your regulatory sites come from distal enhancers or long-range interactions (e.g. scATAC-seq maps) — FateCompass only considers binding sites in promoter regions
- You want a dynamic model of TF activity over time and already have validated motif activities (the matrix alone does not model dynamics; it feeds the linear model)
- Input gene identifiers in the binding-site matrix cannot be mapped to the adata gene names — alignment will silently drop genes

## Inputs

- Pre-processed adata object (single-cell RNA-seq) with at least 'clusters' cell-type annotations, pre-processed with Scanpy
- TF binding-site matrix: columns are TFs or motifs, rows are expressed genes, with predicted promoter binding sites
- Optional RNA velocity field or differentiation potential annotation
- Candidate diffusion exponents t for cross-validated selection

## Outputs

- Aligned gene-by-TF binding-site matrix ready for the linear model of gene regulation
- Regularized TF activity matrix (exported e.g. as regularized_activity_matrix.csv)
- Cross-validated MSE-vs-t selection curve (e.g. mse_vs_t.png)

## How to apply

Construct (or load with pandas) a binding-site matrix where columns are TFs or motifs and rows are the expressed genes, based on binding sites predicted in the promoter regions of target genes. Perform basic pre-processing of the single-cell object with Scanpy (scVelo optionally for velocity) so the adata carries at least cell-type 'clusters' annotations, and restrict the matrix rows to genes actually expressed in the adata. In the FateCompass pipeline this matrix is then combined with the data-diffusion regularization (cell-cell adjacency on the kNN graph, row-normalized, raised to power t selected by cross-validated test MSE) before fitting the linear model that treats observed expression as a linear combination of regulatory sites and TF activities. Verify that the resulting TF activity matrix has one row per cell (or cell type) and one column per TF, and refit at the cross-validated t before exporting.

## Related tools

- **FateCompass** (Consumes the binding-site matrix together with the adata to infer TF activities via the linear model of gene regulation with data-diffusion regularization; module provided in fatecompass.py) — github.com/sarajimenez/fatecompass
- **Scanpy** (Recommended for basic pre-processing of the single-cell data object (adata) whose expressed genes define the matrix rows) — https://scanpy.readthedocs.io/en/stable/
- **Python** (Language in which FateCompass is fully implemented; pandas loads the binding-site matrix and matplotlib plots the MSE-vs-t curve)

## Examples

```
import pandas as pd; B = pd.read_csv('binding_sites_matrix.tsv', sep='\t', index_col=0); B = B[B.index.isin(adata.var_names)]
```

## Evaluation signals

- Matrix dimensions check: rows equal the expressed genes present in the adata, columns equal TFs/motifs; no unmapped or dropped genes silently discarded
- Split the promoter/gene set 80/20 and confirm the cross-validated test MSE has a clear minimum over candidate diffusion exponents t
- Refit the linear model at the selected t on the full data and verify the exported activity matrix has one activity value per cell and TF
- Compare inferred lineage-specific TFs against known pancreatic endocrine regulators (e.g. α-, β-, EC-specific factors such as CDX2) as a biological sanity check

## Limitations

- FateCompass relies on computationally predicted regulatory sites summarized in the binding-site matrix, which biases the structure of the gene-regulatory network
- Only TF-promoter interactions are considered; essential regulatory interactions at distal regulatory sites and long-range interactions are ignored
- Incorporating scATAC-seq-derived binding sites, as well as appropriate regularization terms to solve the linear model, may not be straightforward
- Pipeline input parameters (e.g. the diffusion exponent t) must be hand tuned / cross-validated

## Evidence
