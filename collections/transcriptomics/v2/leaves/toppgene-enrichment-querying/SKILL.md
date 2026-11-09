---
name: toppgene-enrichment-querying
description: Use when a ranked differential-expression gene list (e.g., scTITANS SigGenes
  at fdr 0.01) needs functional interpretation via GO biological-process enrichment,
  especially for non-model organisms like Caenorhabditis elegans.
when_to_use_negative:
- The input is not a gene list from differential expression analysis (e.g., raw expression
  matrices or cell metadata files) — enrichment requires a defined DEG set
- The organism of the DEG list is unknown or unsupported, since the ToppGene query
  requires an explicit organism selection (e.g., C. elegans for CEED)
- The upstream scTITANS run has not been performed at the paper's fdr level (0.01),
  since the skill validates DEGs at that significance threshold
edam_operation: http://edamontology.org/operation_3666
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ToppGene
  role: Web service used to perform GO biological-process enrichment analysis on the
    submitted DEG gene list, returning ranked terms with p-values and FDR
- name: scTITANS
  role: Upstream producer of the differential gene list (fdr level 0.01) that is submitted
    to ToppGene for enrichment
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Constructs the trajectory and pseudotime ordering that underlies scTITANS
    DEG identification prior to enrichment querying
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:0de144865600ee1cf0d49413901552150a6b97e8c8dd3580aac1cc073f29f068
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  repo_url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Monocle3
  - ToppGene
  - scTITANS
  edam_operation: http://edamontology.org/operation_3666
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# toppgene-enrichment-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Submit a list of differential genes (e.g., top scTITANS DEGs at fdr 0.01 from the CEED/GSE126954 C. elegans dataset) to the ToppGene Suite web service to retrieve Gene Ontology biological-process enrichment with p-values and FDR-adjusted significance, then compile the results into a structured table for biological validation of the DEG list.

## When to use

Use when a ranked differential-expression gene list (e.g., scTITANS SigGenes at fdr 0.01) needs functional interpretation via GO biological-process enrichment, especially for non-model organisms like Caenorhabditis elegans.

## When NOT to use

- The input is not a gene list from differential expression analysis (e.g., raw expression matrices or cell metadata files) — enrichment requires a defined DEG set
- The organism of the DEG list is unknown or unsupported, since the ToppGene query requires an explicit organism selection (e.g., C. elegans for CEED)
- The upstream scTITANS run has not been performed at the paper's fdr level (0.01), since the skill validates DEGs at that significance threshold

## Inputs

- Ranked list of top DEGs (e.g., scTITANS SigGenes.fdr0.01.txt) from the CEED (GSE126954) dataset
- Target organism specification (Caenorhabditis elegans)
- Gene Ontology biological-process term database (accessed via ToppGene Suite)

## Outputs

- Ranked GO biological-process enrichment table (toppgene_go_enrichment.csv) with GO term IDs, term names, p-values, and FDR-adjusted significance
- Cross-check of enriched GO terms against literature-reported functions of the top DEGs

## How to apply

Take the top DEG list produced upstream by scTITANS (fdr level 0.01) — e.g., genes written to SigGenes.fdr0.01.txt after Monocle3 pseudotime ordering on the CEED (GSE126954) dataset. Submit the gene list to the ToppGene Suite web service, selecting Caenorhabditis elegans as the organism (matching the dataset's species) and restricting the analysis to Gene Ontology biological-process terms. Retrieve the ranked enrichment results including GO term IDs, term names, p-values, and FDR-adjusted significance for each term. Compile the results into a structured table (toppgene_go_enrichment.csv) keyed by GO term, then compare the enriched terms against literature-reported functions of the top DEGs to confirm biological meaningfulness — in the source task, 11 of the top 12 DEGs were literature-confirmed and the enriched processes (e.g., ascending aorta morphogenesis) were typical developmental processes.

## Related tools

- **ToppGene** (Web service used to perform GO biological-process enrichment analysis on the submitted DEG gene list, returning ranked terms with p-values and FDR)
- **scTITANS** (Upstream producer of the differential gene list (fdr level 0.01) that is submitted to ToppGene for enrichment) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Constructs the trajectory and pseudotime ordering that underlies scTITANS DEG identification prior to enrichment querying)

## Examples

```
enrichment <- read.csv("toppgene_go_enrichment.csv"); top_terms <- enrichment[order(enrichment$p_value), ]
```

## Evaluation signals

- The output file toppgene_go_enrichment.csv exists and parses as a valid structured table keyed by GO term
- The table contains GO BP term annotations plus significance statistics (p-value and FDR) for the submitted DEG list at the paper's fdr level of 0.01
- The enriched GO terms correspond to the organism selected in the query (C. elegans for CEED/GSE126954)
- Enriched terms are consistent with literature-reported developmental functions of the top DEGs (in the source task, 11 of 12 top DEGs were literature-confirmed)
- The submitted gene list matches the upstream scTITANS output (SigGenes.fdr0.01.txt / top 20 DEGs used in the workflow)

## Limitations

- ToppGene enrichment is performed via a web service with manual submission, so the step is not fully scripted and software version numbers for the tools (e.g., Monocle3, scTITANS, ToppGene) are not stated in the source article
- Validation of enriched terms is limited to comparison against literature-reported functions; terms without prior literature support cannot be confirmed this way
- Enrichment quality depends on the upstream scTITANS fdr threshold (0.01); lists derived at other thresholds were not validated in the source task

## Evidence

- [other] Submit the DEG gene list to the ToppGene Suite web service, selecting Caenorhabditis elegans as the organism and restricting the enrichment analysis to Gene Ontology biological-process terms.: [evidence span withheld — anchor status: not_found]
- [other] the enriched GO biological processes for the top 20 DEGs (e.g., ascending aorta morphogenesis) are typical developmental processes: [evidence span withheld — anchor status: not_found]
