---
name: transcription-factor-candidate-selection
description: Use when a cell type has already been identified in a scRNA-Seq dataset
  (SINCERA S4 object) and you need to find its driving forces — key transcription
  factors — from differential expression and network centrality.
when_to_use_negative:
- Cell types have not yet been identified — SINCERA's driving-force step presumes
  clustering and cell type identification have already been run, since candidates
  come from cell type-specific differential expression
- The goal is differential expression or gene signature prediction only, without regulatory-network
  interpretation — the preceding SINCERA stages cover that without needing TRN inference
- Expression data has not been normalized (e.g. per-sample z-score via normalization.zscore),
  so cross-cell-type comparisons and conditional-dependence inference would be invalid
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINCERA
  role: R S4 pipeline providing drivingfoce.selectTFs, drivingforce.selectTGs, drivingforce.inferTRN,
    drivingforce.getLCC, and drivingforce.rankTFs for driving-force identification
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:7a619f5d28680ed60f14a443fe6f824820fbb68b849dd0f474b7b6adc59ae17f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# transcription-factor-candidate-selection

## Summary

Identifies driving forces (key regulatory nodes) of a given cell type in SINCERA by assembling candidate TFs, selecting candidate target genes, inferring a TF-TF/TF-TG transcriptional regulatory network, and ranking TFs by integrated node-importance metrics within its largest connected component.

## When to use

Use when a cell type has already been identified in a scRNA-Seq dataset (SINCERA S4 object) and you need to find its driving forces — key transcription factors — from differential expression and network centrality.

## When NOT to use

- Cell types have not yet been identified — SINCERA's driving-force step presumes clustering and cell type identification have already been run, since candidates come from cell type-specific differential expression
- The goal is differential expression or gene signature prediction only, without regulatory-network interpretation — the preceding SINCERA stages cover that without needing TRN inference
- Expression data has not been normalized (e.g. per-sample z-score via normalization.zscore), so cross-cell-type comparisons and conditional-dependence inference would be invalid

## Inputs

- SINCERA S4 object (built from a gene-by-cell expression exprfile and samplefile) with cell types already assigned
- Cell type-specific differentially expressed genes and/or signature genes for the cell type of interest

## Outputs

- Ranked table of every TF in the LCC with its integrated importance score (driving-force ranking)
- TF-TF/TF-TG transcriptional regulatory network with its largest connected component

## How to apply

After building the SINCERA R S4 object via construct, run drivingfoce.selectTFs to assemble candidate TFs as the union of cell type-specific differentially expressed TFs (one-tailed Welch's t test p-value < 0.05) and commonly expressed TFs (expressed in at least 80% of the cell type's cells). Then run drivingforce.selectTGs to designate cell type-specific differentially expressed or signature genes as candidate targets, and drivingforce.inferTRN with edge.threshold = 0.05 to infer the TF-TF/TF-TG network from first-order conditional dependence of expression. Verify with drivingforce.getLCC that the largest connected component (LCC) contains at least 80% of the selected TFs and TGs, re-running with a higher edge threshold if not. Finally, run drivingforce.rankTFs to integrate six node-importance metrics — degree, closeness, betweenness, disruptive fragmentation, disruptive connection, and disruptive distance centrality — and rank TFs within the LCC as the driving forces.

## Related tools

- **SINCERA** (R S4 pipeline providing drivingfoce.selectTFs, drivingforce.selectTGs, drivingforce.inferTRN, drivingforce.getLCC, and drivingforce.rankTFs for driving-force identification) — https://github.com/xu-lab/SINCERA

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA); tfs <- drivingfoce.selectTFs(s4obj)
```

## Evaluation signals

- The ranked output table lists every TF present in the LCC, each with an integrated node-importance score
- The LCC-size check passes: the largest connected component holds >= 80% of the selected TFs and TGs before ranking is accepted
- The candidate TF set equals the union of TFs with one-tailed Welch's t test p-value < 0.05 and TFs expressed in >= 80% of the cell type's cells
- The TRN was inferred with edge.threshold = 0.05 based on first-order conditional dependence of expression; if the LCC check failed, a higher threshold was used on re-run
- All six importance metrics (degree, closeness, betweenness, disruptive fragmentation, disruptive connection, disruptive distance centrality) are integrated in the ranking

## Limitations

- The driving-force component requires cell types to be defined first; clustering quality 'directly influences all downstream analysis'
- If the largest connected component holds fewer than 80% of selected TFs/TGs at edge.threshold = 0.05, the network must be re-inferred with a higher threshold, which can prune true edges
- Candidate TFs and TGs are restricted to cell type-specific differentially expressed or signature genes, so regulators lacking differential expression may be missed

## Evidence

- [other] Run drivingfoce.selectTFs to assemble candidate TFs as the union of cell type-specific differentially expressed TFs (one-tailed Welch's t test p-value < 0.05): [evidence span withheld — anchor status: not_found]
- [other] Run drivingforce.inferTRN with edge.threshold = 0.05 to infer the TF-TF/TF-TG network based on first-order conditional dependence of expression: [evidence span withheld — anchor status: not_found]
- [other] Run drivingforce.rankTFs to integrate the six node-importance metrics (degree, closeness, betweenness, disruptive fragmentation, disruptive connection, and disruptive distance centrality): [evidence span withheld — anchor status: not_found]
