---
name: differentially-expressed-gene-stratification
description: Use when a differential expression table (FDR-adjusted p < 0.05, |log2
  FC| > 1.5) is available and direction-specific pathway enrichment counts are required.
when_to_use_negative:
- Input is a raw count matrix or FASTQ data — run the differential expression pipeline
  (PerformDEAnal/limma) first; stratification operates on an existing DE results table
  or gene lists
- The DE gene set contains fewer than five genes mapped to a pathway — such gene sets
  cannot pass the enrichment significance criterion, so direction-specific ORA will
  return no result
- The species' identifiers cannot be mapped to orthologs (e.g. EcoOmicsDB subgroup
  not selected properly) — gene-list-mode ORA against KEGG/GO will miss key functional
  groups
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ExpressAnalyst
  role: Web platform whose gene list mode splits DEG lists ('datalist1'), maps IDs
    to orthologs via EcoOmicsDB, and runs over-representation analysis (enrichment
    mode of PerformGSEA) per direction against KEGG/GO gene sets
  repo: github.com/xia-lab/ExpressAnalystR
- name: ExpressAnalystR
  role: R package synchronized with the web server used to reproduce gene-list enrichment
    results locally; gene-list datasets are saved as a qs file named 'datalist1' and
    read back with readDataset()
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:4dc062f46b29f7b34f836075c0c0fc0f181b5f8e807ff71511cdcbaad28edcfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - Gene Ontology (GO)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# differentially-expressed-gene-stratification

## Summary

Splits a differential expression result table into separate up-regulated and down-regulated gene lists so each direction can be run through over-representation analysis independently, recovering direction-specific biological signals (e.g. 27 up vs 81 down enriched pathways in the salamander case study).

## When to use

Use when a differential expression table (FDR-adjusted p < 0.05, |log2 FC| > 1.5) is available and direction-specific pathway enrichment counts are required.

## When NOT to use

- Input is a raw count matrix or FASTQ data — run the differential expression pipeline (PerformDEAnal/limma) first; stratification operates on an existing DE results table or gene lists
- The DE gene set contains fewer than five genes mapped to a pathway — such gene sets cannot pass the enrichment significance criterion, so direction-specific ORA will return no result
- The species' identifiers cannot be mapped to orthologs (e.g. EcoOmicsDB subgroup not selected properly) — gene-list-mode ORA against KEGG/GO will miss key functional groups

## Inputs

- Differential expression results table with per-gene FDR-adjusted p-values and log fold changes
- Up-regulated gene list (non-model species identifiers, e.g. salamander transcript IDs)
- Down-regulated gene list in ExpressAnalyst gene-list format ('datalist1' qs file)
- Pathway/ontology gene set libraries: KEGG, GO biological process, GO molecular function, GO cellular component via EcoOmicsDB

## Outputs

- Per-direction ORA results table with enrichment score, p-value, FDR, and DEG count per pathway (e.g. 27 up-regulated pathways; 81 down-regulated pathways)
- Split up-regulated and down-regulated gene lists mapped to ortholog IDs
- Direction-level functional interpretation (e.g. immune response/cell proliferation/apoptosis vs muscle tissue/metabolism)

## How to apply

Load the differential expression table and split it into up-regulated and down-regulated gene lists using the study's DEG criterion (FDR-adjusted p < 0.05 and absolute log fold change > 1.5; the salamander case study followed the original publication and used FDR < 0.1). In ExpressAnalyst's gene list mode, read each list into the 'datalist1' object and map non-model species identifiers to ortholog IDs with the MapListIds/ReadListFile functions so the lists can be searched against model-organism gene sets. Run over-representation analysis (enrichment mode of PerformGSEA) separately per direction against KEGG, GO biological process, GO molecular function, and GO cellular component sets. A pathway is accepted as significantly enriched if its FDR-adjusted p-value is < 0.05 and the gene set contains at least five DEGs; compile per-pathway statistics (enrichment score, p-value, FDR, DEG count) for both directions into one results table and confirm the direction-specific counts.

## Related tools

- **ExpressAnalyst** (Web platform whose gene list mode splits DEG lists ('datalist1'), maps IDs to orthologs via EcoOmicsDB, and runs over-representation analysis (enrichment mode of PerformGSEA) per direction against KEGG/GO gene sets) — github.com/xia-lab/ExpressAnalystR
- **ExpressAnalystR** (R package synchronized with the web server used to reproduce gene-list enrichment results locally; gene-list datasets are saved as a qs file named 'datalist1' and read back with readDataset()) — github.com/xia-lab/ExpressAnalystR

## Examples

```
Init.Data(FALSE); SetAnalType("onedata"); dataSet <- readDataset("datalist1")
```

## Evaluation signals

- Up- and down-regulated ORA counts reproduce the reported 27 up-regulated and 81 down-regulated significantly enriched pathways for the salamander DEGs
- Every accepted pathway has FDR-adjusted p < 0.05 and at least five DEGs in the gene set
- Gene lists were mapped to ortholog IDs before enrichment (non-model IDs are searchable against model-organism gene sets via EcoOmicsDB)
- Compiled results table contains enrichment score, p-value, FDR, and DEG count columns for both directions
- Direction-specific functional themes match reported biology: up = immune response, cell proliferation, programmed cell death; down = muscle tissue and cellular metabolism

## Limitations

- Pathway calls depend on the DEG threshold: the paper's general criterion is FDR < 0.05 with |log2 FC| > 1.5, but the salamander case study used FDR < 0.1 following the original publication, so counts are threshold-sensitive
- Improper ortholog subgroup database selection (EcoOmicsDB) may cause key functional groups to be missed
- Reads from the same gene may map to similar but different orthologs across species, requiring care in interpretation for taxonomically diverse comparisons

## Evidence

- [discussion] DEG threshold for inclusion in the stratified lists: "Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5."
- [discussion] Pathway significance criterion for stratified ORA: "A pathway was defined as significantly enriched if the FDR-adjusted p-value was less than 0.05 and there were at least five DEGs in the gene set."
- [discussion] Salamander case study used a looser FDR threshold per original publication: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [abstract] Gene-list mode is an explicit input type of the platform: "a series of modules that cover from processing and annotation of FASTQ files to statistical and functional analysis of count tables or gene lists"
- [readme] Gene-list datasets are stored and read back as 'datalist1' qs files in ExpressAnalystR: "The results from R functions are saved in a format qs file named as the file name of original data table. For gene list, the format qs file is named "datalist1"."
