---
name: chip-seq-regulatory-potential-annotation
description: Use when a data-learned gene program (e.g., one of the 86 MeCs) needs
  candidate transcriptional regulators, and TF ChIP-seq binding potentials (Lisa)
  can be combined with per-gene program weights.
when_to_use_negative:
- When the goal is cell-state annotation or signature scoring on new scRNA-seq data
  rather than regulator prioritization — use the MetaTiME-Annotator projection workflow
  instead
- When no TF ChIP-seq data are available for the relevant cell state and inference
  from other cell lines would be inappropriate for the claim being made
- When the gene program is not defined with per-gene weights (z-weights), since ranking
  relies on comparing Lisa potentials with in-program TF weights
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: MetaTiME
  role: Provides the pretrained MeC gene-weight table (mecs module) whose top-100
    z-weight gene sets and TF z-weights are the anchor for regulator prioritization
  repo: https://github.com/yi-zhang/MetaTiME
- name: Lisa2
  role: Computes TF regulatory potentials from TF ChIP-seq binding data for each MeC
    top-100 gene set
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:2ef53673472fca4ab523ef2e31a7dd8c0b3f754b7b3c333b61aa9c604a781349
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - Lisa2
  - MetaTiME
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chip-seq-regulatory-potential-annotation

## Summary

Prioritizes putative transcriptional regulators of MetaTiME meta-components (MeCs) by comparing Lisa ChIP-seq-derived regulatory potentials for TFs against the TF gene weights (z-weights) inside each MeC, linking learned cell states to candidate upstream regulators.

## When to use

Use when a data-learned gene program (e.g., one of the 86 MeCs) needs candidate transcriptional regulators, and TF ChIP-seq binding potentials (Lisa) can be combined with per-gene program weights.

## When NOT to use

- When the goal is cell-state annotation or signature scoring on new scRNA-seq data rather than regulator prioritization — use the MetaTiME-Annotator projection workflow instead
- When no TF ChIP-seq data are available for the relevant cell state and inference from other cell lines would be inappropriate for the claim being made
- When the gene program is not defined with per-gene weights (z-weights), since ranking relies on comparing Lisa potentials with in-program TF weights

## Inputs

- Pretrained MeC gene-weight table (z-weights for 86 meta-components) from the MetaTiME mecs module
- Top-100 gene sets per MeC ranked by z-weight
- TF ChIP-seq binding data used by Lisa2 to compute regulatory potentials

## Outputs

- Per-MeC TF ranking table combining Lisa regulatory potentials with MeC TF z-weights (e.g., lisa_regulator_prioritization.csv)
- Prioritized top regulators per MeC (e.g., STAT1 for MeC-0, PPARG for MeC-37, GR for MeC-20/CXCL13+ Tfh)

## How to apply

Load the pretrained MeC gene-weight table from the MetaTiME mecs module and extract the top 100 genes ranked by z-weight for each MeC (86 components). For each top-100 gene set, run Lisa2 to compute TF regulatory potentials from TF ChIP-seq binding data. Rank TFs per MeC by intersecting/jointly ranking the Lisa regulatory potentials with the MeC TF z-weights, so a TF is prioritized when it scores highly on both axes (e.g., STAT1 ranks top for MeC-0/interferon response on both). Note that a TF can be top-ranked by Lisa alone despite a low z-weight (e.g., PPARG in MeC-37), so report both sources of evidence. Write the full per-MeC TF ranking table (e.g., lisa_regulator_prioritization.csv) for downstream validation.

## Related tools

- **MetaTiME** (Provides the pretrained MeC gene-weight table (mecs module) whose top-100 z-weight gene sets and TF z-weights are the anchor for regulator prioritization) — https://github.com/yi-zhang/MetaTiME
- **Lisa2** (Computes TF regulatory potentials from TF ChIP-seq binding data for each MeC top-100 gene set)

## Evaluation signals

- Reproduced top regulator of MeC-0 (interferon response) is STAT1, which is also highly represented in the MeC-0 z-weights
- Lisa alone top-ranks PPARG in MeC-37 despite a low PPARG z-weight — this divergent case should appear in the ranking table
- GR is top-ranked for MeC-20 (CXCL13+ Tfh), matching outcomes reported in Fig. 5 and Supplementary Data 2-3
- Each of the 86 MeCs yields a top-100 gene set and a completed TF ranking row in the output CSV
- Rankings reconcile both evidence axes: TFs prioritized by joint high Lisa potential AND high MeC z-weight are distinguished from Lisa-only hits

## Limitations

- GR ChIP-seq is not available in the contexts of the relevant T cell states; GR binding is inferred from ChIP-seq in other cell lines
- The function of the GR pathway in different immune cell types is not fully understood
- Supporting TF annotations may only be available in external artifacts (Supplementary Data 2) rather than in the section text
- Lisa-only top regulators can have low z-weights in the MeC, so regulatory potential and program membership must be interpreted as complementary, not redundant, evidence

## Evidence

- [abstract] abstract epigenetics: "Leveraging epigenetics data, MetaTiME reveals critical transcriptional regulators for the cell states."
- [intro] intro TF binding: "by incorporating transcription factor binding data, MetaTiME identifies and prioritizes putative transcriptional regulators that may modulate tumor immunity."
- [discussion] discussion ChIP-seq: "By leveraging ChIP-seq data, MetaTiME implicates critical transcriptional regulators in tumor immunity."
- [discussion] discussion GR caveat: "Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13."
