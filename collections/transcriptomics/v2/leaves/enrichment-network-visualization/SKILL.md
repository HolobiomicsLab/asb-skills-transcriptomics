---
name: enrichment-network-visualization
description: Use when you have a list of predicted target proteins and enrichment
  analysis results (GO/pathway terms) that need to be organized into a visual similarity
  network with cluster annotations, as done for common plant tRF-5/tRF-3 targets.
when_to_use_negative:
- You have only raw small RNA-seq reads or an unfiltered tncRNA pool — target prediction
  (psRNATarget) and enrichment (g:Profiler) must be completed first
- The input is a single enrichment result with only a handful of terms, where a simple
  bar/table view suffices and network clustering adds no interpretive value
- The target list is not protein-coding gene identifiers compatible with the enrichment
  database (e.g., transposon-only targets with no GO annotation)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: g:Profiler
  role: Produces the pathway enrichment results (enriched GO/pathway terms) for the
    tncRNA target protein list that are imported into Cytoscape
- name: Cytoscape (EnrichmentMap app)
  role: Builds the term-similarity network visualization of enriched terms
- name: Cytoscape (AutoAnnotate app)
  role: Labels clusters of enriched terms in the EnrichmentMap network with representative
    annotations
- name: psRNATarget
  role: Upstream step supplying the predicted mRNA/protein target list of common tncRNAs
    used as enrichment input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:cd820c7c7aa96e010522d137da0f507b4d798338d1dd0ef666b486ebd2e9d065
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
  - Cytoscape (AutoAnnotate app)
  - Cytoscape (EnrichmentMap app)
  - g:Profiler
  - psRNATarget
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# enrichment-network-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualizes functional enrichment results (e.g., g:Profiler output on psRNATarget-predicted tncRNA target proteins) as an annotated network in Cytoscape using the EnrichmentMap and AutoAnnotate apps, so enriched biological themes among target genes become interpretable clusters.

## When to use

Use when you have a list of predicted target proteins and enrichment analysis results (GO/pathway terms) that need to be organized into a visual similarity network with cluster annotations, as done for common plant tRF-5/tRF-3 targets.

## When NOT to use

- You have only raw small RNA-seq reads or an unfiltered tncRNA pool — target prediction (psRNATarget) and enrichment (g:Profiler) must be completed first
- The input is a single enrichment result with only a handful of terms, where a simple bar/table view suffices and network clustering adds no interpretive value
- The target list is not protein-coding gene identifiers compatible with the enrichment database (e.g., transposon-only targets with no GO annotation)

## Inputs

- g:Profiler pathway enrichment output for a protein target list (e.g., from psRNATarget predictions of common tncRNAs against Arabidopsis cDNA)
- Predicted protein target list extracted from psRNATarget output

## Outputs

- Cytoscape EnrichmentMap network of enriched terms with cluster annotations (AutoAnnotate labels)
- Figure summarizing enriched biological processes among common tncRNA targets (analogous to Supplementary Fig. 13)

## How to apply

First obtain the protein target list extracted from psRNATarget (2017 release, default parameters) predictions of common tncRNAs (tRF-5 and tRF-3 series; 18–25 nt long) against Arabidopsis cDNA and transposon sequences, and submit it to g:Profiler for pathway enrichment analysis. Then import the enriched terms into Cytoscape and build a term-similarity network with the EnrichmentMap app, where nodes represent enriched terms and edges reflect shared genes between terms. Apply the AutoAnnotate app to label the resulting clusters with representative terms so themes like metabolic and developmental processes are legible. The rationale is that a network view collapses redundant GO terms into interpretable functional modules, which is how the paper summarized common tncRNA target enrichment (Supplementary Fig. 13).

## Related tools

- **g:Profiler** (Produces the pathway enrichment results (enriched GO/pathway terms) for the tncRNA target protein list that are imported into Cytoscape)
- **Cytoscape (EnrichmentMap app)** (Builds the term-similarity network visualization of enriched terms)
- **Cytoscape (AutoAnnotate app)** (Labels clusters of enriched terms in the EnrichmentMap network with representative annotations)
- **psRNATarget** (Upstream step supplying the predicted mRNA/protein target list of common tncRNAs used as enrichment input)

## Evaluation signals

- The EnrichmentMap network shows enriched terms grouped into clusters corresponding to metabolic and developmental processes, consistent with the paper's finding for common tncRNA targets
- Each node in the network corresponds to an enriched term from the g:Profiler output for the psRNATarget-derived target list, with edges reflecting shared genes
- AutoAnnotate labels make each major cluster's functional theme readable (e.g., growth and development, transport and localization, cellular metabolic processes)
- The rendered network reproduces the qualitative content reported in Supplementary Fig. 13 when run on the paper's common tRF-5/tRF-3 target list

## Limitations

- Enrichment reflects predictions from psRNATarget rather than experimentally validated targets; the article notes that DE tncRNAs and their putative targets require further validation
- Visualization depends on upstream tool versions and parameters (psRNATarget 2017 release, default parameters; 18–25 nt tRF-5/tRF-3 queries), so differing inputs will change the network
- No standardized nomenclature exists for tncRNA subtypes, so which sequences count as 'common tncRNAs' for the target query may vary between practitioners

## Evidence

- [supplementary] Supplementary Fig. 13 enrichment finding: [evidence span withheld — anchor status: not_found]
- [other] Cytoscape visualization workflow step: [evidence span withheld — anchor status: not_found]
