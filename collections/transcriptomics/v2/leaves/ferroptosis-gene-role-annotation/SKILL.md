---
name: ferroptosis-gene-role-annotation
description: Use when a list of in-silico knockout essential genes includes ferroptosis-related
  candidates (e.g. SLC3A2, SLC7A11, SLC7A5) and their FerrDb roles and KEGG pathway
  enrichment must be determined.
when_to_use_negative:
- When the analysis targets direct viral proteome docking rather than host metabolic
  network effects — the virus is only modeled through its biomass function, so pathway-level
  annotation of host genes is not a substitute
- When no candidate gene list exists yet — this skill starts from SKO/DKO essential
  gene lists, not raw RNA-seq counts
- When studying cell types outside the modeled lung lines (A549, Calu-3, H1299, NHBE)
  without acknowledging that scope limitation
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: Enrichr
  role: Pathway enrichment of the SKO/DKO essential gene lists against the KEGG library,
    using Recon3D_01 subSystems as background
- name: FerrDb
  role: Source of ferroptosis gene classifications (drivers, suppressors, markers)
    used to cross-reference essential genes
- name: KEGG
  role: Pathway library against which enrichment of the essential genes is computed
- name: R
  role: Environment in which the Enrichr submissions, filtering, and results compilation
    are performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:d1f259f571e179c7d34c13a9e6ff259f83649f272d90f846667dc74f575d7a5f
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
  - FerrDb
  - KEGG
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ferroptosis-gene-role-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates context-specific essential genes with ferroptosis roles (drivers, suppressors, markers) by cross-referencing FerrDb classifications, and links them to KEGG/Enrichr pathway enrichments such as pyrimidine metabolism, oxidative stress, and ferroptosis identified in SARS-CoV-2-infected lung models.

## When to use

Use when a list of in-silico knockout essential genes includes ferroptosis-related candidates (e.g. SLC3A2, SLC7A11, SLC7A5) and their FerrDb roles and KEGG pathway enrichment must be determined.

## When NOT to use

- When the analysis targets direct viral proteome docking rather than host metabolic network effects — the virus is only modeled through its biomass function, so pathway-level annotation of host genes is not a substitute
- When no candidate gene list exists yet — this skill starts from SKO/DKO essential gene lists, not raw RNA-seq counts
- When studying cell types outside the modeled lung lines (A549, Calu-3, H1299, NHBE) without acknowledging that scope limitation

## Inputs

- Predicted essential gene lists from SKO and DKO in silico knockout of context-specific infected lung models (single-gene and gene-pair lists)
- FerrDb ferroptosis gene classification tables (drivers, suppressors, markers)
- Recon3D_01 subSystems list used as the enrichment pathway background

## Outputs

- Filtered KEGG enrichment table (terms with p-value/adjusted p-value < 0.05, single-gene enrichments removed)
- FerrDb cross-reference table assigning each essential gene a ferroptosis role (driver/suppressor/marker)
- Combined results table (CSV) of enriched pathways and annotated ferroptosis gene roles

## How to apply

Load the predicted essential gene lists from the SKO and DKO in silico knockout analyses of the context-specific infected lung models built with rFASTCORMICS on Recon3D_01 plus the viral biomass objective function. Submit each gene list to Enrichr (Kuleshov et al., 2016) against the KEGG pathway library, using the Recon3D_01 subSystems as the pathway background. Filter the returned enrichments to p-value/adjusted p-value < 0.05 and discard all enrichment results that are based on only one gene. Download the FerrDb ferroptosis gene classifications (drivers, suppressors, markers) and cross-reference the essential genes against them to assign roles (e.g. SLC7A11 and SLC3A2 as ferroptosis suppressors, SLC7A5 as a ferroptosis marker). Compile the surviving KEGG enrichment terms and the FerrDb cross-reference into a single results table (CSV).

## Related tools

- **Enrichr** (Pathway enrichment of the SKO/DKO essential gene lists against the KEGG library, using Recon3D_01 subSystems as background)
- **FerrDb** (Source of ferroptosis gene classifications (drivers, suppressors, markers) used to cross-reference essential genes)
- **KEGG** (Pathway library against which enrichment of the essential genes is computed)
- **R** (Environment in which the Enrichr submissions, filtering, and results compilation are performed)

## Examples

```
import gseapy as gp; enr = gp.enrichr(gene_list=['SLC3A2','SLC7A11','SLC7A5'], gene_sets='KEGG_2021_Human', background='Recon3D_01_subSystems', outdir='enr_ferroptosis')
```

## Evaluation signals

- Output table reproduces the reported enrichment of pyrimidine metabolism, oxidative stress, and ferroptosis, matching the article's key-pathway finding
- All retained enrichment terms meet the 0.05 significance cutoff (p-value/adjusted p-value < 0.05)
- No enrichment results based on only one gene remain in the final table
- FerrDb cross-reference assigns SLC3A2 and SLC7A11 as ferroptosis suppressors and SLC7A5 as a ferroptosis marker
- Ferroptosis is the only Enrichr-derived pathway associated with non-ACE2 vector samples and is attributable to SLC3A2 and SLC7A11

## Limitations

- Enrichment results based on only one gene are discarded, so single-gene-driven pathway hits are excluded by design
- The ferroptosis association is limited to non-ACE2 vector samples in the severity study (GSE147507)
- Findings are constrained by the modeled lung cell lines (A549, Calu-3, H1299, NHBE); further in vitro single- and double-gene deletion studies are needed to confirm essentiality

## Evidence

- [results] an Enrichr pathway analysis was performed (Kuleshov et al., 2016): [evidence span withheld — anchor status: not_found]
