---
name: gene-set-transcription-factor-enrichment-analysis
description: Use when you have gene programs (e.g., MetaTiME MeCs) with gene-level
  weights and want to identify putative upstream TF regulators by intersecting TF
  binding/potential scores with the program's TF weights.
when_to_use_negative:
- The gene set has no associated weights, so TF membership cannot be cross-checked
  against program-level z-weights — plain Lisa regulatory-potential ranking alone
  can misprioritize (e.g., PPARG in MeC-37).
- No ChIP-seq data exist for the TF in the relevant cell-state context; binding must
  then be inferred from other cell lines and treated cautiously (as with GR for CXCL13+
  Tfh).
edam_operation: null
tools:
- name: MetaTiME
  role: Provides the pretrained MeC gene-weight table (mecs module) whose top genes
    and TF z-weights are combined with Lisa regulatory potentials
  repo: github.com/yi-zhang/MetaTiME
- name: Lisa
  role: Lisa2 computes TF regulatory potentials from TF ChIP-seq binding data for
    each MeC top-100 gene set
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:9fc25719d627b790bb9edf34fd4cd5eef7e1ccda560e25c974b528ad62629e5f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lisa
  - MetaTiME
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-set-transcription-factor-enrichment-analysis

## Summary

Prioritizes transcription factor (TF) regulators of MetaTiME meta-components (MeCs) by comparing Lisa2-predicted ChIP-seq regulatory potentials for the top genes of each MeC against the TF z-weights encoded in the pretrained MeC gene-weight table.

## When to use

Use when you have gene programs (e.g., MetaTiME MeCs) with gene-level weights and want to identify putative upstream TF regulators by intersecting TF binding/potential scores with the program's TF weights.

## When NOT to use

- The gene set has no associated weights, so TF membership cannot be cross-checked against program-level z-weights — plain Lisa regulatory-potential ranking alone can misprioritize (e.g., PPARG in MeC-37).
- No ChIP-seq data exist for the TF in the relevant cell-state context; binding must then be inferred from other cell lines and treated cautiously (as with GR for CXCL13+ Tfh).

## Inputs

- Pretrained MetaTiME MeC gene-weight table (86 MeCs with per-gene z-weights, via the MetaTiME mecs module)
- Top-100 gene sets per MeC ranked by z-weight
- TF ChIP-seq binding data consumed through Lisa2 regulatory potential computation

## Outputs

- Per-MeC TF regulator ranking table (e.g., lisa_regulator_prioritization.csv)
- Prioritized top TF regulators per MeC (e.g., STAT1 for MeC-0, GR for MeC-20)

## How to apply

Load the pretrained MeC gene-weight table via the MetaTiME mecs module and extract the top 100 genes ranked by z-weight for each of the 86 MeCs. For each top-100 gene set, run Lisa2 to compute TF regulatory potentials from TF ChIP-seq binding data, then rank TFs per MeC by combining Lisa regulatory potentials with the MeC TF z-weights (e.g., PPARG can rank high by Lisa potential in MeC-37 even with a low z-weight, so both signals matter). Validate against reported outcomes: STAT1 as the top regulator of MeC-0 (interferon response), GR top-ranked for MeC-20 (CXCL13+ Tfh). Write the full per-MeC TF ranking table (e.g., lisa_regulator_prioritization.csv) for inspection against Fig. 5 and Supplementary Data 2-3.

## Related tools

- **MetaTiME** (Provides the pretrained MeC gene-weight table (mecs module) whose top genes and TF z-weights are combined with Lisa regulatory potentials) — github.com/yi-zhang/MetaTiME
- **Lisa** (Lisa2 computes TF regulatory potentials from TF ChIP-seq binding data for each MeC top-100 gene set)

## Evaluation signals

- STAT1 is reproduced as the top regulator of MeC-0 (interferon response), matching Fig. 5 and Supplementary Data 2-3.
- GR is reproduced as top-ranked for MeC-20 (CXCL13+ Tfh).
- The per-MeC ranking covers all 86 MeCs and is written as a complete table (lisa_regulator_prioritization.csv).
- TF rankings reflect both Lisa regulatory potentials and MeC z-weights, so high-Lisa/low-z-weight cases (e.g., PPARG in MeC-37) are flagged rather than silently ranked top.
- Each MeC's input gene set contains exactly the top 100 genes by z-weight from the pretrained table.

## Limitations

- GR ChIP-seq data are not available in the contexts of the relevant T cell states, so binding must be inferred from other cell lines.
- The function of the GR pathway in different immune cell types is not fully understood.
- Supporting TF annotations are referenced in external artifact Supplementary Data 2 rather than the article text.
- Prioritization depends on ChIP-seq coverage; a TF without context-appropriate ChIP-seq may be missed or misranked.

## Evidence

- [discussion] Leveraging ChIP-seq data to implicate critical transcriptional regulators in tumor immunity: "By leveraging ChIP-seq data, MetaTiME implicates critical transcriptional regulators in tumor immunity."
- [intro] TF prioritization incorporates binding data: "by incorporating transcription factor binding data, MetaTiME identifies and prioritizes putative transcriptional regulators that may modulate tumor immunity."
- [abstract] Epigenetics-based regulator discovery: "Leveraging epigenetics data, MetaTiME reveals critical transcriptional regulators for the cell states."
- [discussion] GR binding caveat for Tfh states: "Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13."
