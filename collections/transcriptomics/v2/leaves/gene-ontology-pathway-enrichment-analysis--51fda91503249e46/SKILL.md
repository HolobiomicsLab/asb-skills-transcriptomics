---
name: gene-ontology-pathway-enrichment-analysis
description: Use when a list of predicted mRNA/protein targets of small RNAs (e.g.
  tRF-5 and tRF-3 series tncRNAs of 18-25 nt) is available and you need to identify
  overrepresented biological processes such as metabolic and developmental pathways.
when_to_use_negative:
- Input is raw small RNA-seq reads or an unfiltered tncRNA pool rather than a curated
  target gene list — target prediction and filtering (e.g. selecting 18-25 nt common
  tncRNAs) must precede enrichment
- The organism lacks reference gene annotation/background sets compatible with g:Profiler
  (the paper's analysis used Arabidopsis against its annotated cDNA)
- The goal is tncRNA discovery or quantification itself (use the tncRNA Toolkit pipeline)
  rather than functional interpretation of known targets
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: psRNATarget
  role: Predicts putative mRNA targets of the tncRNA queries (2017 release, default
    parameters) against Arabidopsis cDNA and transposon sequences; its output supplies
    the protein target list for enrichment
- name: g:Profiler
  role: Performs pathway/GO enrichment analysis on the extracted protein target list
- name: Cytoscape
  role: Visualizes enriched terms using the EnrichmentMap and AutoAnnotate apps to
    cluster and annotate enriched biological processes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:4ea80cd0eeefea69e8b6d82b4d71ef8e47b3021add67116ec90e95be61601f69
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Cytoscape
  - g:Profiler
  - psRNATarget
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-ontology-pathway-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functional interpretation of predicted tncRNA (tRF-5/tRF-3) mRNA target lists by submitting the extracted protein targets to g:Profiler for GO/pathway enrichment, then visualizing enriched terms in Cytoscape. Used to test whether conserved plant tncRNA targets are overrepresented in metabolism- and development-related biological processes.

## When to use

Use when a list of predicted mRNA/protein targets of small RNAs (e.g. tRF-5 and tRF-3 series tncRNAs of 18-25 nt) is available and you need to identify overrepresented biological processes such as metabolic and developmental pathways.

## When NOT to use

- Input is raw small RNA-seq reads or an unfiltered tncRNA pool rather than a curated target gene list — target prediction and filtering (e.g. selecting 18-25 nt common tncRNAs) must precede enrichment
- The organism lacks reference gene annotation/background sets compatible with g:Profiler (the paper's analysis used Arabidopsis against its annotated cDNA)
- The goal is tncRNA discovery or quantification itself (use the tncRNA Toolkit pipeline) rather than functional interpretation of known targets

## Inputs

- Common tncRNA query sequences (tRF-5 and tRF-3 series; 18-25 nt long) from the identified tncRNA pool
- Arabidopsis thaliana cDNA and transposon sequences (psRNATarget target database, 2017 release)
- psRNATarget output table of predicted targets, from which protein target IDs are extracted
- Protein target list submitted to g:Profiler

## Outputs

- psRNATarget predicted target list for the tncRNA queries
- g:Profiler pathway/GO enrichment results (enriched biological process terms for the tncRNA targets)
- Cytoscape EnrichmentMap/AutoAnnotate network of clustered enriched terms (as reported in Supplementary Fig. 13)

## How to apply

First predict putative mRNA targets by running psRNATarget (2017 release, default parameters) with common tncRNAs (tRF-5 and tRF-3 series; 18-25 nt long) as query sequences against Arabidopsis cDNA and transposon sequences. Extract the predicted protein targets from the psRNATarget output table. Submit this protein target list to g:Profiler for pathway/GO enrichment analysis (gene set enrichment of the target list against the genome background). Finally, visualize the enriched terms in Cytoscape using the EnrichmentMap and AutoAnnotate apps to cluster related terms into interpretable biological themes. Rationale: the enrichment step converts an unstructured target list into testable functional claims, e.g. that common tncRNA targets are majorly enriched in metabolic and developmental processes.

## Related tools

- **psRNATarget** (Predicts putative mRNA targets of the tncRNA queries (2017 release, default parameters) against Arabidopsis cDNA and transposon sequences; its output supplies the protein target list for enrichment)
- **g:Profiler** (Performs pathway/GO enrichment analysis on the extracted protein target list)
- **Cytoscape** (Visualizes enriched terms using the EnrichmentMap and AutoAnnotate apps to cluster and annotate enriched biological processes)

## Evaluation signals

- Enrichment results should show that common tncRNA targets are majorly enriched in metabolic and developmental processes, consistent with the paper's reported finding (Supplementary Fig. 13)
- Specifically, targets of conserved tRF-5 sequences should be enriched in growth and development, transport and localization, and cellular metabolic processes
- The query set submitted to g:Profiler should consist only of protein targets extracted from the psRNATarget output for tRF-5/tRF-3 queries of 18-25 nt
- Cytoscape EnrichmentMap output should cluster enriched terms into coherent themes (metabolism, development) after AutoAnnotate labeling

## Limitations

- Targets are computationally predicted (psRNATarget default parameters, 2017 release) and not experimentally validated; the paper notes that 'Further validation of DE tncRNAs and their putative targets will pave the way for a better understanding of the underlying mechanism'
- Enrichment relies on g:Profiler's reference background and GO annotation coverage for the chosen organism (Arabidopsis); results for other angiosperms may differ
- Only common tncRNAs of the tRF-5 and tRF-3 series (18-25 nt) were used as queries; other tncRNA subtypes (tRF-1, tRNA halves) were not part of this target-enrichment workflow

## Evidence

- [methods] Common tncRNAs used as enrichment query: [evidence span withheld — anchor status: not_found]
- [other] g:Profiler step in the reproducible workflow: [evidence span withheld — anchor status: not_found]
- [other] psRNATarget prediction step and validation criterion: [evidence span withheld — anchor status: not_found]
