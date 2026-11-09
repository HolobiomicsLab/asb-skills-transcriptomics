---
name: centrality-based-network-node-ranking
description: Use when a scRNA-Seq analysis needs to identify the driving forces (key
  nodes) of specific cell types after cell types and signatures have been defined,
  e.g., ranking TFs regulating a cluster.
when_to_use_negative:
- Cell types have not yet been identified — SINCERA's driving-force analysis runs
  only after cell clustering and cell type identification, which 'directly influences
  all downstream analysis'.
- The goal is generic network analysis of an externally supplied graph rather than
  identifying driving forces of cell types from SINCERA's expression-derived TRN.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SINCERA
  role: R package providing drivingfoce.selectTFs, drivingforce.selectTGs, drivingforce.inferTRN,
    drivingforce.getLCC, and drivingforce.rankTFs to build the TRN and rank key TFs
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:4981cf85af8f7aad1493977381c2671a0addbdf7284648fc14147938abff2edb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# centrality-based-network-node-ranking

## Summary

Identifies driving forces (key regulators) of a cell type in SINCERA by building a transcriptional regulatory network (TRN) of candidate TFs and target genes, then ranking TFs inside its largest connected component using six integrated node-importance centrality metrics.

## When to use

Use when a scRNA-Seq analysis needs to identify the driving forces (key nodes) of specific cell types after cell types and signatures have been defined, e.g., ranking TFs regulating a cluster.

## When NOT to use

- Cell types have not yet been identified — SINCERA's driving-force analysis runs only after cell clustering and cell type identification, which 'directly influences all downstream analysis'.
- The goal is generic network analysis of an externally supplied graph rather than identifying driving forces of cell types from SINCERA's expression-derived TRN.

## Inputs

- SINCERA S4 object (from the construct function on a gene-by-cell expression matrix plus sample file)
- Cell type assignments from prior clustering/cell type identification steps
- Cell type-specific differentially expressed genes and signature genes

## Outputs

- Transcriptional regulatory network (TF-TF/TF-TG edges) per cell type
- Largest connected component statistics (fraction of selected TFs/TGs retained)
- Ranked table of every TF in the LCC with its integrated importance score

## How to apply

After cell type identification and signature gene analysis, run drivingfoce.selectTFs to assemble candidate TFs as the union of cell type-specific differentially expressed TFs (one-tailed Welch's t test p-value < 0.05) and commonly expressed TFs (expressed in at least 80% of the cell type's cells). Run drivingforce.selectTGs to designate cell type-specific differentially expressed genes or signature genes as candidate target genes. Run drivingforce.inferTRN with edge.threshold = 0.05 to infer the TF-TF/TF-TG network from first-order conditional dependence of expression, then verify with drivingforce.getLCC that the largest connected component (LCC) holds at least 80% of the selected TFs and TGs, re-running with a higher threshold if not. Finally, run drivingforce.rankTFs to integrate six node-importance metrics (degree, closeness, betweenness, disruptive fragmentation, disruptive connection, and disruptive distance centrality) and rank TFs within the LCC as key regulators.

## Related tools

- **SINCERA** (R package providing drivingfoce.selectTFs, drivingforce.selectTGs, drivingforce.inferTRN, drivingforce.getLCC, and drivingforce.rankTFs to build the TRN and rank key TFs) — https://github.com/xu-lab/SINCERA

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA)
```

## Evaluation signals

- The ranked table lists every TF in the LCC together with its integrated importance score (no TFs missing).
- drivingforce.getLCC shows the largest connected component contains at least 80% of the selected TFs and TGs before ranking is accepted.
- If the LCC check fails, edge.threshold is raised from 0.05 and drivingforce.inferTRN is re-run rather than ranking a fragmented network.
- Candidate TFs include both differentially expressed TFs (Welch's t test p-value < 0.05) and commonly expressed TFs (present in >= 80% of the cell type's cells).

## Limitations

- The LCC must retain at least 80% of selected TFs/TGs; a fragmented network requires re-running with a higher edge threshold, and results depend on this threshold choice.
- Node importance is only computed within the largest connected component, so TFs outside the LCC are not ranked.
- Candidate target genes are restricted to cell type-specific differentially expressed genes or signature genes, so regulators of non-signature targets are not considered.

## Evidence

- [intro] transcriptional regulatory network construction, and identification of driving forces (key nodes) for each cell type: "gene signature prediction, transcriptional regulatory network construction, and identification of driving forces (key nodes) for each cell type"
- [methods] Run drivingforce.inferTRN with edge.threshold = 0.05: [evidence span withheld — anchor status: not_found]
- [methods] Run drivingfoce.selectTFs to assemble candidate TFs: [evidence span withheld — anchor status: not_found]
- [methods] Run drivingforce.rankTFs to integrate the six node-importance metrics: [evidence span withheld — anchor status: not_found]
- [readme] devtools::install_github("xu-lab/SINCERA"): "devtools::install_github("xu-lab/SINCERA")"
