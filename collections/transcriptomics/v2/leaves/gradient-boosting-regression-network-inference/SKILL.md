---
name: gradient-boosting-regression-network-inference
description: Use when a gene co-expression module (e.g. WGCNA/BioNERO module gene
  list) must be converted into a directed TF-to-target regulatory graph, with candidate
  edges pruned by cis-regulatory motif enrichment.
when_to_use_negative:
- Input is bulk RNA-seq or aggregated expression without single-cell resolution —
  the pipeline is built for scRNA-seq co-expression analysis
- The goal is an undirected co-expression network (GCN) with hub detection rather
  than directional TF regulation — SCANet's WGCNA-based module inference covers that
  instead
- No TF annotation or cis-regulatory motif database is available, so RcisTarget edge
  validation cannot be performed and candidate edges cannot be filtered
tools:
- name: SCANet
  role: Python package whose GCN-to-GRN conversion wraps the GRN inference and exposes
    the GRNBoost2/RcisTarget workflow on module gene lists
  repo: https://github.com/oubounyt/SCANet
- name: GRNBoost2
  role: 'Gradient-boosting (GENIE3-architecture) regression method: builds a tree-based
    regression model per target gene to predict target expression from candidate TF
    profiles and retains the strongest contributors as candidate edges'
- name: RcisTarget
  role: Validates each candidate (TF, TG) edge by testing whether the TF regulatory
    binding motif is significantly enriched at the cis-regulatory region of the TG;
    failing edges are discarded
- name: pySCENIC
  role: Workflow whose GRN inference + motif-validation scheme (GRNBoost2 followed
    by RcisTarget) is followed for edge pruning
- name: Scanpy
  role: Loads and preprocesses the single-cell expression matrix before TF scanning
    and GRN inference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:08e39725d7f01220fa3d17320bfe2b91146e79a622575725dcca1ba21074c8a5
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gradient-boosting-regression-network-inference

## Summary

Infers directed transcription factor → target gene regulatory networks from scRNA-seq expression matrices by fitting gradient-boosted tree regression models per target gene (GRNBoost2, GENIE3 architecture) and retaining only motif-validated edges via RcisTarget, as done in SCANet's GCN-to-GRN conversion.

## When to use

Use when a gene co-expression module (e.g. WGCNA/BioNERO module gene list) must be converted into a directed TF-to-target regulatory graph, with candidate edges pruned by cis-regulatory motif enrichment.

## When NOT to use

- Input is bulk RNA-seq or aggregated expression without single-cell resolution — the pipeline is built for scRNA-seq co-expression analysis
- The goal is an undirected co-expression network (GCN) with hub detection rather than directional TF regulation — SCANet's WGCNA-based module inference covers that instead
- No TF annotation or cis-regulatory motif database is available, so RcisTarget edge validation cannot be performed and candidate edges cannot be filtered

## Inputs

- Preprocessed single-cell RNA-seq expression matrix (e.g. processed Cillo et al. 2021 data, GSE180578, .h5ad)
- Gene list(s) of co-expression modules (e.g. M11: 90 genes, M21: 137 genes) from upstream WGCNA/BioNERO GCN inference
- List of transcription factors identified within the module gene lists

## Outputs

- Directed TF -> target gene regulatory graph (GRN) per module with motif-validated edges
- Set of key TF regulators per GRN (e.g. GATA2, MAFF, FOSB for M11; GSX2 for M21)
- Module network statistics: per-gene correlation counts (16 of 90 M11 genes with correlation >= 0.9; 11 of 137 M21 genes with correlation = 1.0) and TF regulator identities (Table 2, Fig. 3B, Fig. 3D)

## How to apply

Scan the module gene list (e.g. the 90-gene M11 or 137-gene M21 classical monocyte modules) for transcription factors and supply these TFs together with the preprocessed scRNA-seq expression matrix (h5ad, loaded with Scanpy; here Cillo et al. 2021 GSE180578) to the SCANet GRN inference function. Run GRNBoost2, which builds a gradient-boosted tree regression model per target gene to predict each target gene's expression profile from candidate TF profiles and retains the strongest contributors as candidate edges. Validate every candidate (TF, TG) edge with RcisTarget by testing whether the TF binding motif is significantly enriched at the cis-regulatory region of the TG, and discard edges failing the enrichment test (pySCENIC workflow). Assemble surviving edges into directed TF -> TG graphs and report per-gene correlation counts and TF regulator sets, e.g. GATA2, MAFF, FOSB for M11 and GSX2 for M21.

## Related tools

- **SCANet** (Python package whose GCN-to-GRN conversion wraps the GRN inference and exposes the GRNBoost2/RcisTarget workflow on module gene lists) — https://github.com/oubounyt/SCANet
- **GRNBoost2** (Gradient-boosting (GENIE3-architecture) regression method: builds a tree-based regression model per target gene to predict target expression from candidate TF profiles and retains the strongest contributors as candidate edges)
- **RcisTarget** (Validates each candidate (TF, TG) edge by testing whether the TF regulatory binding motif is significantly enriched at the cis-regulatory region of the TG; failing edges are discarded)
- **pySCENIC** (Workflow whose GRN inference + motif-validation scheme (GRNBoost2 followed by RcisTarget) is followed for edge pruning)
- **Scanpy** (Loads and preprocesses the single-cell expression matrix before TF scanning and GRN inference)

## Examples

```
pip install scanet
```

## Evaluation signals

- All retained GRN edges are motif-validated: every (TF, TG) edge passed the RcisTarget cis-regulatory enrichment test (no discarded edges remain in the directed graph)
- Reproduced M11 regulator set matches the article: GATA2, MAFF, FOSB (Table 2, Fig. 3B)
- Reproduced M21 regulator matches the article: GSX2 controlling ERRFI1 (MIG6) and AKAP12 (Fig. 3D)
- Per-gene correlation statistics match: 16 of 90 M11 genes with correlation >= 0.9 and 11 of 137 M21 genes with correlation = 1.0
- Edges are oriented TF -> TG in the assembled directed graphs

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- Reproducibility depends on gene–drug/database versions: Drugst.One databases update weekly, so identical analyses and parameters on newer datasets may yield different results

## Evidence

- [methods] RcisTarget motif validation: "RcisTarget validates for every edge (TF, TG) in the GRN, if the regulatory binding motif for the TF is significantly enriched at the cis-regulatory region"
- [results] M11 GRN regulators: "Three TFs (GATA2, MAFF, FOSB) in the GRN regulate expressed TGs of highly pro-inflammatory reactions (Table 2 and Fig. 3B)"
- [results] M21 GRN regulator: "The transcriptional regulator in the GRN was the GSX2, a known regulator of cell fate (Waclaw et al. 2009) (Fig. 3D)"
- [readme] SCANet scope: "`SCANet`, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data"
