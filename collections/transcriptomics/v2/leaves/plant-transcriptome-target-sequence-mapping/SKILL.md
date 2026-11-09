---
name: plant-transcriptome-target-sequence-mapping
description: Use when you have 18–25 nt plant small-RNA sequences (e.g. conserved
  tRF-5/tRF-3 queries) and need predicted mRNA targets plus functional enrichment
  of those targets in metabolic/developmental processes.
when_to_use_negative:
- Queries are not the common tncRNA classes (e.g. tRNA halves, tRF-1s, or sequences
  outside 18–25 nt) — the workflow was validated specifically on tRF-5/tRF-3 series
  18–25 nt long
- Reference transcriptome is non-plant or human — the article notes existing tncRNA
  tools (tDRmapper, MINTmap, tRF2Cancer) are trained/tested on human datasets, and
  this pipeline is validated against Arabidopsis (TAIR10.1)
- A curated, experimentally validated target list already exists — this skill only
  produces computationally predicted targets
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0659
tools:
- name: psRNATarget
  role: Predicts putative tncRNA targets by aligning 18–25 nt tRF-5/tRF-3 queries
    against Arabidopsis cDNA and transposon sequences (2017 release, default parameters)
- name: g:Profiler
  role: Performs pathway enrichment analysis on the protein target list extracted
    from psRNATarget output
- name: Cytoscape
  role: Visualizes enriched terms via the EnrichmentMap and AutoAnnotate apps
- name: tncRNA Toolkit
  role: Upstream workflow that identifies and quantifies the tRF-5/tRF-3 query sequences
    from processed small RNA reads
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:d5c053278f51a604f668821952b97d894a00d4678f4366cd51436d46f819b0b5
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# plant-transcriptome-target-sequence-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps short plant tncRNA queries (tRF-5/tRF-3 series, 18–25 nt) against an Arabidopsis cDNA and transposon reference using psRNATarget to predict putative mRNA targets, then annotates the target list with g:Profiler and visualizes enriched terms in Cytoscape.

## When to use

Use when you have 18–25 nt plant small-RNA sequences (e.g. conserved tRF-5/tRF-3 queries) and need predicted mRNA targets plus functional enrichment of those targets in metabolic/developmental processes.

## When NOT to use

- Queries are not the common tncRNA classes (e.g. tRNA halves, tRF-1s, or sequences outside 18–25 nt) — the workflow was validated specifically on tRF-5/tRF-3 series 18–25 nt long
- Reference transcriptome is non-plant or human — the article notes existing tncRNA tools (tDRmapper, MINTmap, tRF2Cancer) are trained/tested on human datasets, and this pipeline is validated against Arabidopsis (TAIR10.1)
- A curated, experimentally validated target list already exists — this skill only produces computationally predicted targets

## Inputs

- FASTA of common tncRNA query sequences (tRF-5 and tRF-3 series, 18–25 nt)
- Arabidopsis cDNA library and transposon sequences (TAIR10.1 assembly) as psRNATarget reference
- Predicted protein target list (gene IDs) for g:Profiler input

## Outputs

- psRNATarget output table of predicted tncRNA–mRNA target pairs
- Extracted protein target list
- g:Profiler pathway/GO enrichment results for the target list
- Cytoscape EnrichmentMap network with AutoAnnotate labels of enriched terms

## How to apply

Select the common tncRNAs to query — in the source study, the tRF-5 and tRF-3 series, 18–25 nt long, drawn from the identified tncRNA pool. Run psRNATarget (2017 release, default parameters) with these tncRNA queries against Arabidopsis cDNA and transposon sequences (genome assembly TAIR10.1) to predict putative targets; the decision point is to keep the predicted protein targets from the psRNATarget output as the working target list. Submit that protein list to g:Profiler for pathway enrichment analysis. Visualize the enriched terms in Cytoscape with the EnrichmentMap and AutoAnnotate apps. Correct application is judged by whether the enrichment recapitulates the paper's finding that common tncRNA targets are majorly enriched in metabolic and developmental processes (Supplementary Fig. 13).

## Related tools

- **psRNATarget** (Predicts putative tncRNA targets by aligning 18–25 nt tRF-5/tRF-3 queries against Arabidopsis cDNA and transposon sequences (2017 release, default parameters))
- **g:Profiler** (Performs pathway enrichment analysis on the protein target list extracted from psRNATarget output)
- **Cytoscape** (Visualizes enriched terms via the EnrichmentMap and AutoAnnotate apps)
- **tncRNA Toolkit** (Upstream workflow that identifies and quantifies the tRF-5/tRF-3 query sequences from processed small RNA reads) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -s arabidopsis -i processed_sRNA.fastq -o out/  # generate tRF queries, then submit to psRNATarget (2017 release) vs TAIR10.1 cDNA
```

## Evaluation signals

- Enrichment results should show common tncRNA targets majorly enriched in metabolic and developmental processes, matching the paper's reported finding (Supplementary Fig. 13)
- Predicted target proteins should be extractable as a clean gene/protein ID list from the psRNATarget output suitable for direct g:Profiler submission
- Query sequences should be restricted to 18–25 nt tRF-5/tRF-3 members of the identified tncRNA pool, as in the source workflow
- EnrichmentMap network in Cytoscape should display clustered, annotated enriched terms rather than isolated singleton nodes

## Limitations

- Predicted targets are computational only — the article notes further validation of putative targets is needed to understand underlying regulatory mechanisms
- No single standardized nomenclature exists for tncRNA subtypes, complicating cross-study comparison of query definitions
- Mismatch-tolerant mapping can misidentify bases and inflate false positives/negatives in the upstream tncRNA calls feeding this workflow

## Evidence
