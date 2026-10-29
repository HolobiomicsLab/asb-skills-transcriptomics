---
name: enrichr-api-pathway-query
description: Use when a gene list (e.g. in silico knockout essential genes) must be
  interpreted for pathway enrichment beyond the metabolic model's own Recon3D_01 subSystems,
  via Enrichr/KEGG plus FerrDb annotation.
when_to_use_negative:
- The question is restricted to pathways already captured by the Recon3D_01 subSystems
  of the context-specific model — the point of this skill is to find enrichment beyond
  subSystems.
- The gene list has not been filtered for essentiality or model context (e.g. raw
  DEG lists), so enrichments would not be attributable to context-specific essential
  genes.
- Single-gene 'enrichments' would be the dominant result (very short gene lists),
  which the workflow explicitly discards.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: Enrichr
  role: Web API used to perform pathway enrichment of the SKO/DKO essential gene lists
    against the KEGG pathway library (Kuleshov et al., 2016)
- name: KEGG
  role: Pathway library queried via Enrichr for SKO and DKO pathway analysis
- name: Recon3D_01
  role: Generic metabolic model whose subSystems serve as the pathway background for
    the enrichment analysis
- name: R
  role: Implementation environment for submitting gene lists, filtering enrichment
    results, and compiling the CSV output
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:b8efed1d8a00da224491d6bbf0452511c75f70c81425392f841b33e69038ed2f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - Enrichr
  - KEGG
  - R
  - Recon3D_01
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Enrichr API pathway query

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Submits context-specific essential gene lists (from SKO/DKO in silico knockouts of SARS-CoV-2-infected lung metabolic models) to the Enrichr API against the KEGG pathway library, using Recon3D_01 subSystems as the pathway background, then cross-references hits with FerrDb ferroptosis gene classifications. This surfaces pathways beyond those captured by the metabolic model's subSystems.

## When to use

Use when a gene list (e.g. in silico knockout essential genes) must be interpreted for pathway enrichment beyond the metabolic model's own Recon3D_01 subSystems, via Enrichr/KEGG plus FerrDb annotation.

## When NOT to use

- The question is restricted to pathways already captured by the Recon3D_01 subSystems of the context-specific model — the point of this skill is to find enrichment beyond subSystems.
- The gene list has not been filtered for essentiality or model context (e.g. raw DEG lists), so enrichments would not be attributable to context-specific essential genes.
- Single-gene 'enrichments' would be the dominant result (very short gene lists), which the workflow explicitly discards.

## Inputs

- Predicted essential gene lists from SKO and DKO in silico knockout analysis of context-specific infected lung models
- Recon3D_01 model subSystems used as the pathway background
- FerrDb ferroptosis gene classifications (drivers, suppressors, markers)

## Outputs

- Filtered KEGG enrichment terms (p-value/adjusted p-value < 0.05, no single-gene enrichments)
- FerrDb cross-reference of essential genes to ferroptosis roles (e.g. SLC7A11/SLC3A2 suppressors, SLC7A5 marker)
- Single results table (CSV) combining enrichment terms and FerrDb annotations

## How to apply

Load the predicted essential gene lists from the SKO and DKO in silico knockout analysis of the context-specific infected lung models. Submit each list to the Enrichr API (Kuleshov et al., 2016) against the KEGG pathway library, passing the Recon3D_01 subSystems as the pathway background so enrichments are judged relative to the modeled metabolic space. Retain only terms with p-value/adjusted p-value < 0.05 and discard all enrichment results based on only one gene to remove single-gene artifacts. Separately download FerrDb ferroptosis gene classifications (drivers, suppressors, markers) and cross-reference the essential genes: in this analysis SLC7A11 and SLC3A2 resolve as ferroptosis suppressors and SLC7A5 as a ferroptosis marker. Compile the surviving KEGG terms and the FerrDb cross-reference into a single CSV results table.

## Related tools

- **Enrichr** (Web API used to perform pathway enrichment of the SKO/DKO essential gene lists against the KEGG pathway library (Kuleshov et al., 2016))
- **KEGG** (Pathway library queried via Enrichr for SKO and DKO pathway analysis)
- **Recon3D_01** (Generic metabolic model whose subSystems serve as the pathway background for the enrichment analysis)
- **R** (Implementation environment for submitting gene lists, filtering enrichment results, and compiling the CSV output)

## Examples

```
curl -X POST -F 'list=SLC3A2,SLC7A11,SLC7A5' -F 'description=SKO_essential_genes' https://maayanlab.cloud/Enrichr/addList
```

## Evaluation signals

- All retained enrichment terms have p-value/adjusted p-value < 0.05.
- No retained enrichment result is based on only one gene.
- The output table reproduces the reported enrichment of pyrimidine metabolism, oxidative stress, and ferroptosis, matching the abstract's key-pathway finding.
- FerrDb cross-reference identifies SLC7A11 and SLC3A2 as ferroptosis suppressors and SLC7A5 as a ferroptosis marker.
- Ferroptosis enrichment is attributable specifically to SLC3A2 and SLC7A11 and is observed in non-ACE2 vector samples.

## Limitations

- Findings are limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE); enrichments may not generalize to other tissues or cell types.
- Ferroptosis was the only Enrichr-derived pathway associated with non-ACE2 vector samples, so pathway coverage for other sample groups may be sparse after filtering.
- Enrichment conclusions depend on the background choice (Recon3D_01 subSystems); using a different background changes the significance of terms.

## Evidence

- [results] An Enrichr pathway analysis was performed.: [evidence span withheld — anchor status: not_found]
- [full_text] Ferroptosis was the only Enrichr-derived pathway associated with non-ACE2 vector samples.: [evidence span withheld — anchor status: not_found]
