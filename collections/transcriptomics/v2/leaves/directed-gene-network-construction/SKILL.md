---
name: directed-gene-network-construction
description: Use when you have a bulk gene expression matrix (RNA-seq or microarray)
  with matched survival data and cell-type marker genes, and need to construct a directed
  gene-gene network whose edges encode favorable prognostic associations.
when_to_use_negative:
- You lack matched survival/time-to-event data, since the pair selection relies entirely
  on univariate Cox regression against survival
- You need ligand-receptor-based interaction inference rather than prognostic marker-pair
  associations
- Your cohort is too small to power the survival analysis (TimiGP requires a large
  gene expression dataset with matched survival information)
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TimiGP
  role: R package providing TimiGenePair, TimiCOX, and TimiGeneNetwork to binarize
    marker pairs, select prognostic pairs, and export the directed network
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Execution environment for the TimiGP pipeline
- name: devtools
  role: Used to install the TimiGP package from GitHub
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:30cc3ec153f78cc70d285d21283c4708c8f59a96c27bd95f6fdd0b51b6480870
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# directed-gene-network-construction

## Summary

Builds a directed prognostic gene-gene network from bulk transcriptomic and survival data using TimiGP: marker pairs are binarized into a Marker Pair Score matrix, filtered by univariate Cox regression with BH adjustment (QV<0.05, favorably oriented), and exported as Cytoscape-ready network files.

## When to use

Use when you have a bulk gene expression matrix (RNA-seq or microarray) with matched survival data and cell-type marker genes, and need to construct a directed gene-gene network whose edges encode favorable prognostic associations.

## When NOT to use

- You lack matched survival/time-to-event data, since the pair selection relies entirely on univariate Cox regression against survival
- You need ligand-receptor-based interaction inference rather than prognostic marker-pair associations
- Your cohort is too small to power the survival analysis (TimiGP requires a large gene expression dataset with matched survival information)

## Inputs

- Bulk transcriptomic profile (e.g., TCGA SKCM06 RNA-seq, SKCM06rna)
- Clinical survival information (e.g., SKCM06info) with event and time-to-event fields
- Cell-type marker gene annotations (e.g., Bindea2013 cancer cell-type markers, Immune_Marker_n1293)

## Outputs

- gene_network.sif (directed gene-gene network in Cytoscape SIF format)
- gene_node.txt (node attribute table for Cytoscape)
- gene_edge.txt (edge attribute table for Cytoscape)
- Selected set of favorably oriented, prognostic marker gene pairs (QV<0.05)

## How to apply

First filter low-quality clinical records with TimiCheckEvent and preprocess marker expression with TimiPrePropress (log=TRUE, GMNorm=TRUE for gene-wise median normalization). Generate the binary Marker Pair Score (MPS) matrix with TimiGenePair, which assigns 1 when gene A expression exceeds gene B and 0 otherwise. Run TimiCOX univariate Cox regression on every marker pair against survival, adjust p-values with Benjamini-Hochberg, and retain pairs with QV<0.05; orient each retained pair toward favorable prognosis by reversing pairs with HR>1. Export the resulting directed gene-gene network with TimiGeneNetwork (export=TRUE), producing gene_network.sif, gene_node.txt, and gene_edge.txt for Cytoscape import.

## Related tools

- **TimiGP** (R package providing TimiGenePair, TimiCOX, and TimiGeneNetwork to binarize marker pairs, select prognostic pairs, and export the directed network) — https://github.com/CSkylarL/TimiGP
- **R** (Execution environment for the TimiGP pipeline)
- **devtools** (Used to install the TimiGP package from GitHub) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); net <- TimiGeneNetwork(mps = mps, cox = cox_res, export = TRUE)
```

## Evaluation signals

- Every retained marker pair passes the Cox BH-adjusted significance threshold QV<0.05
- All selected pairs are oriented toward favorable prognosis: any pair with HR>1 has been reversed
- The MPS matrix is binary (values 1 when gene A expression exceeds gene B, 0 otherwise)
- TimiGeneNetwork with export=TRUE yields exactly three files: gene_network.sif, gene_node.txt, and gene_edge.txt, all loadable in Cytoscape

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- The power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of the inferred cell-cell interaction

## Evidence

- [readme] TimiGP is an R package for inferring interactions from gene pairs: [evidence span withheld — anchor status: external_doc]
