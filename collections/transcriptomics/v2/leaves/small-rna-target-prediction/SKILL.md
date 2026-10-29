---
name: small-rna-target-prediction
description: Use when you have a set of conserved plant small RNAs (e.g. tRF-5/tRF-3
  series 18–25 nt) and need to predict their putative mRNA targets and test whether
  targets are enriched in specific biological processes.
when_to_use_negative:
- Input small RNAs are from non-plant systems or fall outside the tRF-5/tRF-3 series
  18–25 nt window (e.g. tRF-1s or tRNA halves)
- Goal is de novo tncRNA discovery from sRNA-seq reads rather than target prediction
  of an already-identified tncRNA pool
- No suitable target reference (cDNA/transposon sequences) is available for the species
  of interest
edam_operation: http://edamontology.org/operation_0464
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: psRNATarget
  role: Web tool (2017 release, default parameters) used to predict putative mRNA
    targets of the 18–25 nt tRF-5/tRF-3 query sequences against Arabidopsis cDNA and
    transposon sequences
- name: g:Profiler
  role: Performs pathway/go-term enrichment analysis on the list of predicted protein
    targets extracted from psRNATarget output
- name: Cytoscape
  role: Visualization of enriched terms using the EnrichmentMap and AutoAnnotate apps
- name: tncRNA Toolkit
  role: Upstream pipeline that generates the identified tncRNA pool from which the
    common tRF-5/tRF-3 query sequences are selected
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:6653a10349289bb4c558c52fd3667fd32515c58b25485749183302748c77f581
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Cytoscape
  - g:Profiler
  - psRNATarget
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0464
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# small-rna-target-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict plant mRNA targets of conserved tRNA-derived non-coding RNAs (tRF-5 and tRF-3 series, 18–25 nt) using psRNATarget, then infer their collective function via g:Profiler pathway enrichment and Cytoscape EnrichmentMap visualization.

## When to use

Use when you have a set of conserved plant small RNAs (e.g. tRF-5/tRF-3 series 18–25 nt) and need to predict their putative mRNA targets and test whether targets are enriched in specific biological processes.

## When NOT to use

- Input small RNAs are from non-plant systems or fall outside the tRF-5/tRF-3 series 18–25 nt window (e.g. tRF-1s or tRNA halves)
- Goal is de novo tncRNA discovery from sRNA-seq reads rather than target prediction of an already-identified tncRNA pool
- No suitable target reference (cDNA/transposon sequences) is available for the species of interest

## Inputs

- List of common tncRNA query sequences (tRF-5 and tRF-3 series, 18–25 nt long)
- Target species reference sequences (e.g. Arabidopsis cDNA and transposon sequences)
- Predicted protein target gene list (for enrichment analysis)

## Outputs

- psRNATarget target-prediction output table mapping each tncRNA to putative protein-coding targets
- g:Profiler pathway/go-term enrichment results for the target list
- Cytoscape EnrichmentMap network with AutoAnnotate labels of enriched biological processes

## How to apply

Select common tncRNAs from the identified tncRNA pool, restricting to the tRF-5 and tRF-3 series that are 18–25 nt long. Run psRNATarget (2017 release, default parameters) with these small-RNA sequences as query against Arabidopsis cDNA and transposon sequences to predict putative targets. Extract the predicted protein targets from the psRNATarget output table and submit the target list to g:Profiler for pathway enrichment analysis. Visualize the enriched terms in Cytoscape using the EnrichmentMap and AutoAnnotate apps. The rationale is that target prediction followed by enrichment reveals whether common tncRNA targets are majorly enriched in metabolic and developmental processes, as reported in the source study (Supplementary Fig. 13).

## Related tools

- **psRNATarget** (Web tool (2017 release, default parameters) used to predict putative mRNA targets of the 18–25 nt tRF-5/tRF-3 query sequences against Arabidopsis cDNA and transposon sequences)
- **g:Profiler** (Performs pathway/go-term enrichment analysis on the list of predicted protein targets extracted from psRNATarget output)
- **Cytoscape** (Visualization of enriched terms using the EnrichmentMap and AutoAnnotate apps)
- **tncRNA Toolkit** (Upstream pipeline that generates the identified tncRNA pool from which the common tRF-5/tRF-3 query sequences are selected) — https://github.com/skbinfo/tncRNA-Toolkit

## Evaluation signals

- Enrichment results should show that common tncRNA targets are majorly enriched in metabolic and developmental processes (growth/development, transport and localization, cellular metabolic process), consistent with Supplementary Fig. 13
- Query set contains only tRF-5 and tRF-3 series sequences in the 18–25 nt range
- psRNATarget output yields an extractable list of protein-coding targets that is accepted by g:Profiler
- EnrichmentMap network in Cytoscape renders annotated clusters of enriched terms rather than an empty or single-node graph

## Limitations

- Predicted tncRNA targets require further experimental validation; the study notes that 'Further validation of DE tncRNAs and their putative targets will pave the way for a better understanding of the underlying mechanism in regulating stress responses'
- Findings are grounded in six angiosperms and Arabidopsis target references; transferability to other species is untested
- psRNATarget was run with the 2017 release and default parameters, so newer scoring schemes or non-default settings are not covered

## Evidence
