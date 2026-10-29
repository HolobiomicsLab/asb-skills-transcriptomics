---
name: kegg-pathway-interpretation
description: Use when a list of essential genes from SKO/DKO in silico knockout of
  context-specific metabolic models must be functionally interpreted via KEGG enrichment
  with a custom background and ferroptosis annotation via FerrDb.
when_to_use_negative:
- Essential genes have no pathways in Recon3D_01 subSystems — such genes (and their
  connected drugs) were discarded from the article's network, so enrichment has no
  valid background
- Any surviving enrichment would rest on only one gene — the workflow explicitly discards
  single-gene enrichment results
- The question requires direct targeting of the viral proteome — here drugs/genes
  are assessed by network effects on the host metabolome with the virus modeled only
  through its biomass function
edam_operation: http://edamontology.org/operation_3939
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Enrichr
  role: Web/API service used to submit essential gene lists against the KEGG pathway
    library with the Recon3D_01 subSystems background
- name: KEGG
  role: Pathway library providing the enrichment terms for SKO and DKO essential gene
    lists
- name: Recon3D_01
  role: Generic genome-scale metabolic model whose subSystems define the pathway background
    for enrichment
- name: R
  role: Implementation environment for the enrichment and FerrDb cross-reference workflow
- name: DCcov
  role: Repository of the analysis pipeline that produced the essential gene lists
    being interpreted
  repo: https://github.com/sysbiolux/DCcov
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:1bbc5716ec873a1a2b857c508405c4523b6a50bc985705c395ae0799642542b9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  repo_url: https://github.com/sysbiolux/DCcov
  tools:
  - DCcov
  - Enrichr
  - KEGG
  - R
  - Recon3D_01
  edam_operation: http://edamontology.org/operation_3939
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# kegg-pathway-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functionally interpret lists of predicted essential genes from in silico gene deletion by Enrichr pathway analysis against the KEGG library, using Recon3D_01 subSystems as the pathway background, and cross-reference hits against FerrDb ferroptosis classifications (drivers, suppressors, markers).

## When to use

Use when a list of essential genes from SKO/DKO in silico knockout of context-specific metabolic models must be functionally interpreted via KEGG enrichment with a custom background and ferroptosis annotation via FerrDb.

## When NOT to use

- Essential genes have no pathways in Recon3D_01 subSystems — such genes (and their connected drugs) were discarded from the article's network, so enrichment has no valid background
- Any surviving enrichment would rest on only one gene — the workflow explicitly discards single-gene enrichment results
- The question requires direct targeting of the viral proteome — here drugs/genes are assessed by network effects on the host metabolome with the virus modeled only through its biomass function

## Inputs

- Predicted essential gene lists from SKO and DKO in silico knockout of context-specific infected lung models (A549, Calu-3, H1299, NHBE)
- Recon3D_01 subSystems used as the KEGG enrichment background
- FerrDb ferroptosis gene classifications (drivers, suppressors, markers)

## Outputs

- Filtered KEGG enrichment table (terms with p-value/adjusted p-value < 0.05, no single-gene enrichments)
- CSV results table combining surviving KEGG terms and FerrDb ferroptosis annotations (e.g. SLC7A11/SLC3A2 suppressors, SLC7A5 marker)

## How to apply

Load the predicted essential gene lists from the SKO and DKO in silico knockout of the context-specific infected lung models built with rFASTCORMICS on Recon3D_01. Submit each gene list to the Enrichr API (Kuleshov et al., 2016) against the KEGG pathway library, explicitly passing the Recon3D_01 subSystems as the pathway background so enrichments are judged within the modeled metabolic network. Retain only terms with p-value/adjusted p-value < 0.05 and discard all enrichment results based on only one gene, since single-gene hits are uninformative. Independently download FerrDb ferroptosis gene classifications (drivers, suppressors, markers) and cross-reference the essential genes: in this study this identified SLC7A11 and SLC3A2 as ferroptosis suppressors and SLC7A5 as a ferroptosis marker. Compile the surviving KEGG terms and FerrDb cross-reference into a single results table (CSV); the analysis is validated if the table reproduces enrichment of pyrimidine metabolism, oxidative stress, and ferroptosis as reported for COVID-19 severity in lung tissue.

## Related tools

- **Enrichr** (Web/API service used to submit essential gene lists against the KEGG pathway library with the Recon3D_01 subSystems background)
- **KEGG** (Pathway library providing the enrichment terms for SKO and DKO essential gene lists)
- **Recon3D_01** (Generic genome-scale metabolic model whose subSystems define the pathway background for enrichment)
- **R** (Implementation environment for the enrichment and FerrDb cross-reference workflow)
- **DCcov** (Repository of the analysis pipeline that produced the essential gene lists being interpreted) — https://github.com/sysbiolux/DCcov

## Examples

```
curl -X POST https://maayanlab.cloud/Enrichr/addList -F 'list=SLC3A2,SLC7A11,SLC7A5' -F 'description=sko_essential_genes'
```

## Evaluation signals

- All retained KEGG terms meet the p-value/adjusted p-value < 0.05 cutoff
- No enrichment results based on only one gene remain in the output table
- The CSV output reproduces the reported enrichment of pyrimidine metabolism, oxidative stress, and ferroptosis
- FerrDb cross-reference correctly classifies SLC7A11 and SLC3A2 as ferroptosis suppressors and SLC7A5 as a ferroptosis marker
- Ferroptosis enrichment is attributable specifically to SLC3A2 and SLC7A11 and appears only for non-ACE2 vector samples

## Limitations

- Ferroptosis was the only Enrichr-derived pathway associated with non-ACE2 vector samples, so findings are conditional on the sample/vector configuration
- Analysis is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE)
- Discarding single-gene enrichments can remove plausible but underpowered pathway hits

## Evidence

- [results] Ferroptosis was the only Enrichr-derived pathway associated with non-ACE2 vector samples: [evidence span withheld — anchor status: not_found]
- [results] an Enrichr pathway analysis was performed (Kuleshov et al., 2016): [evidence span withheld — anchor status: not_found]
