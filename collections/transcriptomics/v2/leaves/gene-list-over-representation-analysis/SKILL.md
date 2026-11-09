---
name: gene-list-over-representation-analysis
description: Use when you have a differential-expression gene list (up- or down-regulated
  DEGs), especially from a non-model species, and need to identify which KEGG/GO pathways
  are significantly over-represented.
when_to_use_negative:
- The input is a full count/expression matrix rather than a gene list — use ExpressAnalyst's
  differential-expression module first, since ORA here operates on gene-list mode
  ('datalist1')
- The gene set of interest contains fewer than five DEGs — such sets cannot be called
  significantly enriched under the applied criterion
- The species has gene-level IDs and a reference transcriptome that map cleanly; the
  ortholog-mapping route is mainly needed when native IDs cannot be searched against
  model-organism gene sets
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalyst (ExpressAnalystR)
  role: Web/R platform that performs annotation, ID mapping, and enrichment analysis
    of gene lists synchronized with the web server
  repo: github.com/xia-lab/ExpressAnalystR
- name: KEGG
  role: Pathway gene-set library used as one of the over-representation analysis targets
- name: Gene Ontology (GO)
  role: Ontology providing biological process, molecular function, and cellular component
    gene sets for ORA
- name: EcoOmicsDB
  role: Ortholog database enabling analysis for species without a reference transcriptome,
    used when mapping non-model gene IDs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:2539efcea512ee9e2352daf4715493a189217e0469e82a2a024a89c848c255bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - ExpressAnalyst (ExpressAnalystR)
  - Gene Ontology (GO)
  - KEGG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-list-over-representation-analysis

## Summary

Performs over-representation analysis (ORA) of up- and down-regulated differentially expressed gene (DEG) lists — e.g. from the ExpressAnalyst salamander case study — against KEGG and Gene Ontology (GO BP/MF/CC) gene sets, using ExpressAnalyst/ExpressAnalystR with ortholog-mapped IDs for non-model species.

## When to use

Use when you have a differential-expression gene list (up- or down-regulated DEGs), especially from a non-model species, and need to identify which KEGG/GO pathways are significantly over-represented.

## When NOT to use

- The input is a full count/expression matrix rather than a gene list — use ExpressAnalyst's differential-expression module first, since ORA here operates on gene-list mode ('datalist1')
- The gene set of interest contains fewer than five DEGs — such sets cannot be called significantly enriched under the applied criterion
- The species has gene-level IDs and a reference transcriptome that map cleanly; the ortholog-mapping route is mainly needed when native IDs cannot be searched against model-organism gene sets

## Inputs

- Differentially expressed gene (DEG) table from upstream differential expression analysis, split into up- and down-regulated gene lists
- Species gene identifiers for a non-model organism (to be mapped to ortholog/KO IDs)
- KEGG, GO biological process, GO molecular function, and GO cellular component gene set libraries
- EcoOmicsDB ortholog database for species without a reference transcriptome

## Outputs

- Per-pathway ORA statistics table (enrichment score, p-value, FDR-adjusted p-value, DEG count) for each DEG list
- List of significantly enriched pathways per direction (e.g. 27 up-regulated and 81 down-regulated pathways in the salamander case study)

## How to apply

Load the DEG table from the upstream differential expression analysis and split it into up-regulated and down-regulated gene lists. Map the species' gene identifiers to ortholog IDs using ExpressAnalyst's ID-mapping functions in gene list mode (the resulting qs file is named 'datalist1') so that non-model species IDs can be searched against model-organism gene sets. Run over-representation analysis (the enrichment mode of PerformGSEA) separately for each DEG list against KEGG, GO biological process, GO molecular function, and GO cellular component gene sets. A pathway is accepted as significantly enriched only if its FDR-adjusted p-value is less than 0.05 AND the gene set contains at least five DEGs; note that in the salamander case study the DEGs themselves were defined at a looser threshold (FDR-adjusted p < 0.1) following the original publication. Compile per-pathway statistics (enrichment score, p-value, FDR, DEG count) for both directions into a single results table and check the recovered up/down pathway counts against the reference (e.g. 27 up-regulated pathways for immune response, cell proliferation, programmed cell death; 81 down-regulated pathways for muscle tissue and cellular metabolism).

## Related tools

- **ExpressAnalyst (ExpressAnalystR)** (Web/R platform that performs annotation, ID mapping, and enrichment analysis of gene lists synchronized with the web server) — github.com/xia-lab/ExpressAnalystR
- **KEGG** (Pathway gene-set library used as one of the over-representation analysis targets)
- **Gene Ontology (GO)** (Ontology providing biological process, molecular function, and cellular component gene sets for ORA)
- **EcoOmicsDB** (Ortholog database enabling analysis for species without a reference transcriptome, used when mapping non-model gene IDs)

## Examples

```
library(ExpressAnalystR); Init.Data(FALSE); dataSet <- readDataset("datalist1")
```

## Evaluation signals

- Every reported enriched pathway has an FDR-adjusted p-value < 0.05 and at least five DEGs in the gene set
- ORA is run separately for the up-regulated and down-regulated DEG lists, and both results are compiled into a single table with enrichment score, p-value, FDR, and DEG count
- Gene IDs were mapped to ortholog IDs before enrichment (gene-list qs file 'datalist1'), confirming non-model IDs were searchable against model-organism gene sets
- Re-implementation on the salamander case study recovers 27 up-regulated pathways (immune response, cell proliferation, programmed cell death) and 81 down-regulated pathways (muscle tissue, cellular metabolism)

## Limitations

- The salamander DEG definition used a looser threshold (FDR-adjusted p < 0.1) following the original publication; different upstream DEG thresholds will change the input lists and the enriched pathway counts
- Reads from the same gene may map to similar but different orthologs for different species, so care must be taken while interpreting results for taxonomically diverse comparisons
- ExpressAnalystR is still under development with no guarantee of full functionality, and its R package returns are mainly for statistical analysis rather than visual analytics

## Evidence

- [discussion] Pathway enrichment threshold: FDR < 0.05 and at least five DEGs in the gene set: "A pathway was defined as significantly enriched if the FDR-adjusted p-value was less than 0.05 and there were at least five DEGs in the gene set."
- [discussion] Salamander DEG threshold follows the original publication: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [intro] KEGG and GO gene set libraries: "functions as defined by several widely used pathway libraries and ontologies such as Kyoto Encyclopedia of Genes and Genomes (KEGG) and Gene Ontology (GO)"
- [readme] ExpressAnalystR supports enrichment analysis of gene lists: "It is designed for statistical analysis, enrichment analysis and visual analytics of single and multiple gene expression data, both matrix and gene list."
- [readme] Gene-list mode dataset file naming: "For gene list, the format qs file is named "datalist1". use this function to access:"
- [abstract] EcoOmicsDB enables analysis for species without a reference transcriptome: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
