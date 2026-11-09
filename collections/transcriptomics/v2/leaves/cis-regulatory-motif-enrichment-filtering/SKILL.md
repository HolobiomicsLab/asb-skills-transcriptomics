---
name: cis-regulatory-motif-enrichment-filtering
description: 'Use when converting co-expression modules into directed gene regulatory
  networks: after GRNBoost2 proposes candidate TF–TG edges from single-cell expression
  data, to retain only edges with cis-regulatory motif evidence.'
when_to_use_negative:
- The goal is co-expression module detection or hub-gene identification only, with
  no need for directed TF–TG relationships
- The candidate edges are not TF–TG pairs (e.g. gene–drug interaction queries, which
  instead use correlation thresholds such as >= 0.8 in SCANet)
- Input is already a motif-validated directed GRN; re-running the filter would only
  remove edges without adding information
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RcisTarget
  role: Validates each proposed (TF, TG) edge by cis-regulatory TF-binding motif enrichment
    and discards edges without significant enrichment
- name: GRNBoost2
  role: Upstream proposer of candidate TF–TG edges from the TFs of the selected module
    and the processed expression data, prior to motif filtering
- name: SCANet
  role: Python package that orchestrates GCN inference, TF detection, GRN reconstruction
    and the RcisTarget motif-enrichment filter
  repo: https://github.com/oubounyt/SCANet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:3dfe7e807e4fc5f31fcdf59ce2f149f83b5bbd2de36db019ff8bc3aa8c86f27a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cis-regulatory motif enrichment filtering

## Summary

A filter step in SCANet's GCN-to-GRN conversion: RcisTarget validates each candidate TF–target-gene edge proposed by GRNBoost2 by checking whether the TF's regulatory binding motif is significantly enriched at the target's cis-regulatory region, discarding unsupported edges to yield a directed GRN.

## When to use

Use when converting co-expression modules into directed gene regulatory networks: after GRNBoost2 proposes candidate TF–TG edges from single-cell expression data, to retain only edges with cis-regulatory motif evidence.

## When NOT to use

- The goal is co-expression module detection or hub-gene identification only, with no need for directed TF–TG relationships
- The candidate edges are not TF–TG pairs (e.g. gene–drug interaction queries, which instead use correlation thresholds such as >= 0.8 in SCANet)
- Input is already a motif-validated directed GRN; re-running the filter would only remove edges without adding information

## Inputs

- GRNBoost2-proposed candidate TF–TG edge list for a selected co-expression module (e.g. M4)
- List of transcription factor genes detected in the selected module
- Processed single-cell expression matrix (H5ad/HDF5/loom) loaded via Scanpy/SCANet
- cis-regulatory sequences / motif ranking database used by RcisTarget

## Outputs

- Directed gene regulatory network (GRN) containing only edges whose TF motif is significantly enriched at the TG cis-regulatory region
- Module-level regulator set, e.g. the four lipid-metabolism TFs RXRA, NR1D1, SREBF1, RORC controlling ISC-associated module M4 in HFHSD mice

## How to apply

First run GRNBoost2 on the transcription factors detected in a selected co-expression module (e.g. module M4 from the WGCNA/BioNERO co-expression analysis of Aliluev et al. 2021 enteroendocrine scRNA-seq) together with the processed expression matrix to propose candidate TF–TG edges. Then, within SCANet, apply RcisTarget to every proposed edge: for each (TF, TG) pair, test whether the regulatory binding motif of the TF is significantly enriched in the cis-regulatory region of the TG; edges failing this significance test are discarded. The surviving edges form the directed GRN, in which the module's TFs and their validated targets can then be inspected (e.g. M4 is controlled by the lipid-metabolism TFs RXRA, NR1D1, SREBF1/SREBP-1 and RORC). The rationale is that motif enrichment adds cis-regulatory evidence on top of expression-based inference, pruning indirect or spurious co-expression links before downstream drug-repurposing queries (e.g. GCN–drug interactions at correlation >= 0.8).

## Related tools

- **RcisTarget** (Validates each proposed (TF, TG) edge by cis-regulatory TF-binding motif enrichment and discards edges without significant enrichment)
- **GRNBoost2** (Upstream proposer of candidate TF–TG edges from the TFs of the selected module and the processed expression data, prior to motif filtering)
- **SCANet** (Python package that orchestrates GCN inference, TF detection, GRN reconstruction and the RcisTarget motif-enrichment filter) — https://github.com/oubounyt/SCANet

## Evaluation signals

- Every edge retained in the final GRN has a TF regulatory binding motif significantly enriched at the TG's cis-regulatory region; any edge failing this test is absent
- For the Aliluev et al. 2021 ISC use case, the run is judged successful when M4 remains the module most strongly correlated with ISCs in HFHSD mice and its filtered GRN is controlled by RXRA, NR1D1, SREBF1, and RORC
- The filtered GRN is strictly directed (TF -> TG) and restricted to TFs identified in the selected co-expression module
- The validated GRN can be carried forward to GCN/GRN–drug interaction queries (e.g. M4 GCN–drug interactions at correlation >= 0.8) without re-inferring edges

## Limitations

- Converting GCNs into GRNs may increase the risk of false negatives, causing subtle true relationships between genes to go undetected
- GCNs rely on correlation, not causation, so motif filtering reduces but does not eliminate false positives in co-expression analysis
- Downstream drug-gene databases (e.g. Drugst.One/NeDRex) update frequently, so results of follow-on analyses may differ across database versions

## Evidence

- [methods] RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region of the TG"
- [results] The GRN of the gene module with the strongest correlation to ISC in HFHSD (M4) revealed a genetic program controlled by four TFs (RXRA, NR1D1, SREBF1/SREBP-1, and RORC), all known regulators of lipid metabolism: "The GRN of the gene module with the strongest correlation to ISC in HFHSD (M4) revealed a genetic program controlled by four TFs (RXRA, NR1D1, SREBF1/SREBP-1, and RORC)"
- [discussion] Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected: [evidence span withheld — anchor status: not_found]
- [intro] SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection: "SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection"
