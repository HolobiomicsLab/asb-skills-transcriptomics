---
name: grn-inference-with-gradient-boosting
description: Use when a co-expression module (e.g. a WGCNA module correlated with
  a trait or cell type) must be converted into a directed TF–target regulatory network
  to identify the transcription factors controlling it.
when_to_use_negative:
- When the goal is a pure co-expression (undirected) GCN without TF attribution —
  SCANet's WGCNA-based module inference is the appropriate upstream step, not GRNBoost2
- When no transcription factor genes are present in the selected module — GRNBoost2
  requires TFs to propose TF–TG edges
- When downstream drug-repurposing needs only gene–drug interactions on the GCN rather
  than regulatory directionality
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: SCANet
  role: All-in-one Python package hosting the GCN-to-GRN reconstruction workflow,
    including TF identification and GRN edge filtering
  repo: https://github.com/oubounyt/SCANet
- name: GRNBoost2
  role: Gradient-boosting algorithm that proposes candidate TF–TG regulatory edges
    from the TFs and the processed expression data
- name: RcisTarget
  role: Validates each proposed (TF, TG) edge by checking whether the TF's regulatory
    binding motif is significantly enriched at the TG's cis-regulatory region; edges
    failing enrichment are discarded
- name: BioNERO
  role: Framework via which SCANet runs WGCNA to infer the co-expression modules from
    which TFs and targets are drawn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:4f400c18e777ee6db5b0d9ca8d844270852480b23b0861e4113dc1780e18f8ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - BioNERO
  - GRNBoost2
  - Python
  - RcisTarget
  - SCANet
  - Scanpy
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# GRN inference with gradient boosting

## Summary

Infer a directed gene regulatory network (GRN) for a selected co-expression module by first proposing candidate TF–target edges with gradient-boosting (GRNBoost2) and then retaining only edges whose TF-binding motif is significantly enriched at the target's cis-regulatory region (RcisTarget), as applied in SCANet to the M4 lipid-metabolism GRN of intestinal stem cells in HFHSD obese mice.

## When to use

Use when a co-expression module (e.g. a WGCNA module correlated with a trait or cell type) must be converted into a directed TF–target regulatory network to identify the transcription factors controlling it.

## When NOT to use

- When the goal is a pure co-expression (undirected) GCN without TF attribution — SCANet's WGCNA-based module inference is the appropriate upstream step, not GRNBoost2
- When no transcription factor genes are present in the selected module — GRNBoost2 requires TFs to propose TF–TG edges
- When downstream drug-repurposing needs only gene–drug interactions on the GCN rather than regulatory directionality

## Inputs

- Processed single-cell expression matrix (H5ad/HDF5/loom) with cell-level metadata (cell type, diet condition, batch)
- A selected co-expression gene module (e.g. M4) and its gene list
- List of transcription factor genes found within the module

## Outputs

- Directed TF–TG regulatory network (GRN) with motif-validated edges
- Set of controlling transcription factors for the module (e.g. RXRA, NR1D1, SREBF1, RORC)

## How to apply

First scan the selected co-expression module (e.g. the module most strongly correlated with ISCs in HFHSD mice, M4) for transcription factor genes. Run GRNBoost2 on those TFs together with the processed scRNA-seq expression data to propose candidate directed TF–TG edges. Then filter every proposed edge with RcisTarget cis-regulatory TF-binding motif enrichment: an edge (TF, TG) is kept only if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded. The rationale is that gradient boosting alone yields candidate regulators, while motif enrichment supplies cis-regulatory evidence that converts the co-expression-derived GCN into a directed GRN with fewer spurious regulators.

## Related tools

- **SCANet** (All-in-one Python package hosting the GCN-to-GRN reconstruction workflow, including TF identification and GRN edge filtering) — https://github.com/oubounyt/SCANet
- **GRNBoost2** (Gradient-boosting algorithm that proposes candidate TF–TG regulatory edges from the TFs and the processed expression data)
- **RcisTarget** (Validates each proposed (TF, TG) edge by checking whether the TF's regulatory binding motif is significantly enriched at the TG's cis-regulatory region; edges failing enrichment are discarded)
- **BioNERO** (Framework via which SCANet runs WGCNA to infer the co-expression modules from which TFs and targets are drawn)

## Examples

```
import scanet as sn; grn = sn.grnboost2(module_genes=M4_genes, tfs=M4_tfs, expr=adata); grn = sn.rcis_target_filter(grn)
```

## Evaluation signals

- The number of co-expression modules reproduced matches the reference (25 modules in the Aliluev et al. 2021 ISC dataset, with M4 most strongly correlated with ISCs in HFHSD mice)
- The filtered GRN is controlled by the expected lipid-metabolism TFs: RXRA, NR1D1, SREBF1/SREBP-1, and RORC
- Every retained GRN edge passes the RcisTarget check — the TF's binding motif is significantly enriched at the TG's cis-regulatory region
- Proposed GRNBoost2 edges that fail cis-regulatory motif enrichment are correctly discarded
- The module graph fits the scale-free topology, a characteristic of real-world biological networks

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- RcisTarget filtering can only validate edges with detectable motif enrichment; TFs lacking annotated motifs may be lost

## Evidence

- [methods] RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG"
- [results] The GRN of the gene module with the strongest correlation to ISC in HFHSD (M4) revealed a genetic program controlled by four TFs (RXRA, NR1D1, SREBF1/SREBP-1, and RORC), all known regulators of lipid metabolism.: "The GRN of the gene module with the strongest correlation to ISC in HFHSD (M4) revealed a genetic program controlled by four TFs (RXRA, NR1D1, SREBF1/SREBP-1, and RORC)"
- [discussion] GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis: "GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis"
- [methods] They can also check whether the module graph fits the scale-free topology, a characteristic of real-world biological networks: "whether the module graph fits the scale-free topology, a characteristic of real-world biological networks"
- [intro] SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection: "SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection"
