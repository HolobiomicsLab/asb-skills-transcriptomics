---
name: network-connected-component-analysis
description: Use when a cell type's driving forces (key nodes) must be identified
  and a gene-gene regulatory network needs to be inferred, checked for connectivity,
  and its key nodes ranked.
when_to_use_negative:
- The goal is differential expression or signature-gene prediction only, without regulatory-network
  or key-node (driving force) analysis
- Expression data have not been preprocessed through the SINCERA pipeline (filtering,
  normalization, clustering, cell type identification), so no cell types or candidate
  DE genes exist
- You need genome-wide regulatory inference with external binding evidence rather
  than expression-based conditional dependence networks
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SINCERA
  role: R S4 pipeline whose drivingforce.* functions (selectTFs, selectTGs, inferTRN,
    getLCC, rankTFs) infer the regulatory network, extract the LCC, and rank key regulators
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:aa603b19c69a68e5845c7b413cabeb1f19e918ccdb9013dd7283f4ce7116661d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# network-connected-component-analysis

## Summary

Within the SINCERA driving-force analysis, infer a TF-TF/TF-TG transcriptional regulatory network from first-order conditional dependence of expression, then extract and verify the largest connected component (LCC) before ranking TFs as key regulators (driving forces) of a cell type.

## When to use

Use when a cell type's driving forces (key nodes) must be identified and a gene-gene regulatory network needs to be inferred, checked for connectivity, and its key nodes ranked.

## When NOT to use

- The goal is differential expression or signature-gene prediction only, without regulatory-network or key-node (driving force) analysis
- Expression data have not been preprocessed through the SINCERA pipeline (filtering, normalization, clustering, cell type identification), so no cell types or candidate DE genes exist
- You need genome-wide regulatory inference with external binding evidence rather than expression-based conditional dependence networks

## Inputs

- SINCERA S4 object (constructed from a gene-by-cell expression matrix via exprfile and samplefile with the construct function)
- Cell type assignments from SINCERA clustering
- Candidate TF list (cell type-specific DE TFs plus commonly expressed TFs)
- Candidate target gene list (cell type-specific DE genes or signature genes)

## Outputs

- Inferred TF-TF/TF-TG transcriptional regulatory network at edge.threshold = 0.05
- Largest connected component (LCC) of the regulatory network
- Ranked table listing every TF in the LCC with its integrated importance score across six centrality metrics

## How to apply

Assemble candidate regulators with drivingfoce.selectTFs as the union of cell type-specific differentially expressed TFs (one-tailed Welch's t test p-value < 0.05) and commonly expressed TFs (expressed in at least 80% of the cell type's cells), then select candidate targets with drivingforce.selectTGs from cell type-specific differentially expressed genes or signature genes. Infer the network with drivingforce.inferTRN using edge.threshold = 0.05, building TF-TF/TF-TG edges on first-order conditional dependence of expression. Verify with drivingforce.getLCC that the largest connected component retains at least 80% of the selected TFs and TGs; if not, re-run inference with a higher edge threshold. Only after the LCC-size check passes, run drivingforce.rankTFs to integrate six node-importance metrics (degree, closeness, betweenness, disruptive fragmentation, disruptive connection, and disruptive distance centrality) and rank TFs within the LCC as key regulators.

## Related tools

- **SINCERA** (R S4 pipeline whose drivingforce.* functions (selectTFs, selectTGs, inferTRN, getLCC, rankTFs) infer the regulatory network, extract the LCC, and rank key regulators) — https://github.com/xu-lab/SINCERA

## Examples

```
library(SINCERA); drivingforce.inferTRN(s4, edge.threshold = 0.05); drivingforce.getLCC(s4); drivingforce.rankTFs(s4)
```

## Evaluation signals

- drivingforce.getLCC reports that the LCC contains >= 80% of the selected TFs and TGs before ranking is accepted
- If the 80% LCC-size check fails, inferTRN is re-run with a higher edge threshold until it passes
- The ranked output table lists every TF present in the LCC, each with an integrated importance score combining degree, closeness, betweenness, and the three disruptive centrality metrics
- Candidate TFs satisfy the stated criteria: DE TFs at one-tailed Welch's t test p-value < 0.05 or expression in >= 80% of the cell type's cells

## Limitations

- Network inference relies on first-order conditional dependence of expression only (no external TF-binding evidence), so inferred edges are statistical associations
- The LCC-size check (>= 80% of selected TFs/TGs) may require iterating edge.threshold upward; a too-low threshold can fragment the network
- Ranking is restricted to TFs within the largest connected component; disconnected nodes are excluded from driving-force ranking

## Evidence

- [methods] SINCERA handles driving-force identification by first constructing a transcriptional regulatory network for each cell type and then identifying the driving forces as the key nodes of that network.: [evidence span withheld — anchor status: not_found]
- [other] Run drivingforce.inferTRN with edge.threshold = 0.05 to infer the TF-TF/TF-TG network based on first-order conditional dependence of expression: [evidence span withheld — anchor status: not_found]
