---
name: clinvar-pfam-annotation-mapping
description: Use when RNA-seq isoform usage comparison yields isoform switching/dMDT
  events and each event must be annotated with ClinVar disease-gene status and PFAM
  domain IDs to score functional impact on protein interactions.
when_to_use_negative:
- The analysis goal is differential gene expression rather than isoform usage switching
  — ClinVar/PFAM interaction-disruption mapping requires per-isoform usage events
  (dMDTs), not gene-level abundance changes.
- The organism or protein set is not covered by STRING/ClinVar/PFAM annotations —
  domain IDs and disease-gene highlighting cannot be produced.
- Input is already a finished CanIsoNet-schema results table awaiting upload — annotation
  mapping is redundant; go directly to database submission.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0128
tools:
- name: CanIsoNet
  role: Database and pipeline that merges isoform switching events with functional
    impact level, ClinVar gene highlighting, and PFAM domain IDs of disrupted interactions;
    hosts the annotated results across 31 disease types
  repo: github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Protein interaction network on which isoform-specific interaction disruptions
    are shown and ClinVar-annotated genes are highlighted
- name: ClinVar
  role: Source of disease-gene annotations used to predict the pathogenic impact of
    isoform switching events
- name: PFAM
  role: Source of domain IDs reported for disrupted interactions caused by isoform
    switching
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:2dfd5949b01a279e302d8852b6de51820708a141be331518afe6f694e7808d36
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CanIsoNet
  - ClinVar
  - PFAM
  - STRING
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# clinvar-pfam-annotation-mapping

## Summary

Annotates isoform switching events from disease RNA-seq analyses with ClinVar gene annotations and PFAM domain IDs of disrupted interactions, so the pathogenic impact of each dMDT event can be assessed and visualized on the STRING interaction network as implemented in CanIsoNet.

## When to use

Use when RNA-seq isoform usage comparison yields isoform switching/dMDT events and each event must be annotated with ClinVar disease-gene status and PFAM domain IDs to score functional impact on protein interactions.

## When NOT to use

- The analysis goal is differential gene expression rather than isoform usage switching — ClinVar/PFAM interaction-disruption mapping requires per-isoform usage events (dMDTs), not gene-level abundance changes.
- The organism or protein set is not covered by STRING/ClinVar/PFAM annotations — domain IDs and disease-gene highlighting cannot be produced.
- Input is already a finished CanIsoNet-schema results table awaiting upload — annotation mapping is redundant; go directly to database submission.

## Inputs

- Isoform switching / dMDT event table (gene, participating isoforms, switching statistics, per-sample isoform expression values)
- Kallisto TPM transcript-level quantifications per sample
- STRING isoform-specific interaction network
- ClinVar gene annotations
- PFAM domain ID annotations

## Outputs

- Per-event annotation table with ClinVar-annotated gene flags and PFAM domain IDs of disrupted interactions
- Interactive STRING interaction network with ClinVar-annotated genes highlighted and disrupted interactions visualized
- Disease-type schema-conformant results suitable for upload to the CanIsoNet database

## How to apply

Start from per-event isoform switching results (gene, participating isoforms, isoform usage statistics, per-sample TPM expression values from Kallisto). For each event, map the affected gene to ClinVar annotations to flag whether the gene carries known pathogenic variants, and retrieve the PFAM domain IDs for the protein regions involved so disrupted interactions can be localized to specific domains. Overlay the events on the STRING isoform-specific interaction network to compute the functional impact level of losing the switched isoform's interactions, then highlight ClinVar-annotated genes in the interactive network view. Package the annotations for each event so that the output matches the schema used by the 31 disease types in the CanIsoNet database (caniso.net), allowing the results to be browsed, searched, and visualized alongside existing dMDT events.

## Related tools

- **CanIsoNet** (Database and pipeline that merges isoform switching events with functional impact level, ClinVar gene highlighting, and PFAM domain IDs of disrupted interactions; hosts the annotated results across 31 disease types) — github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Protein interaction network on which isoform-specific interaction disruptions are shown and ClinVar-annotated genes are highlighted)
- **ClinVar** (Source of disease-gene annotations used to predict the pathogenic impact of isoform switching events)
- **PFAM** (Source of domain IDs reported for disrupted interactions caused by isoform switching)

## Evaluation signals

- Every output event record carries the required fields: gene, participating isoforms, switching statistics, interaction disruption/impact scores, ClinVar gene annotation flag, and PFAM domain IDs.
- PFAM domain IDs are present for each disrupted interaction, as the database reports 'PFAM domain IDs of disrupted interactions'.
- ClinVar-annotated genes are visibly highlighted in the rendered interactive STRING interaction network for each disease type.
- The results table conforms to the CanIsoNet database schema used by the 31 hosted disease types (11,811 isoforms / 14,357 unique events reference scale), so the new disease can be added alongside them.
- Pipeline code provenance is traceable to github.com/kahramanlab/CanIsoNet_Web under its Creative Commons license.

## Limitations

- Functional impact is inferred by merging isoform expression with STRING networks and ClinVar annotations; it is a prediction of pathogenic impact, not experimental validation.
- Coverage is anchored to isoforms already analyzed by the pipeline (11,811 isoforms; README variant reports 11,528), so isoform counts differ between the article and README and new events outside this frame may need re-annotation.
- Adding a new disease dataset requires contacting the authors — 'Upon request, new datasets can be analysed and uploaded to the CanIsoNet database' — so upload is not fully self-service.

## Evidence

- [abstract] CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
- [abstract] the genes annotated in ClinVar are highlighted in interactive interaction networks: "the genes annotated in ClinVar are highlighted in interactive interaction networks"
- [readme] It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases.: "It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases."
- [abstract] The source codes can be found under a Creative Common License at https://github.com/kahramanlab/CanIsoNet_Web.: "The source codes can be found under a Creative Common License at https://github.com/kahramanlab/CanIsoNet_Web."
