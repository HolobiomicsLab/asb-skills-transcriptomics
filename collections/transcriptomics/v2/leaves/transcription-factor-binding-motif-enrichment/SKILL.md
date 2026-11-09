---
name: transcription-factor-binding-motif-enrichment
description: Use when converting a co-expression network (GCN) to a directed GRN and
  candidate TF→TG edges (e.g. from GRNBoost2/GENIE3-style regression) must be filtered
  by cis-regulatory motif enrichment to keep only motif-supported regulators.
when_to_use_negative:
- The goal is co-expression structure or hub detection rather than directionality
  — the plain GCN (correlation-based, e.g. correlation ≥ 0.9 for M11) suffices and
  motif validation is unnecessary
- The input module contains no transcription factors to serve as candidate regulators
- Edges must be interpreted causally without motif support — RcisTarget validation
  removes edges, so false negatives may cause subtle true relationships between genes
  to be undetected
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCANet
  role: Package whose GCN-to-GRN conversion step runs motif enrichment validation
    of TF–TG edges
  repo: https://github.com/oubounyt/SCANet
- name: GRNBoost2
  role: Generates candidate TF–TG edges via gradient-boosting (GENIE3-architecture)
    regression before motif filtering
- name: RcisTarget
  role: Tests whether the TF regulatory binding motif is significantly enriched at
    the TG cis-regulatory region; discards edges that fail
- name: pySCENIC
  role: Workflow whose motif-validation step (RcisTarget) is followed for GRN pruning
- name: Scanpy
  role: Loads and preprocesses the scRNA-seq expression matrix prior to GRN inference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:d279fe2f641d0c15cede7e1e75e40d1d2cd596ce6b262cbb65c2f2ec4d49b5e1
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# transcription-factor-binding-motif-enrichment

## Summary

Validates candidate transcription factor–target gene (TF, TG) edges in single-cell gene regulatory networks by testing whether the TF's regulatory binding motif is significantly enriched at the TG's cis-regulatory region, discarding edges that fail — as done in SCANet's GCN-to-GRN conversion following the pySCENIC/RcisTarget workflow.

## When to use

Use when converting a co-expression network (GCN) to a directed GRN and candidate TF→TG edges (e.g. from GRNBoost2/GENIE3-style regression) must be filtered by cis-regulatory motif enrichment to keep only motif-supported regulators.

## When NOT to use

- The goal is co-expression structure or hub detection rather than directionality — the plain GCN (correlation-based, e.g. correlation ≥ 0.9 for M11) suffices and motif validation is unnecessary
- The input module contains no transcription factors to serve as candidate regulators
- Edges must be interpreted causally without motif support — RcisTarget validation removes edges, so false negatives may cause subtle true relationships between genes to be undetected

## Inputs

- Preprocessed scRNA-seq expression matrix (e.g. Cillo et al. 2021, GSE180578, loaded with Scanpy)
- Module gene lists from upstream WGCNA/BioNERO GCN inference (e.g. M11: 90 genes; M21: 137 genes)
- List of transcription factors found among the module genes
- Candidate TF→TG edges from GRNBoost2 regression
- TF regulatory binding motif database used by RcisTarget

## Outputs

- Directed GRN (TF -> TG orientation) containing only motif-validated edges for each module
- Per-module regulator sets (e.g. M11: GATA2, MAFF, FOSB; M21: GSX2)
- Per-gene correlation counts and network statistics for module validation

## How to apply

First infer candidate edges with GRNBoost2, which builds a tree-based regression model per target gene to predict each TG expression profile from candidate TF profiles and retains the strongest contributors. Then, for every candidate (TF, TG) edge, run RcisTarget to test whether the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG; discard edges that fail the enrichment test, following the pySCENIC workflow. Assemble the surviving validated edges into directed regulatory graphs (TF -> TG orientation) for the modules of interest. In the SCANet paper this converted the M11 co-expression module (classical monocytes of deceased COVID-19 patients, GSE180578 data) into a GRN whose regulators were GATA2, MAFF, and FOSB, and M21 into a GRN regulated by GSX2 (controlling ERRFI1/MIG6 and AKAP12). Transcription factors can be identified by scanning the module gene lists before GRN inference.

## Related tools

- **SCANet** (Package whose GCN-to-GRN conversion step runs motif enrichment validation of TF–TG edges) — https://github.com/oubounyt/SCANet
- **GRNBoost2** (Generates candidate TF–TG edges via gradient-boosting (GENIE3-architecture) regression before motif filtering)
- **RcisTarget** (Tests whether the TF regulatory binding motif is significantly enriched at the TG cis-regulatory region; discards edges that fail)
- **pySCENIC** (Workflow whose motif-validation step (RcisTarget) is followed for GRN pruning)
- **Scanpy** (Loads and preprocesses the scRNA-seq expression matrix prior to GRN inference)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- All retained (TF, TG) edges pass the RcisTarget cis-regulatory motif enrichment test; edges failing the test are discarded
- Reproduced regulator identities match the reported values: M11 regulators are GATA2, MAFF, and FOSB (Table 2, Fig. 3B); M21 regulator is GSX2 (Fig. 3D)
- Reproduced network statistics match the article: 16 of 90 M11 genes with correlation ≥ 0.9 and 11 of 137 M21 genes with correlation = 1.0
- Resulting GRNs are directed graphs with TF -> TG edge orientation
- The GRN's target genes correspond to highly pro-inflammatory reactions for M11, and for M21 GSX2 controls two TGs: ERRFI1 (MIG6) and AKAP12

## Limitations

- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis upstream of motif validation

## Evidence

- [methods] RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG, otherwise it is discarded"
- [results] Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions: "Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B)"
- [results] The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate: "The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate (Waclaw et al. 2009) (Fig. 3D)"
- [readme] pip install scanet: "To install `SCANet`.
console
pip install scanet
"
