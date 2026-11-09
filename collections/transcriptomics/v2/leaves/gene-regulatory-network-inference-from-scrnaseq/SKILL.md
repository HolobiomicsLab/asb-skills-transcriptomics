---
name: gene-regulatory-network-inference-from-scrnaseq
description: Use when a pre-inferred co-expression module (GCN module, e.g. WGCNA/BioNERO-derived
  gene lists) must be converted into a directed, motif-validated GRN to identify its
  key transcription-factor regulators.
when_to_use_negative:
- Input is raw scRNA-seq data without upstream GCN/module inference — SCANet's GRN
  step expects preprocessed data and pre-identified co-expression modules, not raw
  counts
- The goal is co-expression network inference or module detection itself rather than
  converting existing modules into regulatory networks
- Grounding causal regulatory relationships is required — GRN conversion increases
  the risk of false negatives and GCNs rely on correlation, not causation, potentially
  leading to false positives
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCANet
  role: All-in-one Python package hosting the GRN inference function that converts
    GCN modules into directed regulatory networks
  repo: https://github.com/oubounyt/SCANet
- name: GRNBoost2
  role: Gradient-boosting (GENIE3-architecture) regression method that builds a tree-based
    model per target gene to propose candidate TF→TG edges
- name: RcisTarget
  role: Validates every candidate (TF, TG) edge by testing whether the TF regulatory
    binding motif is significantly enriched at the TG cis-regulatory region, discarding
    failed edges
- name: pySCENIC
  role: Workflow reference followed for the GRNBoost2-plus-RcisTarget edge validation
    scheme
- name: Scanpy
  role: Loads and handles the preprocessed single-cell expression matrix prior to
    GRN inference
- name: Python
  role: Language in which SCANet (available on PyPI, MIT license) is distributed and
    run
  repo: https://pypi.org/project/scanet/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:6bc9574392485f1047c6bdaee5222c96b89eb5e09e5f7257af3e144e90c7e88f
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# gene-regulatory-network-inference-from-scrnaseq

## Summary

Reconstructs directed transcriptional gene regulatory networks (GRNs) from single-cell co-expression modules by inferring TF→TG candidate edges with GRNBoost2 and validating each edge via RcisTarget motif enrichment, as implemented in the SCANet pipeline.

## When to use

Use when a pre-inferred co-expression module (GCN module, e.g. WGCNA/BioNERO-derived gene lists) must be converted into a directed, motif-validated GRN to identify its key transcription-factor regulators.

## When NOT to use

- Input is raw scRNA-seq data without upstream GCN/module inference — SCANet's GRN step expects preprocessed data and pre-identified co-expression modules, not raw counts
- The goal is co-expression network inference or module detection itself rather than converting existing modules into regulatory networks
- Grounding causal regulatory relationships is required — GRN conversion increases the risk of false negatives and GCNs rely on correlation, not causation, potentially leading to false positives

## Inputs

- Preprocessed scRNA-seq expression matrix (h5ad-style processed data, e.g. Cillo et al. 2021 GSE180578)
- Co-expression module gene lists retained from upstream WGCNA/BioNERO GCN inference (e.g. M11: 90 genes, M21: 137 genes)
- List of transcription factors present within the module gene sets

## Outputs

- Directed GRN graphs (TF -> TG orientation) with motif-validated edges per module
- Per-gene correlation counts and TF regulator sets (e.g. M11: 16/90 genes with correlation >= 0.9; M21: 11/137 genes with correlation = 1.0)
- Identified TF regulators per module (e.g. GATA2, MAFF, FOSB for M11; GSX2 for M21)

## How to apply

Load the preprocessed scRNA-seq expression matrix (e.g. Cillo et al. 2021 processed data, GSE180578) with Scanpy together with the retained module gene lists from the upstream WGCNA/BioNERO GCN inference. Scan the module genes for transcription factors and feed the TFs plus the expression matrix into the SCANet GRN inference function. Run GRNBoost2, the gradient-boosting (GENIE3-architecture) regression method, which builds a tree-based regression model per target gene to predict each TG expression profile from candidate TF profiles and retains the strongest contributors as candidate edges. Validate every candidate (TF, TG) edge with RcisTarget by testing whether the TF regulatory binding motif is significantly enriched at the cis-regulatory region of the TG, discarding edges that fail the enrichment test (pySCENIC workflow), then assemble the surviving edges into directed TF→TG regulatory graphs.

## Related tools

- **SCANet** (All-in-one Python package hosting the GRN inference function that converts GCN modules into directed regulatory networks) — https://github.com/oubounyt/SCANet
- **GRNBoost2** (Gradient-boosting (GENIE3-architecture) regression method that builds a tree-based model per target gene to propose candidate TF→TG edges)
- **RcisTarget** (Validates every candidate (TF, TG) edge by testing whether the TF regulatory binding motif is significantly enriched at the TG cis-regulatory region, discarding failed edges)
- **pySCENIC** (Workflow reference followed for the GRNBoost2-plus-RcisTarget edge validation scheme)
- **Scanpy** (Loads and handles the preprocessed single-cell expression matrix prior to GRN inference)
- **Python** (Language in which SCANet (available on PyPI, MIT license) is distributed and run) — https://pypi.org/project/scanet/

## Examples

```
pip install scanet; import scanet as sn  # then run the SCANet GRN inference function on module TFs + expression matrix
```

## Evaluation signals

- Computed directed GRNs retain motif-validated TF-TG edges (no edges that failed RcisTarget enrichment remain)
- Per-module correlation-count statistics match reported values: 16 of 90 M11 genes with correlation >= 0.9 and 11 of 137 M21 genes with correlation = 1.0 (Table 2, Fig. 3B, Fig. 3D)
- Identified TF regulators reproduce the reported sets: GATA2, MAFF, FOSB for M11 and GSX2 for M21
- Every retained edge carries a TF -> TG orientation in the assembled directed graph
- Module graph fits the scale-free topology, a characteristic of real-world biological networks

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- Gene–drug databases in Drugst.One update weekly, impacting reproducibility; running identical analyses and parameters on newer datasets may yield different results, and a version control system for Drugst.One is still in development

## Evidence

- [methods] RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded"
- [results] Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B): "Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B)"
- [discussion] GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis: "GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis"
- [intro] we developed a Python package for single-cell co-expression network analysis (SCANet), which is available on PyPI and distributed under the MIT license: "we developed a Python package for single-cell co-expression network analysis (SCANet), which is available on PyPI and distributed under the MIT license"
- [readme] Refer to this [Jupyter Notebook](https://github.com/oubounyt/SCAn/blob/main/docs/full-example.ipynb): "Refer to this [Jupyter Notebook](https://github.com/oubounyt/SCAn/blob/main/docs/full-example.ipynb)"
