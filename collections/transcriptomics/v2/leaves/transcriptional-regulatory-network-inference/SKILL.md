---
name: transcriptional-regulatory-network-inference
description: Use when a single-cell RNA-seq analysis needs cell type-specific TF-TF/TF-TG
  regulatory networks and ranked key regulators (driving forces) after cell types
  and signatures have been defined.
when_to_use_negative:
- Cell types have not yet been defined — SINCERA requires cell type identification
  and signature gene analysis to precede driving-force analysis
- The goal is upstream preprocessing (filtering, normalization, clustering) rather
  than regulatory network reconstruction
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SINCERA
  role: R S4 pipeline providing drivingfoce.selectTFs, drivingforce.selectTGs, drivingforce.inferTRN,
    drivingforce.getLCC, and drivingforce.rankTFs for TRN construction and driving-force
    ranking
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:683729608407565296894149294ce2ddc95f99bef0f35025c652ad9bb99a7b67
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# transcriptional-regulatory-network-inference

## Summary

Reconstructs a transcriptional regulatory network (TRN) for each cell type with SINCERA's driving-force functions, then identifies driving forces as the key nodes of that network by integrating six node-importance centrality metrics.

## When to use

Use when a single-cell RNA-seq analysis needs cell type-specific TF-TF/TF-TG regulatory networks and ranked key regulators (driving forces) after cell types and signatures have been defined.

## When NOT to use

- Cell types have not yet been defined — SINCERA requires cell type identification and signature gene analysis to precede driving-force analysis
- The goal is upstream preprocessing (filtering, normalization, clustering) rather than regulatory network reconstruction

## Inputs

- SINCERA S4 object constructed from a gene-by-cell expression matrix (exprfile) and samplefile
- Cell type labels from SINCERA cell type identification
- Cell type-specific differentially expressed genes and signature genes
- List of transcription factors (candidate TFs)

## Outputs

- Cell type-specific TF-TF/TF-TG transcriptional regulatory network (with largest connected component)
- Ranked table of TFs within the LCC with integrated node-importance scores identifying driving forces (key nodes)

## How to apply

Start from the SINCERA S4 object (built with construct from the gene-by-cell exprfile and samplefile). Run drivingfoce.selectTFs to assemble candidate TFs as the union of cell type-specific differentially expressed TFs (one-tailed Welch's t test p-value < 0.05) and commonly expressed TFs (expressed in at least 80% of the cell type's cells), then drivingforce.selectTGs to designate cell type-specific differentially expressed genes or signature genes as candidate targets. Run drivingforce.inferTRN with edge.threshold = 0.05 to infer the TF-TF/TF-TG network from first-order conditional dependence of expression, and verify with drivingforce.getLCC that the largest connected component (LCC) holds at least 80% of selected TFs and TGs; if it does not, re-run with a higher edge threshold. Finally, run drivingforce.rankTFs to integrate the six node-importance metrics — degree, closeness, betweenness, disruptive fragmentation, disruptive connection, and disruptive distance centrality — and rank TFs within the LCC as driving forces.

## Related tools

- **SINCERA** (R S4 pipeline providing drivingfoce.selectTFs, drivingforce.selectTGs, drivingforce.inferTRN, drivingforce.getLCC, and drivingforce.rankTFs for TRN construction and driving-force ranking) — https://github.com/xu-lab/SINCERA

## Examples

```
library(SINCERA); obj <- drivingforce.inferTRN(obj, edge.threshold = 0.05); obj <- drivingforce.getLCC(obj); obj <- drivingforce.rankTFs(obj)
```

## Evaluation signals

- drivingforce.getLCC reports that the largest connected component contains at least 80% of the selected TFs and TGs
- If the LCC-size check fails at edge.threshold = 0.05, the network is re-inferred with a higher threshold before ranking
- The ranked table from drivingforce.rankTFs lists every TF in the LCC together with its integrated importance score
- Candidate TF set equals the union of differentially expressed TFs (one-tailed Welch's t test p < 0.05) and TFs expressed in >= 80% of the cell type's cells
- Candidate target genes correspond to cell type-specific differentially expressed genes or signature genes

## Limitations

- TRN edges are inferred from first-order conditional dependence of expression, so the network captures statistical rather than direct biochemical regulation
- The LCC must retain >= 80% of selected nodes; sparse cell type-specific signal may force raising edge.threshold and alter network topology
- Driving-force analysis is cell type-specific and depends on the quality of upstream cell clustering and signature gene identification

## Evidence
