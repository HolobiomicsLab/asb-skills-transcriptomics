---
name: gmt-gene-set-parsing
description: Use when a single-cell RNA-seq analysis must encode cells over a priori
  annotated latent variables, requiring GMT gene-set files (pathways, cell type markers,
  GRNs) to be converted into VEGA's decoder membership matrix.
when_to_use_negative:
- The goal is a data-driven latent representation without prior annotations — VEGA's
  decoder is hard-coded to user-provided GMT modules and cannot discover unannotated
  gene modules or correct prior knowledge when context requires it
- Analysis requires strong generative/predictive power — VEGA's sparse single-layer
  decoder sacrifices predictive performance for interpretability
- The input is already a gene-module activity matrix or pathway activity score table
  rather than raw/normalized single-cell expression data
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: Consumes the parsed GMT-derived membership matrix as the sparse mask wiring
    its linear decoder; implemented with PyTorch in the scvi-tools ecosystem
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset as an AnnData object whose gene space must be
    aligned with the GMT membership matrix
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scvi-tools
  role: Ecosystem in which VEGA is implemented and trained on the organoid dataset
  repo: https://scvi-tools.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:0e4b3593572503aaee1306969f0967adc43423a583699d5ac41584474d866602
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gmt-gene-set-parsing

## Summary

Parse GMT gene-set files (e.g., Reactome pathway GMTs and cell type marker GMTs) and merge them into a combined gene-module membership matrix that wires the sparse linear decoder of VEGA, defining its interpretable gene-module variational (GMV) latent variables.

## When to use

Use when a single-cell RNA-seq analysis must encode cells over a priori annotated latent variables, requiring GMT gene-set files (pathways, cell type markers, GRNs) to be converted into VEGA's decoder membership matrix.

## When NOT to use

- The goal is a data-driven latent representation without prior annotations — VEGA's decoder is hard-coded to user-provided GMT modules and cannot discover unannotated gene modules or correct prior knowledge when context requires it
- Analysis requires strong generative/predictive power — VEGA's sparse single-layer decoder sacrifices predictive performance for interpretability
- The input is already a gene-module activity matrix or pathway activity score table rather than raw/normalized single-cell expression data

## Inputs

- GMT gene-set file(s) specifying gene module variables (GMVs) and gene membership, e.g., Reactome pathway GMT from MSigDB
- Cell type marker GMT file (e.g., marker sets for cortical organoid cell types)
- Preprocessed single-cell dataset as a Scanpy AnnData object (normalized expression)

## Outputs

- Combined gene-module membership matrix (genes x [Reactome pathways + cell type marker sets]) defining VEGA's sparse linear decoder
- Per-cell GMV activity matrix after VEGA training (e.g., organoid_gmv_results.csv with log-Bayes-Factor significance values)

## How to apply

Collect the GMT files specifying each gene module variable (GMV) and its gene membership, e.g., a Reactome pathway GMT and a cell type marker GMT for the Field et al. week-2 cortical organoid dataset (GEO GSE106245). Parse each GMT to extract module names and their gene lists, then merge them into a single gene-module membership matrix where rows correspond to genes and columns to all Reactome pathways plus all cell type marker sets; this matrix is the mask that hard-codes the sparse linear decoder of VEGA. Ensure the gene dimension is aligned with the preprocessed Scanpy AnnData object (filtered cells/genes, normalized expression) that will be passed to the VEGA variational autoencoder, since unannotated genes are excluded from the decoder. After training, each GMV activity can be interrogated with VEGA's one-vs-rest Bayesian differential activity test, retaining significant modules at |log_e(BF)| > 3 (e.g., cell type marker GMVs segregating their annotated populations, or CELL_CYCLE_MITOTIC GMV correlating with MKI67, R2 ≈ 0.64).

## Related tools

- **VEGA** (Consumes the parsed GMT-derived membership matrix as the sparse mask wiring its linear decoder; implemented with PyTorch in the scvi-tools ecosystem) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset as an AnnData object whose gene space must be aligned with the GMT membership matrix) — https://scanpy.readthedocs.io/en/stable/
- **scvi-tools** (Ecosystem in which VEGA is implemented and trained on the organoid dataset) — https://scvi-tools.org/

## Examples

```
import vega; model = vega.VEGA(adata, gmts=['reactome_pathways.gmt','celltype_markers.gmt'])
```

## Evaluation signals

- The combined membership matrix columns equal the union of modules from both GMT files (Reactome pathways + cell type marker sets), and each column contains exactly the genes listed for that module in its GMT
- Matrix gene rows align 1:1 with the gene dimension of the preprocessed Scanpy AnnData object passed to VEGA
- After training, cell type marker-set GMV activity correctly segregates the corresponding annotated cell type at |log_e(BF)| > 3 in the one-vs-rest Bayesian differential activity test
- CELL_CYCLE_MITOTIC GMV activity correlates with MKI67 expression per cell with R2 ≈ 0.64, matching the reported result

## Limitations

- Prior biological knowledge from databases like MSigDB/Reactome can be incomplete or not context-specific — additional unannotated genes can play important roles in gene modules but are excluded by the hard-coded decoder mask
- The hard-coded connections of the sparse single-layer linear decoder leave no room for correcting prior knowledge about gene modules when the context requires it
- VEGA handles normalized expression profiles rather than raw counts, so GMT gene space must match preprocessed Scanpy data

## Evidence

- [readme] A GMT file specifying the gene module variables (GMVs) and gene membership, eg. from MSigDB: [evidence span withheld — anchor status: not_found]
- [methods] GMT files provide gene module membership to VEGA: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [intro] Gene module sources for decoder wiring: "guided by gene module membership as recorded in gene annotation databases (e.g., Gene Ontology, PANTHER, MolSigDB, or Reactome)"
- [abstract] Sparse decoder mirrors user-provided gene modules: "a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
