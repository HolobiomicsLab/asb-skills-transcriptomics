---
name: single-cell-meta-component-interpretation
description: Use when annotating data-driven single-cell gene programs (86 MetaTiME
  MeCs) with candidate transcriptional regulators using TF ChIP-seq binding evidence,
  or when validating TF prioritizations for tumor immune cell states.
when_to_use_negative:
- The goal is cell-state annotation or signature scoring of new scRNA-seq data rather
  than TF regulator prioritization — use the MetaTiME-Annotator projection workflow
  instead
- The gene program of interest is not one of the 86 pretrained MeCs and has no z-weight
  table to compare against Lisa potentials
- Only ChIP-seq evidence is available without gene weights (or vice versa) — the prioritization
  requires both signals per MeC
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: MetaTiME
  role: Provides the pretrained MeC gene-weight table (mecs module) whose z-weights
    are combined with Lisa regulatory potentials to rank TFs per MeC
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
  source_content_hash: sha256:be30d39a3cbb69ce8df9027d3e2a08efac34d3ba96b0f3dcedcc2d16af34e223
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - Lisa2
  - MetaTiME
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-cell-meta-component-interpretation

## Summary

Prioritizes transcription factor regulators of MetaTiME meta-components (MeCs) by integrating the pretrained MeC gene z-weight table with Lisa-predicted TF ChIP-seq regulatory potentials, e.g. STAT1 as top regulator of MeC-0 (interferon response).

## When to use

Use when annotating data-driven single-cell gene programs (86 MetaTiME MeCs) with candidate transcriptional regulators using TF ChIP-seq binding evidence, or when validating TF prioritizations for tumor immune cell states.

## When NOT to use

- The goal is cell-state annotation or signature scoring of new scRNA-seq data rather than TF regulator prioritization — use the MetaTiME-Annotator projection workflow instead
- The gene program of interest is not one of the 86 pretrained MeCs and has no z-weight table to compare against Lisa potentials
- Only ChIP-seq evidence is available without gene weights (or vice versa) — the prioritization requires both signals per MeC

## Inputs

- Pretrained MetaTiME MeC gene-weight table (z-weights for 86 MeCs, from the MetaTiME mecs module)
- TF ChIP-seq binding data accessible through Lisa2

## Outputs

- Per-MeC TF regulator ranking table (lisa_regulator_prioritization.csv)
- Top-100 gene sets ranked by z-weight for each of the 86 MeCs
- Prioritized top regulators per MeC (e.g. STAT1 for MeC-0, PPARG for MeC-37, GR for MeC-20)

## How to apply

Load the pretrained MeC gene-weight table via the MetaTiME mecs module and extract the top 100 genes ranked by z-weight for each of the 86 MeCs. For each top-100 gene set, run Lisa2 to compute TF regulatory potentials from TF ChIP-seq binding data, then rank TFs per MeC by comparing Lisa regulatory potentials against the MeC TF z-weights within each component. Validate the prioritization against known outcomes: STAT1 should emerge as top regulator of MeC-0 (interferon response, with STAT1 highly represented in the MeC-0 z-weights), PPARG top-ranked by Lisa in MeC-37 despite a low z-weight, and GR top-ranked for MeC-20 (CXCL13+ Tfh). The rationale is that combining epigenetic binding evidence with data-learnt gene weights separates true drivers from components where a TF ranks highly by only one signal. Write the full per-MeC ranking to lisa_regulator_prioritization.csv for downstream comparison with Fig. 5 and Supplementary Data 2-3.

## Related tools

- **MetaTiME** (Provides the pretrained MeC gene-weight table (mecs module) whose z-weights are combined with Lisa regulatory potentials to rank TFs per MeC) — https://github.com/yi-zhang/MetaTiME
- **Lisa2** (Computes TF regulatory potentials from TF ChIP-seq binding data for each MeC top-100 gene set)

## Examples

```
from metatime import mecs; mec_weights = mecs.load_mec_gene_weights(); top100 = mec_weights.iloc[:, 0].nlargest(100).index.tolist()  # then run Lisa2 per MeC top-100 set
```

## Evaluation signals

- Reproduced top regulator for MeC-0 (interferon response) is STAT1, with STAT1 also highly represented in the MeC-0 z-weights
- Lisa top-ranks PPARG for MeC-37 despite PPARG having a low z-weight in that MeC (agreement check between the two signals)
- GR is top-ranked for MeC-20 (CXCL13+ Tfh), matching Fig. 5 and Supplementary Data 2-3
- The ranking table covers TFs for all 86 MeCs and is written to lisa_regulator_prioritization.csv with a per-MeC ranking schema

## Limitations

- GR ChIP-seq data is not available in the contexts of the relevant T cell states; GR binding was inferred from other cell lines
- The function of the GR pathway in different immune cell types is not fully understood
- Supporting TF annotations are documented only in external artifacts (Supplementary Data 2-3), not in the section text

## Evidence

- [abstract] Leveraging epigenetics data, MetaTiME reveals critical transcriptional regulators for the cell states.: "Leveraging epigenetics data, MetaTiME reveals critical transcriptional regulators for the cell states."
- [intro] by incorporating transcription factor binding data, MetaTiME identifies and prioritizes putative transcriptional regulators that may modulate tumor immunity.: "by incorporating transcription factor binding data, MetaTiME identifies and prioritizes putative transcriptional regulators that may modulate tumor immunity."
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [discussion] Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13.: "Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13."
- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment.: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
