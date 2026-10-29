---
name: gene-ontology-enrichment-analysis
description: Use when you have a gene list derived from differential IPA usage (or
  another differential transcript event) and want to identify enriched GO biological-process
  terms explaining which functions the affected genes share.
when_to_use_negative:
- The input is raw read counts or transcript TPMs rather than a curated list of genes
  passing a differential significance cutoff
- You need pathway-level or custom-geneset enrichment rather than GO biological-process
  terms
- The gene list is too small or unfiltered to support FDR < 0.05 enrichment testing
edam_operation: http://edamontology.org/operation_3941
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: ClusterProfiler
  role: Performs Gene Ontology biological-process enrichment on the gene list from
    differentially used IPA sites, with significance defined at FDR < 0.05
- name: DRIMSeq
  role: 'Upstream: Dirichlet-multinomial differential transcript usage analysis that
    produces the significant IPA event list whose genes feed the enrichment'
- name: InPACT
  role: 'Upstream: identifies and quantifies novel IPA sites/isoforms whose differential
    usage defines the gene set'
  repo: https://github.com/YY-TMU/InPACT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:bda8a303d997e8c1ce095161b6a56410b0faeac6dae4990706368c1d602f2334
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - ClueGO
  - ClusterProfiler
  - Cytoscape
  - DRIMSeq
  - GOSemSim
  - InPACT
  - MU-PseUDeep
  - Salmon
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_3941
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-ontology-enrichment-analysis

## Summary

Perform Gene Ontology biological-process enrichment on genes carrying differentially used intronic polyadenylation (IPA) sites from the LPS monocyte activation analysis, using ClusterProfiler to reveal biological processes associated with IPA regulation.

## When to use

Use when you have a gene list derived from differential IPA usage (or another differential transcript event) and want to identify enriched GO biological-process terms explaining which functions the affected genes share.

## When NOT to use

- The input is raw read counts or transcript TPMs rather than a curated list of genes passing a differential significance cutoff
- You need pathway-level or custom-geneset enrichment rather than GO biological-process terms
- The gene list is too small or unfiltered to support FDR < 0.05 enrichment testing

## Inputs

- Gene symbol list for genes carrying significantly differentially used IPA sites (from DRIMSeq output at |log2fold_change| > 1 and FDR < 0.05)
- DRIMSeq differential transcript usage results table (204 significant events in the monocyte activation analysis)

## Outputs

- Table of enriched GO biological-process terms with FDR-adjusted p-values (significant at FDR < 0.05)
- Mapping of enriched GO terms to input genes for interpretation of IPA-regulated processes

## How to apply

First run differential transcript usage (DRIMSeq Dirichlet-multinomial model) between control and LPS-stimulated monocyte samples, calling significant events at |log2fold_change| > 1 and FDR < 0.05, and collect the gene symbols of the significant IPA events. Then supply this gene list to ClusterProfiler version 3.18.1 to perform Gene Ontology biological-process (BP) enrichment. Define significantly enriched GO terms at FDR < 0.05. The enrichment connects IPA regulation of genes such as ARHGAP24 to biological processes coordinated during monocyte activation.

## Related tools

- **ClusterProfiler** (Performs Gene Ontology biological-process enrichment on the gene list from differentially used IPA sites, with significance defined at FDR < 0.05)
- **DRIMSeq** (Upstream: Dirichlet-multinomial differential transcript usage analysis that produces the significant IPA event list whose genes feed the enrichment)
- **InPACT** (Upstream: identifies and quantifies novel IPA sites/isoforms whose differential usage defines the gene set) — https://github.com/YY-TMU/InPACT

## Examples

```
Rscript -e 'library(clusterProfiler); ego <- enrichGO(gene = sig_ipa_genes, OrgDb = org.Hs.eg.db, ont = "BP", p.adjust.method = "BH", qvalueCutoff = 0.05)'
```

## Evaluation signals

- The enrichment is run only on genes from DRIMSeq-significant IPA events meeting |log2fold_change| > 1 and FDR < 0.05
- ClusterProfiler version 3.18.1 is used and GO biological-process ontology is selected
- Enriched terms are reported only at FDR < 0.05
- Enriched processes are biologically consistent with monocyte activation (e.g., immune-response terms) and connect to genes such as ARHGAP24

## Limitations

- Enrichment interpretation depends on the upstream DRIMSeq cutoffs; different thresholds change the input gene list
- GO biological-process terms may not fully clarify roles such as ARHGAP24 IPA in monocyte activation, for which the paper notes a comprehensive study is still needed
- The paper does not report a background/universe specification for the enrichment

## Evidence

- [other] Perform Gene Ontology biological-process enrichment on genes carrying differentially used IPA sites with ClusterProfiler version 3.18.1, defining significantly enriched terms at FDR < 0.05.: [evidence span withheld — anchor status: not_found]
- [methods] by the cutoff of | log2fold_change | > 1 and FDR < 0.05, thereby identifying the differentially used IPA sites: "by the cutoff of | log2fold_change | > 1 and FDR < 0.05, thereby identifying the differentially used IPA sites"
- [methods] we applied DRIMSeq to conduct differential transcript usage analysis between control and LPS stimulated group: "we applied DRIMSeq to conduct differential transcript usage analysis between control and LPS stimulated group"
- [discussion] Undoubtedly, a comprehensive study is needed to clarify the role played by ARHGAP24 IPA in monocyte activation: "Undoubtedly, a comprehensive study is needed to clarify the role played by ARHGAP24 IPA in monocyte activation"
