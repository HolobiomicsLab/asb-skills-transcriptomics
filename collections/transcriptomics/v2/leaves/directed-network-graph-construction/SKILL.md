---
name: directed-network-graph-construction
description: Use when an undirected co-expression module (e.g. from WGCNA/BioNERO)
  must be converted into a directed TF→target regulatory graph whose edges need motif-level
  validation.
when_to_use_negative:
- The goal is undirected co-expression structure or hub detection only — GCN inference
  (modified WGCNA) suffices without GRNBoost2/RcisTarget conversion
- The expression matrix has not been preprocessed/pseudo-bulked by the upstream SCANet
  GCN workflow, so TF profiles are too sparse for reliable regression
- The analysis requires proven causal regulatory edges — GRN edges are inference-based
  and may introduce false negatives for subtle true relationships
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: SCANet
  role: All-in-one Python package whose GRN inference function converts module GCNs
    to directed GRNs
  repo: https://github.com/oubounyt/SCANet
- name: GRNBoost2
  role: Gradient-boosting (GENIE3-architecture) regression that predicts each TG expression
    from candidate TF profiles and retains strongest contributors as candidate edges
- name: RcisTarget
  role: Validates each (TF, TG) edge by testing whether the TF regulatory binding
    motif is significantly enriched at the TG cis-regulatory region; discards failing
    edges
- name: pySCENIC
  role: Workflow reference for the GRNBoost2 + RcisTarget edge-validation scheme
- name: Scanpy
  role: Loads and provides the preprocessed scRNA-seq expression matrix used as GRN
    input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:6a72e42349d6f457f10571ca75a46a5e63e1b9b335f8a65ddda1964d259f2aea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - GRNBoost2
  - Python
  - RcisTarget
  - SCANet
  - Scanpy
  - pySCENIC
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# directed-network-graph-construction

## Summary

Converts an undirected gene co-expression network (GCN) module into a directed transcriptional gene regulatory network (GRN) by inferring TF→TG edges with GRNBoost2 and validating each edge with RcisTarget motif enrichment, as done in SCANet for the M11 and M21 monocyte modules.

## When to use

Use when an undirected co-expression module (e.g. from WGCNA/BioNERO) must be converted into a directed TF→target regulatory graph whose edges need motif-level validation.

## When NOT to use

- The goal is undirected co-expression structure or hub detection only — GCN inference (modified WGCNA) suffices without GRNBoost2/RcisTarget conversion
- The expression matrix has not been preprocessed/pseudo-bulked by the upstream SCANet GCN workflow, so TF profiles are too sparse for reliable regression
- The analysis requires proven causal regulatory edges — GRN edges are inference-based and may introduce false negatives for subtle true relationships

## Inputs

- Preprocessed single-cell expression matrix (e.g. Scanpy-loaded GSE180578 processed scRNA-seq data from Cillo et al. 2021)
- Retained co-expression module gene lists from upstream WGCNA/BioNERO GCN inference (e.g. M11 with 90 genes, M21 with 137 genes)
- List of transcription factors found within the module genes

## Outputs

- Directed GRN graph with motif-validated TF→TG edges per module (e.g. M11 regulators GATA2, MAFF, FOSB; M21 regulator GSX2)
- Module network statistics: per-gene correlation counts and TF regulator sets (e.g. 16/90 M11 genes with correlation ≥ 0.9; 11/137 M21 genes with correlation = 1.0)

## How to apply

Load the preprocessed scRNA-seq expression matrix (e.g. the Cillo et al. 2021 / GSE180578 matrix with Scanpy) together with the retained module gene lists from upstream GCN inference. Scan the module genes for transcription factors and feed the TFs plus the expression matrix into the SCANet GRN inference function. Run GRNBoost2, the gradient-boosting (GENIE3-architecture) regression method that builds a tree-based regression model per target gene to predict each target gene's expression from candidate TF profiles and retains the strongest contributors as candidate edges. Validate every candidate (TF, TG) edge with RcisTarget by testing whether the TF's regulatory binding motif is significantly enriched at the cis-regulatory region of the TG, discarding edges that fail the enrichment test (pySCENIC workflow). Assemble the surviving edges into a directed regulatory graph with TF→TG orientation, then compute per-gene correlation counts and TF regulator sets (e.g. M11: 16/90 genes with correlation ≥ 0.9, regulators GATA2, MAFF, FOSB; M21: 11/137 genes with correlation = 1.0, regulator GSX2).

## Related tools

- **SCANet** (All-in-one Python package whose GRN inference function converts module GCNs to directed GRNs) — https://github.com/oubounyt/SCANet
- **GRNBoost2** (Gradient-boosting (GENIE3-architecture) regression that predicts each TG expression from candidate TF profiles and retains strongest contributors as candidate edges)
- **RcisTarget** (Validates each (TF, TG) edge by testing whether the TF regulatory binding motif is significantly enriched at the TG cis-regulatory region; discards failing edges)
- **pySCENIC** (Workflow reference for the GRNBoost2 + RcisTarget edge-validation scheme)
- **Scanpy** (Loads and provides the preprocessed scRNA-seq expression matrix used as GRN input)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- Directed GRN retains only motif-validated TF-TG edges (edges failing RcisTarget enrichment are discarded)
- M11 statistics match the article: 16 of 90 genes exhibit correlation ≥ 0.9 and regulators are GATA2, MAFF, FOSB (Table 2, Fig. 3B)
- M21 statistics match the article: 11 of 137 genes exhibit correlation = 1.0 and the regulator is GSX2 controlling ERRFI1 (MIG6) and AKAP12 (Fig. 3D)
- Every edge in the output graph has a TF→TG orientation (directed, not undirected)

## Limitations

- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis that propagate into the GRN
- GRN inference is bounded by the TF list detectable in the module genes and the expressed target genes available in the matrix

## Evidence

- [methods] RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded"
- [results] Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B): "Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B)"
- [results] The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate (Waclaw et al. 2009) (Fig. 3D): "The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate (Waclaw et al. 2009) (Fig. 3D)"
- [results] In the GCN of M11 (Fig. 2B, 16 out of 90 genes exhibit correlation ≥ 0.9), over half of the genes and hubs code for heat shock proteins: "In the GCN of M11 (Fig. 2B, 16 out of 90 genes exhibit correlation ≥ 0.9), over half of the genes and hubs code for heat shock proteins"
- [readme] SCANet, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data: "SCANet, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data and a complete analysis of the identified modules"
