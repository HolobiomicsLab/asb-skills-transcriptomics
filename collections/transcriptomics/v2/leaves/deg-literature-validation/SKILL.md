---
name: deg-literature-validation
description: Use when a ranked DEG list (e.g., scTITANS output at fdr level 0.01)
  must be confirmed as biologically meaningful, via literature surveys plus GO BP
  enrichment of the top genes.
when_to_use_negative:
- The input is raw gene expression data rather than a finalized DEG list — run the
  upstream differential expression step (e.g., scTITANS_full/scTITANS_partial) first.
- Cell-level annotation or trajectory analysis is the goal; this skill only validates
  a gene list, it does not identify genes or subclusters.
- Enrichment is required for non-GO categories (e.g., pathways, networks) or a different
  ontology than GO biological process, which the task's ToppGene configuration restricts
  against.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: scTITANS
  role: Produces the top DEG list (fdr level 0.01) that is validated by this skill
  repo: https://github.com/ZJUFanLab/scTITANS
- name: ToppGene
  role: Web service used to compute GO biological-process enrichment of the submitted
    DEG list for the specified organism
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:48bbaf770c2b33a3393a010e22751851d1d4d96be8b95ebe0fc5970f6c91e954
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
  - ToppGene
  - scTITANS
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# deg-literature-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that top differential genes (e.g., scTITANS DEGs at fdr 0.01) are biologically meaningful by combining literature verification with Gene Ontology biological-process enrichment analysis (ToppGene), compiling the results into a structured enrichment table.

## When to use

Use when a ranked DEG list (e.g., scTITANS output at fdr level 0.01) must be confirmed as biologically meaningful, via literature surveys plus GO BP enrichment of the top genes.

## When NOT to use

- The input is raw gene expression data rather than a finalized DEG list — run the upstream differential expression step (e.g., scTITANS_full/scTITANS_partial) first.
- Cell-level annotation or trajectory analysis is the goal; this skill only validates a gene list, it does not identify genes or subclusters.
- Enrichment is required for non-GO categories (e.g., pathways, networks) or a different ontology than GO biological process, which the task's ToppGene configuration restricts against.

## Inputs

- Top DEG list from scTITANS (SigGenes.fdr0.01.txt) at the paper's fdr level of 0.01
- DEG gene list (e.g., top 20 DEGs) formatted for submission to ToppGene Suite
- Organism identifier matching the DEG source dataset (e.g., Caenorhabditis elegans for CEED/GSE126954)

## Outputs

- Structured GO BP enrichment table (toppgene_go_enrichment.csv) keyed by GO term, with term IDs, term names, p-values, and FDR-adjusted significance
- Per-gene literature verification record (e.g., 11 of top 12 CEED DEGs confirmed)

## How to apply

Load the top DEG list produced by the upstream differential expression method (e.g., scTITANS at fdr level 0.01, e.g. on the CEED/GSE126954 C. elegans dataset). Submit the DEG gene list (e.g., top 20 genes) to the ToppGene Suite web service, selecting the correct organism (Caenorhabditis elegans for CEED) and restricting enrichment to Gene Ontology biological-process terms. Retrieve ranked GO BP results including term IDs, term names, p-values, and FDR-adjusted significance, then compile them into a keyed table (toppgene_go_enrichment.csv). In parallel, survey the literature for each top DEG to confirm reported functions; in the source task 11 of the top 12 CEED DEGs were literature-confirmed and enriched terms (e.g., ascending aorta morphogenesis) reflected typical developmental processes. Judge success by the enrichment terms' agreement with literature-reported gene functions and by the output table's validity (parses, contains GO BP terms with p-value and FDR at fdr 0.01).

## Related tools

- **scTITANS** (Produces the top DEG list (fdr level 0.01) that is validated by this skill) — https://github.com/ZJUFanLab/scTITANS
- **ToppGene** (Web service used to compute GO biological-process enrichment of the submitted DEG list for the specified organism)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01); # then submit SigGenes.fdr0.01.txt top genes to ToppGene (organism=C. elegans, category=GO Biological Process) -> toppgene_go_enrichment.csv
```

## Evaluation signals

- toppgene_go_enrichment.csv exists and parses as a valid table keyed by GO term
- The table contains GO BP term annotations plus significance statistics (p-value and FDR) for the submitted DEG list at fdr 0.01
- A majority of the top DEGs (e.g., 11 of the top 12 CEED DEGs) are confirmed by literature survey
- Enriched GO BP terms are biologically coherent with the literature-reported functions of the top DEGs (e.g., developmental processes such as ascending aorta morphogenesis)
- The organism used for enrichment matches the DEG source dataset (e.g., Caenorhabditis elegans for GSE126954)

## Limitations

- Validation quality depends on literature coverage — genes lacking prior publications cannot be confirmed by survey (1 of the top 12 CEED DEGs went unconfirmed).
- Enrichment analysis requires choosing the correct organism in ToppGene; a mismatched organism yields meaningless GO terms.
- Only GO biological-process terms were used in the source task; other ontologies/categories were out of scope.
- Missing signals in the source article include software version numbers for tools (e.g., Monocle3, scran, scTITANS), which limits exact reproducibility.

## Evidence

- [results] Eleven of the top 12 differential genes identified by scTITANS in the CEED dataset were successfully confirmed via literature surveys: [evidence span withheld — anchor status: not_found]
- [methods] Submit the DEG gene list to the ToppGene Suite web service, selecting Caenorhabditis elegans as the organism and restricting the enrichment analysis to Gene Ontology biological-process terms.: [evidence span withheld — anchor status: not_found]
- [methods] the output table exists, parses as a valid table, and contains GO BP term annotations plus significance statistics (p-value and FDR) for the submitted DEG list at the paper's fdr level of 0.01: [evidence span withheld — anchor status: not_found]
- [methods] Load the top DEG list produced by scTITANS (fdr level 0.01) on the CEED dataset (GSE126954).: [evidence span withheld — anchor status: not_found]
