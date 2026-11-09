---
name: coexpression-module-to-grn-conversion
description: Use when a trait- or cell-type-specific co-expression module (e.g. M11
  or M21 in classical monocytes) has already been inferred and the goal is to identify
  its transcriptional regulators (TFs) and build a directed TF→TG regulatory graph.
when_to_use_negative:
- No co-expression module gene list exists yet — run SCANet's modified WGCNA GCN inference
  and module detection (pseudo-bulk aggregation + module selection) first.
- The goal is co-expression hub detection or drug–gene interaction mining within a
  GCN rather than TF-to-target regulatory directionality.
- The dataset has no reliable cis-regulatory motif annotation for the candidate TFs,
  making RcisTarget motif validation infeasible.
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: SCANet
  role: All-in-one Python package whose GCN-to-GRN conversion function takes module
    TFs plus the expression matrix and reconstructs the directed GRN
  repo: https://github.com/oubounyt/SCANet
- name: GRNBoost2
  role: Gradient-boosting (GENIE3-architecture) regression method that builds a tree-based
    model per target gene and retains strongest TF contributors as candidate edges
- name: RcisTarget
  role: Validates every candidate (TF, TG) edge by testing TF binding-motif enrichment
    at the TG cis-regulatory region; discards edges failing the test
- name: pySCENIC
  role: Workflow whose motif-validation step (GRNBoost2 + RcisTarget) SCANet follows
    for GRN reconstruction
- name: Scanpy
  role: Loads and handles the preprocessed scRNA-seq expression matrix supplied to
    the GRN inference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:8034370364391289b9792def6b8ccfadf7a39ba66c1e4cbd6b97c504d0a08614
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# coexpression-module-to-grn-conversion

## Summary

Converts a co-expression gene module (GCN) inferred by SCANet's modified WGCNA/BioNERO pipeline into a directed transcriptional gene regulatory network (GRN) by identifying transcription factors among the module genes and validating TF–target edges with GRNBoost2 plus RcisTarget motif enrichment, following the pySCENIC workflow.

## When to use

Use when a trait- or cell-type-specific co-expression module (e.g. M11 or M21 in classical monocytes) has already been inferred and the goal is to identify its transcriptional regulators (TFs) and build a directed TF→TG regulatory graph.

## When NOT to use

- No co-expression module gene list exists yet — run SCANet's modified WGCNA GCN inference and module detection (pseudo-bulk aggregation + module selection) first.
- The goal is co-expression hub detection or drug–gene interaction mining within a GCN rather than TF-to-target regulatory directionality.
- The dataset has no reliable cis-regulatory motif annotation for the candidate TFs, making RcisTarget motif validation infeasible.

## Inputs

- Preprocessed single-cell expression matrix (processed scRNA-seq data from Cillo et al. 2021, GSE180578), loadable with Scanpy
- Retained co-expression module gene lists from upstream WGCNA/BioNERO GCN inference (e.g. M11: 90 genes, M21: 137 genes)
- List of transcription factors present within the module genes

## Outputs

- Directed regulatory graphs (TF -> TG orientation) for each module (e.g. M11 and M21 GRNs)
- Set of motif-validated TF–TG edges surviving RcisTarget enrichment testing
- Module network statistics: per-gene correlation counts and TF regulator identities (e.g. GATA2, MAFF, FOSB for M11; GSX2 for M21)

## How to apply

Load the preprocessed scRNA-seq expression matrix (Cillo et al. 2021, GSE180578) with Scanpy together with the retained module gene lists (e.g. M11 with 90 genes, M21 with 137 genes) from the upstream WGCNA/BioNERO GCN inference. Scan the module genes for transcription factors and feed the TFs plus the expression matrix into the SCANet GRN inference function. Run GRNBoost2, the gradient-boosting (GENIE3-architecture) regression method, which builds a tree-based regression model per target gene to predict each target-gene expression profile from candidate TF profiles and retains the strongest contributors as candidate edges. Validate every candidate (TF, TG) edge with RcisTarget by testing whether the TF regulatory binding motif is significantly enriched at the cis-regulatory region of the target gene, discarding edges that fail the enrichment test, then assemble the surviving edges into directed regulatory graphs (TF -> TG orientation). The rationale is that co-expression modules capture correlation rather than causation, so the motif-validation step filters co-expression-derived candidate edges to biologically plausible regulatory relationships.

## Related tools

- **SCANet** (All-in-one Python package whose GCN-to-GRN conversion function takes module TFs plus the expression matrix and reconstructs the directed GRN) — https://github.com/oubounyt/SCANet
- **GRNBoost2** (Gradient-boosting (GENIE3-architecture) regression method that builds a tree-based model per target gene and retains strongest TF contributors as candidate edges)
- **RcisTarget** (Validates every candidate (TF, TG) edge by testing TF binding-motif enrichment at the TG cis-regulatory region; discards edges failing the test)
- **pySCENIC** (Workflow whose motif-validation step (GRNBoost2 + RcisTarget) SCANet follows for GRN reconstruction)
- **Scanpy** (Loads and handles the preprocessed scRNA-seq expression matrix supplied to the GRN inference)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- Reproduced M11 statistics match the article: 16 of 90 M11 genes exhibit correlation >= 0.9, and M11 GRN regulators are exactly GATA2, MAFF, and FOSB (Table 2, Fig. 3B).
- Reproduced M21 statistics match the article: 11 of 137 M21 genes exhibit correlation = 1.0, and the M21 GRN regulator is GSX2 controlling two target genes, ERRFI1 (MIG6) and AKAP12 (Fig. 3D).
- Every retained edge in the directed GRN has passed the RcisTarget TF-motif enrichment test at the target gene's cis-regulatory region; failed edges must be discarded.
- The output graph is directed with TF -> TG orientation and contains only transcription factors from the input module gene list as source nodes.
- The module graph can additionally be checked for scale-free topology, a characteristic of real-world biological networks.

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis before GRN conversion.
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to go undetected.
- GRN regulators identified (e.g. GSX2 as M21 regulator) should be interpreted cautiously; the precise mechanistic effect of linked targets (e.g. erlotinib on ERRFI1) remains uncertain and requires experimental examination.

## Evidence

- [results] Three TFs (GATA2, MAFF, FOSB) in the M11 GRN regulate expressed TGs of highly pro-inflammatory reactions: "Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B)"
- [methods] RcisTarget edge validation by cis-regulatory motif enrichment: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG"
- [abstract] GCN-to-GRN conversion workflow step: "transcriptional gene regulatory network reconstruction"
- [readme] SCANet is a Python package installable from PyPI: "`SCANet`, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data"
- [results] M21 GRN regulator GSX2 with two target genes: "The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate (Waclaw et al. 2009) (Fig. 3D)"
