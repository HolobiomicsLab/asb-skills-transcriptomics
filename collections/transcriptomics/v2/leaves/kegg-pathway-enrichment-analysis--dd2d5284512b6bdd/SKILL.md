---
name: kegg-pathway-enrichment-analysis
description: Use when a table of DEGs (up- and down-regulated lists, e.g. from non-model
  species like salamander) needs to be tested for significantly enriched KEGG/GO biological
  functions via ORA.
when_to_use_negative:
- Input is a full count table rather than a DEG gene list — use ExpressAnalyst's expression-matrix
  workflow (normalization, PerformDEAnal) instead, and derive DEG lists first
- You need genome-wide expression profiling directly from raw RNA-seq reads — use
  Seq2Fun/ExpressAnalyst read-quantification modules, not ORA, as ORA only interprets
  pre-computed gene lists
- Vitellogenin or other biomarkers absent from the KO system are of interest — KO-based
  annotation has limited coverage and these genes will not enrich
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0089
tools:
- name: ExpressAnalyst
  role: Web platform / R package performing gene-list annotation (MapListIds/ReadListFile,
    'datalist1' mode) and over-representation analysis (enrichment mode of PerformGSEA)
    against KEGG and GO gene sets
  repo: github.com/xia-lab/ExpressAnalystR
- name: EcoOmicsDB
  role: Ortholog database enabling mapping of non-model species gene identifiers onto
    model-organism gene sets used in enrichment analysis
- name: KEGG
  role: Pathway library supplying gene sets for over-representation analysis
- name: Gene Ontology (GO)
  role: Ontology supplying biological process, molecular function, and cellular component
    gene sets for over-representation analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:7d63973b3ccad257976351131439d7d812d9f5d0f02a6a89478cfae20a397f38
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - Gene Ontology (GO)
  - KEGG
  - TFvelo
  - gseapy
  - scEFSC
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kegg-pathway-enrichment-analysis

## Summary

Over-representation analysis (ORA) of up- and down-regulated differentially expressed gene (DEG) lists against KEGG and Gene Ontology (GO) gene sets using ExpressAnalyst, enabling functional interpretation in non-model species via EcoOmicsDB ortholog mapping. In the salamander case study this recovers 27 enriched pathways among up-regulated genes and 81 among down-regulated genes.

## When to use

Use when a table of DEGs (up- and down-regulated lists, e.g. from non-model species like salamander) needs to be tested for significantly enriched KEGG/GO biological functions via ORA.

## When NOT to use

- Input is a full count table rather than a DEG gene list — use ExpressAnalyst's expression-matrix workflow (normalization, PerformDEAnal) instead, and derive DEG lists first
- You need genome-wide expression profiling directly from raw RNA-seq reads — use Seq2Fun/ExpressAnalyst read-quantification modules, not ORA, as ORA only interprets pre-computed gene lists
- Vitellogenin or other biomarkers absent from the KO system are of interest — KO-based annotation has limited coverage and these genes will not enrich

## Inputs

- DEG table from upstream differential expression analysis (up- and down-regulated gene lists, e.g. salamander DEGs called at FDR-adjusted p < 0.1)
- Gene identifier type for the non-model species (for ortholog mapping via EcoOmicsDB)
- Gene set libraries: KEGG, GO biological process, GO molecular function, GO cellular component

## Outputs

- Per-direction results table of enriched pathways with enrichment score, p-value, FDR, and DEG count
- Count of significantly enriched pathways (expected 27 for up-regulated, 81 for down-regulated in the salamander case)
- Biological interpretation: up-regulated genes enrich immune response, cell proliferation, programmed cell death; down-regulated genes enrich muscle tissue and cellular metabolism

## How to apply

Split the DEG table from the upstream differential expression analysis into separate up-regulated and down-regulated gene lists. Map non-model species gene identifiers to ortholog IDs using ExpressAnalyst's MapListIds/ReadListFile functions in gene list mode (the results file is named 'datalist1'), so the IDs can be searched against model-organism gene sets in EcoOmicsDB. Run the over-representation analysis (the enrichment mode of PerformGSEA) separately for each DEG list against KEGG, GO biological process, GO molecular function, and GO cellular component gene sets. Apply the significance criterion of FDR-adjusted p-value < 0.05 with at least five DEGs in the gene set (salamander DEGs themselves were called at FDR < 0.1, following the original publication). Compile per-pathway statistics — enrichment score, p-value, FDR, DEG count — into a single results table for both directions.

## Related tools

- **ExpressAnalyst** (Web platform / R package performing gene-list annotation (MapListIds/ReadListFile, 'datalist1' mode) and over-representation analysis (enrichment mode of PerformGSEA) against KEGG and GO gene sets) — github.com/xia-lab/ExpressAnalystR
- **EcoOmicsDB** (Ortholog database enabling mapping of non-model species gene identifiers onto model-organism gene sets used in enrichment analysis)
- **KEGG** (Pathway library supplying gene sets for over-representation analysis)
- **Gene Ontology (GO)** (Ontology supplying biological process, molecular function, and cellular component gene sets for over-representation analysis)

## Examples

```
library(ExpressAnalystR); Init.Data(FALSE); dataSet <- readDataset("datalist1")
```

## Evaluation signals

- A pathway is accepted as significantly enriched only if its FDR-adjusted p-value is < 0.05 AND the gene set contains at least five DEGs
- Reproduced up/down pathway counts should match the reported 27 up-regulated and 81 down-regulated enriched pathways for the salamander case study
- Enriched terms should match reported biology: up-regulated pathways mainly immune response, cell proliferation, and programmed cell death; down-regulated mainly muscle tissue and cellular metabolism
- All ExpressAnalystR commands must be executed in sequence without skipping steps, and results are read back via readDataset on the 'datalist1' qs file for gene-list mode
- Per-pathway statistics table should contain enrichment score, p-value, FDR, and DEG count columns for both up and down directions

## Limitations

- Reads/genes from the same gene may map to similar but different orthologs across species, requiring care in interpretation for taxonomically diverse comparisons
- Improper sub-group database selection in EcoOmicsDB (e.g. protist reads mapped to the 'fishes' database) may miss key functional groups
- Many uncharacterized IDs remain in published reference genomes of non-model organisms, limiting identifier mapping

## Evidence

- [discussion] Pathway enrichment significance criterion: "A pathway was defined as significantly enriched if the FDR-adjusted p-value was less than 0.05 and there were at least five DEGs in the gene set."
- [discussion] Salamander DEG threshold: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [readme] README: gene-list results file and read-back function: [evidence span withheld — anchor status: not_found]
- [readme] README: R package scope for annotation and enrichment-backed analysis: "It is designed for statistical analysis, enrichment analysis and visual analytics of single and multiple gene expression data, both matrix and gene list."
- [abstract] Ortholog database enabling non-model species analysis: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
