---
name: gene-ontology-biological-process-enrichment
description: Use when you have a ranked or significant gene list (e.g., SigGenes at
  fdr level 0.01) and need to confirm it is biologically meaningful via GO biological-process
  term enrichment.
when_to_use_negative:
- The input is not a gene list but raw scRNA-seq data — run scTITANS first to identify
  the DEGs before enrichment.
- The analysis goal is cell-type annotation rather than functional interpretation
  of DEGs — use marker-gene-based annotation (e.g., scCATCH) instead.
- Cell ontology or pathway (non-GO) enrichment is required and GO biological-process
  terms are not an acceptable annotation space.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: ToppGene
  role: Web service used to compute GO biological-process enrichment of the submitted
    top DEG gene list, returning ranked terms with p-values and FDR
- name: scTITANS
  role: Upstream method that identifies the differential gene list (SigGenes.fdr0.01.txt
    at fdr level 0.01) that is submitted for enrichment
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Constructs the trajectory and pseudotime that scTITANS uses to derive the
    time-dependent DEGs fed into the enrichment analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:81da23f59f1059da20698f51bc634abb9dbe8f50eb44ce9568ebba21636393a4
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-ontology-biological-process-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a list of differential genes (e.g., top scTITANS DEGs from a time-series scRNA-seq dataset such as CEED/GSE126954) by submitting it to the ToppGene Suite for Gene Ontology biological-process enrichment, producing a ranked table of GO terms with p-values and FDR-adjusted significance.

## When to use

Use when you have a ranked or significant gene list (e.g., SigGenes at fdr level 0.01) and need to confirm it is biologically meaningful via GO biological-process term enrichment.

## When NOT to use

- The input is not a gene list but raw scRNA-seq data — run scTITANS first to identify the DEGs before enrichment.
- The analysis goal is cell-type annotation rather than functional interpretation of DEGs — use marker-gene-based annotation (e.g., scCATCH) instead.
- Cell ontology or pathway (non-GO) enrichment is required and GO biological-process terms are not an acceptable annotation space.

## Inputs

- List of top differential genes from scTITANS (SigGenes.fdr0.01.txt), fdr level 0.01
- Organism identifier for the gene list (e.g., Caenorhabditis elegans for CEED/GSE126954)

## Outputs

- Ranked GO biological-process enrichment table (toppgene_go_enrichment.csv) keyed by GO term ID, with term names, p-values, and FDR-adjusted significance

## How to apply

Take the top DEG list produced by scTITANS at the paper's fdr level of 0.01 (e.g., the eleven-of-twelve literature-confirmed CEED genes). Submit the gene list to the ToppGene Suite web service, selecting the correct organism for the data (Caenorhabditis elegans for the CEED dataset) and restricting the analysis to Gene Ontology biological-process terms. Retrieve the ranked enrichment results including GO term IDs, term names, p-values, and FDR-adjusted significance, and compile them into a structured table (toppgene_go_enrichment.csv) keyed by GO term. Interpret the enriched terms against the expected biology of the dataset — for the CEED top 20 DEGs, terms such as 'ascending aorta morphogenesis' reflect typical developmental processes, which supports the validation of the top genes.

## Related tools

- **ToppGene** (Web service used to compute GO biological-process enrichment of the submitted top DEG gene list, returning ranked terms with p-values and FDR)
- **scTITANS** (Upstream method that identifies the differential gene list (SigGenes.fdr0.01.txt at fdr level 0.01) that is submitted for enrichment) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Constructs the trajectory and pseudotime that scTITANS uses to derive the time-dependent DEGs fed into the enrichment analysis)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)  # then submit SigGenes.fdr0.01.txt to ToppGene Suite (organism: Caenorhabditis elegans, category: GO Biological Process)
```

## Evaluation signals

- The output file toppgene_go_enrichment.csv exists and parses as a valid table keyed by GO term.
- The table contains GO biological-process term annotations plus significance statistics (p-value and FDR) computed at the fdr level 0.01.
- Enriched GO terms are consistent with the literature-reported functions of the top DEGs (e.g., typical developmental processes such as ascending aorta morphogenesis for CEED).
- The majority of top DEGs are independently corroborable: in the reference case, eleven of the top 12 scTITANS genes were confirmed via literature surveys.
- The organism selected in ToppGene matches the dataset (Caenorhabditis elegans for GSE126954); a mismatch would invalidate the background term mapping.

## Limitations

- Enrichment depends on the upstream DEG calling quality; scTITANS requires a suitable trajectory reconstruction, and not all TI methods recover the correct branches (e.g., only Monocle3 reconstructed the three branches in MEF).
- The paper does not state software version numbers for tools such as Monocle3, scran, or scTITANS, so exact reproducibility of the DEG list may vary.
- Enrichment of the top 20 DEGs reflects typical developmental processes and may not discriminate fine-grained mechanisms; literature cross-check is used as an additional validation signal.

## Evidence

- [other] Source card workflow: submit the DEG gene list to ToppGene and restrict to GO BP terms: [evidence span withheld — anchor status: not_found]
- [other] Source card validation criterion for the enrichment table: [evidence span withheld — anchor status: not_found]
