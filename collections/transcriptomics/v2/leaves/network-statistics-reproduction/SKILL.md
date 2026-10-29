---
name: network-statistics-reproduction
description: Use when a task asks you to reproduce, verify, or cross-check reported
  network statistics (e.g. '16 of 90 genes with correlation >= 0.9', 'GRN regulators
  GATA2, MAFF, FOSB', Table 2 / Fig. 3B / Fig. 3D values) from a co-expression or
  regulatory network analysis.
when_to_use_negative:
- The goal is de novo discovery of regulators or modules rather than checking previously
  reported statistics — use the upstream GCN/GRN inference skill instead
- Input data is not the classical monocyte population or the module gene lists were
  not retained from a prior GCN run, making the reported correlation counts (16/90,
  11/137) unreproducible
- Only correlation-based GCN statistics are needed and no GRN conversion is in scope;
  GRNBoost2/RcisTarget overhead is unnecessary
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCANet
  role: Python package hosting the GCN-to-GRN conversion and network statistics workflow
    used to regenerate the reported M11/M21 values
  repo: https://github.com/oubounyt/SCANet
- name: GRNBoost2
  role: Gradient-boosting regression that builds a tree-based model per target gene
    and retains strongest TF contributors as candidate edges
- name: RcisTarget
  role: Validates each (TF, TG) edge by testing whether the TF regulatory binding
    motif is significantly enriched at the TG cis-regulatory region; failing edges
    are discarded
- name: Scanpy
  role: Loads and handles the preprocessed scRNA-seq expression matrix (GSE180578)
    feeding the statistics computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:d323b778f016536f6273df71162add26d068f0a9e59df7e1cf6626abde1b9014
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# network-statistics-reproduction

## Summary

A validation skill for reproducing published gene co-expression network (GCN) and gene regulatory network (GRN) statistics — per-module correlation-count thresholds and transcription factor regulator identities — so that a re-run pipeline can be checked against reported values such as SCANet's M11/M21 monocyte modules.

## When to use

Use when a task asks you to reproduce, verify, or cross-check reported network statistics (e.g. '16 of 90 genes with correlation >= 0.9', 'GRN regulators GATA2, MAFF, FOSB', Table 2 / Fig. 3B / Fig. 3D values) from a co-expression or regulatory network analysis.

## When NOT to use

- The goal is de novo discovery of regulators or modules rather than checking previously reported statistics — use the upstream GCN/GRN inference skill instead
- Input data is not the classical monocyte population or the module gene lists were not retained from a prior GCN run, making the reported correlation counts (16/90, 11/137) unreproducible
- Only correlation-based GCN statistics are needed and no GRN conversion is in scope; GRNBoost2/RcisTarget overhead is unnecessary

## Inputs

- Preprocessed single-cell expression matrix (Scanpy object) from Cillo et al. 2021 PBMC data, GSE180578
- Retained module gene lists from upstream WGCNA/BioNERO GCN inference (e.g. M11 with 90 genes, M21 with 137 genes)
- Reported reference statistics (correlation counts, TF regulator identities) from the source article (Table 2, Fig. 3B, Fig. 3D)

## Outputs

- Computed per-gene correlation counts per module (e.g. counts at correlation >= 0.9 or = 1.0)
- Identified TF regulator sets per module GRN (e.g. GATA2, MAFF, FOSB for M11; GSX2 for M21)
- Directed, motif-validated TF->TG regulatory graphs for M11 and M21
- Pass/fail comparison against the article-reported values

## How to apply

Load the preprocessed scRNA-seq expression matrix (Cillo et al. 2021, GSE180578) with Scanpy together with the retained module gene lists (M11: 90 genes, M21: 137 genes) from the upstream WGCNA/BioNERO GCN inference. Scan module genes for transcription factors and run GRNBoost2 (GENIE3-architecture gradient boosting) per target gene, then validate every candidate (TF, TG) edge with RcisTarget, discarding edges whose TF binding motif is not significantly enriched at the TG cis-regulatory region (pySCENIC workflow). Assemble surviving edges into directed TF->TG graphs for each module and compute per-gene correlation counts and TF regulator sets. Accept the reproduction only when computed values match the published statistics: 16 of 90 M11 genes with correlation >= 0.9, 11 of 137 M21 genes with correlation = 1.0, M11 regulators GATA2/MAFF/FOSB, and M21 regulator GSX2 (Table 2, Fig. 3B, Fig. 3D).

## Related tools

- **SCANet** (Python package hosting the GCN-to-GRN conversion and network statistics workflow used to regenerate the reported M11/M21 values) — https://github.com/oubounyt/SCANet
- **GRNBoost2** (Gradient-boosting regression that builds a tree-based model per target gene and retains strongest TF contributors as candidate edges)
- **RcisTarget** (Validates each (TF, TG) edge by testing whether the TF regulatory binding motif is significantly enriched at the TG cis-regulatory region; failing edges are discarded)
- **Scanpy** (Loads and handles the preprocessed scRNA-seq expression matrix (GSE180578) feeding the statistics computation)

## Examples

```
pip install scanet
```

## Evaluation signals

- Computed M11 correlation count equals 16 of 90 genes with correlation >= 0.9
- Computed M21 correlation count equals 11 of 137 genes with correlation = 1.0
- M11 GRN regulator set matches exactly {GATA2, MAFF, FOSB}
- M21 GRN regulator matches exactly {GSX2}, controlling TGs ERRFI1 (MIG6) and AKAP12
- Every retained GRN edge passed the RcisTarget motif-enrichment test and graphs are oriented TF -> TG

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing subtle true relationships between genes to be undetected
- Gene-drug databases underlying SCANet (Drugst.One) update weekly, so running identical analyses and parameters on newer datasets may yield different results, impacting reproducibility

## Evidence

- [results] Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B): "Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B)"
- [results] In the GCN of M11 (Fig. 2B, 16 out of 90 genes exhibit correlation >= 0.9): "In the GCN of M11 (Fig. 2B, 16 out of 90 genes exhibit correlation ≥ 0.9)"
- [results] The M21 co-expressed gene module (Fig. 2B, 11 genes out of 137 genes exhibit correlation = 1.0): "The M21 co-expressed gene module (Fig. 2B, 11 genes out of 137 genes exhibit correlation = 1.0)"
- [results] The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate (Waclaw et al. 2009) (Fig. 3D): "The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate (Waclaw et al. 2009) (Fig. 3D)"
- [methods] RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched"
- [readme] SCANet, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data: "SCANet, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data"
