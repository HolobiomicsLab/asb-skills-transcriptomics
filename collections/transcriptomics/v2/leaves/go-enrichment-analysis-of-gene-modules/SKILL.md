---
name: go-enrichment-analysis-of-gene-modules
description: Use when co-expressed gene modules (e.g., from WGCNA cutreeDynamic on
  CS-CORE co-expression or differential co-expression matrices) need functional annotation
  via GO enrichment to identify relevant biological pathways per module.
when_to_use_negative:
- Modules are from organisms other than human — org.Hs.eg.db with keyType = SYMBOL
  is human-specific
- Input is a single gene list without a defined background/universe — enrichment without
  universe = selected genes inflates significance
- The goal is transcription factor–target enrichment (TRRUST), not GO term enrichment
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: clusterProfiler
  role: Runs enrichGO per gene module with OrgDb = org.Hs.eg.db, keyType = SYMBOL,
    ont = ALL, BH adjustment, pvalueCutoff = 0.05
- name: enrichplot
  role: Visualizes enriched modules (e.g., dotplot of the defense-response module
    in COVID-19 monocytes)
- name: org.Hs.eg.db
  role: Human Gene Ontology annotation database used as OrgDb in enrichGO
- name: WGCNA
  role: Upstream module extraction (adjacency.fromSimilarity, TOMsimilarity, hierarchical
    clustering) whose clusters are the enrichment inputs
- name: dynamicTreeCut
  role: cutreeDynamic with minClusterSize = 10 defines the module gene lists passed
    to enrichGO
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:c7084ef3ac8b08a352e3b2a979f41cb6217e062064b88aca9b3481155e0e71ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CSCORE
  - CSCORE (R package)
  - RColorBrewer
  - Seurat
  - WGCNA
  - clusterProfiler
  - diff_CSCORE
  - dynamicTreeCut
  - enrichplot
  - gridExtra
  - org.Hs.eg.db
  - p.adjust BH
  - pheatmap
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# go-enrichment-analysis-of-gene-modules

## Summary

Functional interpretation of co-expressed gene modules extracted from CS-CORE co-expression networks by running Gene Ontology enrichment with clusterProfiler::enrichGO against org.Hs.eg.db, using the selected genes as the universe and BH-adjusted p-values to flag biologically coherent modules.

## When to use

Use when co-expressed gene modules (e.g., from WGCNA cutreeDynamic on CS-CORE co-expression or differential co-expression matrices) need functional annotation via GO enrichment to identify relevant biological pathways per module.

## When NOT to use

- Modules are from organisms other than human — org.Hs.eg.db with keyType = SYMBOL is human-specific
- Input is a single gene list without a defined background/universe — enrichment without universe = selected genes inflates significance
- The goal is transcription factor–target enrichment (TRRUST), not GO term enrichment

## Inputs

- List of gene symbols per co-expressed module (module_list from dynamicTreeCut clustering)
- Universe gene set: genes selected for co-expression inference (e.g., top 1000 by mean relative expression in CD14 monocytes, or top 5000 by mean expression)
- org.Hs.eg.db annotation database

## Outputs

- Per-module enrichGO result objects with GO terms, counts, and BH-adjusted p-values
- Flagged/enriched module list (modules with >=1 GO term at BH p < 0.001 and >=10 enriched terms)
- Dotplots (enrichplot::dotplot) of enriched GO terms for top modules

## How to apply

After extracting modules with the WGCNA workflow (adjacency.fromSimilarity with power = 1, TOMsimilarity, average-linkage hierarchical clustering, dynamicTreeCut::cutreeDynamic with minClusterSize = 10), collect the gene symbols per module and run clusterProfiler::enrichGO with OrgDb = org.Hs.eg.db, keyType = SYMBOL, ont = 'ALL', pAdjustMethod = 'BH', universe = the genes selected for the co-expression analysis (e.g., top 1000–5000 highly expressed genes), and pvalueCutoff = 0.05. Apply this per module and per cell type; for differential co-expression (diff_CSCORE with donor-level permutations), enrich modules built from significant differential co-expressions (BH-adjusted permutation p < 0.05). Flag a module as meaningfully enriched when it has at least one GO term with BH-adjusted p < 0.001 and at least 10 enriched GO terms. Visualize top modules (e.g., an interferon/defense-response module in COVID monocytes) with enrichplot::dotplot.

## Related tools

- **clusterProfiler** (Runs enrichGO per gene module with OrgDb = org.Hs.eg.db, keyType = SYMBOL, ont = ALL, BH adjustment, pvalueCutoff = 0.05)
- **enrichplot** (Visualizes enriched modules (e.g., dotplot of the defense-response module in COVID-19 monocytes))
- **org.Hs.eg.db** (Human Gene Ontology annotation database used as OrgDb in enrichGO)
- **WGCNA** (Upstream module extraction (adjacency.fromSimilarity, TOMsimilarity, hierarchical clustering) whose clusters are the enrichment inputs)
- **dynamicTreeCut** (cutreeDynamic with minClusterSize = 10 defines the module gene lists passed to enrichGO)

## Examples

```
ego_result <- lapply(1:length(module_list), function(i){ enrichGO(gene = module_list[[i]], OrgDb = 'org.Hs.eg.db', keyType = 'SYMBOL', ont = 'ALL', pAdjustMethod = 'BH', universe = genes_selected, pvalueCutoff = 0.05) })
```

## Evaluation signals

- Each module's enrichGO run used universe = genes_selected (not the full genome) and returned BH-adjusted p-values
- Accepted modules have at least one GO term with BH-adjusted p-value < 0.001 and at least 10 enriched GO terms
- For the COVID-19 monocyte differential analysis, the defense response to virus module shows significant GO enrichment (BH-adjusted p < 1e-3)
- Module gene lists contain >= 10 genes (minClusterSize = 10) and use SYMBOL keyType resolvable against org.Hs.eg.db

## Limitations

- Module-level GO enrichment is only as meaningful as the upstream co-expression inference; with biased co-expression estimates (e.g., inflated false positives from rho-sctransform), enriched terms may reflect artifacts
- Only Gene Ontology (GO) enrichment via enrichGO is demonstrated; other functional databases (e.g., TRRUST TF-target pairs) require separate analyses
- Human-only via org.Hs.eg.db; ont = 'ALL' mixes BP/CC/MF terms and may need post-hoc interpretation

## Evidence

- [methods] GO enrichment with enrichGO on modules (OrgDb, keyType, universe, BH): "enrichGO(gene = module_list[[i]],"
- [methods] GO enrichment described as standard downstream analysis: "Gene Ontology (GO) enrichment analysis is one of the common downstream functional enrichment analyses"
- [methods] OrgDb = org.Hs.eg.db used as human annotation: "OrgDb = 'org.Hs.eg.db', # human"
- [other] Task validation criterion: modules accepted with strong GO enrichment: [evidence span withheld — anchor status: not_found]
- [readme] README vignette describes module extraction plus functional enrichment pipeline: [evidence span withheld — anchor status: external_doc]
