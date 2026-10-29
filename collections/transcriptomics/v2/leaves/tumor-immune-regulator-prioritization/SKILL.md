---
name: tumor-immune-regulator-prioritization
description: Use when you have pretrained MetaTiME MeC gene-weight tables (86 MeCs)
  and want to identify and rank putative TF regulators of a cell-state or signaling
  component using ChIP-seq binding evidence.
when_to_use_negative:
- When the input is a raw scRNA-seq count matrix needing cell-state annotation rather
  than regulator inference — use the MetaTiME-Annotator workflow instead
- When no relevant ChIP-seq data exist for the TF or cell context; e.g. GR ChIP-seq
  is not available in the relevant T cell states and binding was inferred from other
  cell lines, so confidence is limited
- When analyzing non-tumor-microenvironment cells outside the scope of the MeC space,
  since MeCs were learnt from TME single cells across cancer types
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: MetaTiME
  role: Provides the pretrained MeC gene-weight table (mecs module) whose TF z-weights
    are combined with Lisa regulatory potentials
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:0acf2c24775d0c7f680dbe8df63faae611a2d6b0c9e8e92e3899a87d24d5a2f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tumor-immune-regulator-prioritization

## Summary

Prioritizes transcription factors regulating MetaTiME meta-components (MeCs) by combining MeC TF gene z-weights with Lisa2-predicted TF ChIP-seq regulatory potentials. It links data-learnt cell-state programs to putative upstream regulators, e.g. STAT1 for the interferon-response MeC-0.

## When to use

Use when you have pretrained MetaTiME MeC gene-weight tables (86 MeCs) and want to identify and rank putative TF regulators of a cell-state or signaling component using ChIP-seq binding evidence.

## When NOT to use

- When the input is a raw scRNA-seq count matrix needing cell-state annotation rather than regulator inference — use the MetaTiME-Annotator workflow instead
- When no relevant ChIP-seq data exist for the TF or cell context; e.g. GR ChIP-seq is not available in the relevant T cell states and binding was inferred from other cell lines, so confidence is limited
- When analyzing non-tumor-microenvironment cells outside the scope of the MeC space, since MeCs were learnt from TME single cells across cancer types

## Inputs

- Pretrained MetaTiME MeC gene-weight table (z-weights for 86 MeCs) loaded via the MetaTiME mecs module
- TF ChIP-seq binding data accessed through Lisa2 for regulatory potential computation
- Optional: list of target MeCs to prioritize (e.g. MeC-0, MeC-20, MeC-37)

## Outputs

- Per-MeC ranked TF regulator table written to lisa_regulator_prioritization.csv
- Prioritized top TFs per MeC (e.g. STAT1 for MeC-0, GR for MeC-20, PPARG for MeC-37)

## How to apply

Load the pretrained MeC gene-weight table using the MetaTiME mecs module and extract the top 100 genes ranked by z-weight for each of the 86 MeCs. For each top-100 gene set, run Lisa2 to compute TF regulatory potentials from TF ChIP-seq binding data. Rank TFs per MeC by comparing (e.g. integrating) the Lisa regulatory potentials against the MeC TF z-weights, so that a TF scores highly when it both binds the MeC's top genes and is itself highly weighted in the MeC program. Validate the ranking against published anchors: STAT1 as top regulator of MeC-0 (interferon response), PPARG top-ranked by Lisa in MeC-37 despite a low z-weight, and GR top-ranked for MeC-20 (CXCL13+ Tfh). Write the full per-MeC TF ranking table to lisa_regulator_prioritization.csv.

## Related tools

- **MetaTiME** (Provides the pretrained MeC gene-weight table (mecs module) whose TF z-weights are combined with Lisa regulatory potentials) — https://github.com/yi-zhang/MetaTiME

## Examples

```
from metatime import mecs; mec_weights = mecs.load_mec_gene_weights(); top100 = mec_weights['MeC-0'].nlargest(100).index.tolist()  # then run Lisa2 on top100 and merge with TF z-weights
```

## Evaluation signals

- Reproduced top regulator of MeC-0 (interferon response) is STAT1, matching Fig. 5 and Supplementary Data 2-3
- Lisa alone ranks PPARG top in MeC-37 despite PPARG's low z-weight in the MeC, showing the two signals are compared rather than trivially concatenated
- GR is top-ranked for MeC-20 (CXCL13+ Tfh), with GR ChIP-seq showing robust binding nearby the top gene CXCL13
- Each MeC's TF ranking is derived from exactly the top 100 z-weight genes of that MeC
- Output CSV lisa_regulator_prioritization.csv contains a full ranking for each of the 86 MeCs

## Limitations

- GR ChIP-seq is not available in the contexts of the relevant T cell states; binding was inferred from ChIP-seq in other cell lines
- The function of the GR pathway in different immune cell types is not fully understood
- Supporting TF annotations are referenced only in the external artifact Supplementary Data 2, not contained in the section text
- Lisa regulatory potential can dominate the ranking where the TF z-weight is low (e.g. PPARG in MeC-37), so the two evidence sources must be interpreted jointly

## Evidence

- [results] Comparing Lisa regulatory potentials with MeC gene weights prioritizes STAT1 as the top regulator of MeC-0 (interferon response), with STAT1 also highly represented in the MeC z-weights.: [evidence span withheld — anchor status: not_found]
- [intro] Incorporate transcription factor binding data to identify and prioritize putative transcriptional regulators that may modulate tumor immunity: "by incorporating transcription factor binding data, MetaTiME identifies and prioritizes putative transcriptional regulators that may modulate tumor immunity"
- [discussion] Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13.: "Though GR ChIP-seq is not available in the contexts of the relevant T cell states, GR ChIP-seq in other cell lines demonstrate robust binding nearby the top gene CXCL13"
- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment.: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states"
