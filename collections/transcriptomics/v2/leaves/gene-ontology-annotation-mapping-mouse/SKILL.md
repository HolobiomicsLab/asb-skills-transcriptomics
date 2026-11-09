---
name: gene-ontology-annotation-mapping-mouse
description: Use when mouse (3T3-L1) differential expression results need to be translated
  into GO terms — e.g. ranking genes by log fold-change and testing which adipogenic
  GO categories (BP/CC/MF) are significantly enriched.
when_to_use_negative:
- Input is non-mouse (e.g. human) gene expression — Org.Mm.eg.db and the 3T3-L1 curated
  datasets are mouse-specific
- Input is raw probe-level intensities rather than gene-collapsed differential expression
  results — the curatedAdipoArray data are already collapsed to gene symbols using
  collapsRows
- The goal is RNA-seq count-level enrichment (curatedAdipoRNA) or ChIP-seq peak enrichment
  (curatedAdipoChIP) rather than microarray expression ranking
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: curatedAdipoArray
  role: Provides the curated 3T3-L1 microarray SummarizedExperiment used as the source
    of differential expression and fold-change rankings
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: clusterProfiler
  role: Runs the gene set enrichment analysis (enrichGO/GSEA) against the Gene Ontology
- name: Org.Mm.eg.db
  role: Mouse gene-to-GO annotation mapping used by clusterProfiler
- name: GO.db
  role: GO ontology definitions (BP, CC, MF) backing the enrichment test
- name: ExperimentHub
  role: Access point for the clean and processed SummarizedExperiment data objects
- name: BiocManager
  role: Installs curatedAdipoArray and annotation packages from Bioconductor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:52b85b56b480c7aa5797f45e0098d1c6fdebc0f4a1273c154a4c66f81c56ae33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - BiocManager
  - ExperimentHub
  - GO.db
  - Org.Mm.eg.db
  - R/Bioconductor
  - clusterProfiler
  - curatedAdipoArray
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-ontology-annotation-mapping-mouse

## Summary

Maps differentially expressed genes from mouse 3T3-L1 adipocyte microarray data to Gene Ontology terms and identifies enriched GO categories via clusterProfiler GSEA against BP, CC and MF ontologies, using Org.Mm.eg.db/GO.db annotations.

## When to use

Use when mouse (3T3-L1) differential expression results need to be translated into GO terms — e.g. ranking genes by log fold-change and testing which adipogenic GO categories (BP/CC/MF) are significantly enriched.

## When NOT to use

- Input is non-mouse (e.g. human) gene expression — Org.Mm.eg.db and the 3T3-L1 curated datasets are mouse-specific
- Input is raw probe-level intensities rather than gene-collapsed differential expression results — the curatedAdipoArray data are already collapsed to gene symbols using collapsRows
- The goal is RNA-seq count-level enrichment (curatedAdipoRNA) or ChIP-seq peak enrichment (curatedAdipoChIP) rather than microarray expression ranking

## Inputs

- curatedAdipoArray SummarizedExperiment object of MDI-induced 3T3-L1 microarray samples (accessed via ExperimentHub)
- Differential expression results with per-gene log fold-change values (mouse genes, collapsed from probes to gene symbols)

## Outputs

- Ranked gene list (genes ordered by decreasing log fold-change)
- Enriched GO term table (GO IDs, term descriptions, enrichment scores, adjusted p-values) written to go_enrichment_results.csv

## How to apply

Load the curated 3T3-L1 microarray SummarizedExperiment from the curatedAdipoArray package through ExperimentHub (installed via BiocManager). Take the differential expression results and rank all genes by decreasing log fold-change to form the ranked gene list required by GSEA. Run clusterProfiler gene set enrichment (enrichGO/GSEA) against the Gene Ontology using the mouse annotation packages Org.Mm.eg.db and GO.db, testing the standard GO ontologies (BP, CC, MF). Extract the significant enriched terms (e.g. GO:0050873 'brown fat cell differentiation', GO:0045600 'positive regulation of fat cell differentiation', GO:0033993 'response to lipid', GO:0019395 fatty acid oxidation, GO:0030374 nuclear receptor coactivator activity) and export GO IDs, descriptions, enrichment scores and adjusted p-values to a table such as go_enrichment_results.csv.

## Related tools

- **curatedAdipoArray** (Provides the curated 3T3-L1 microarray SummarizedExperiment used as the source of differential expression and fold-change rankings) — https://github.com/MahShaaban/curatedAdipoArray
- **clusterProfiler** (Runs the gene set enrichment analysis (enrichGO/GSEA) against the Gene Ontology)
- **Org.Mm.eg.db** (Mouse gene-to-GO annotation mapping used by clusterProfiler)
- **GO.db** (GO ontology definitions (BP, CC, MF) backing the enrichment test)
- **ExperimentHub** (Access point for the clean and processed SummarizedExperiment data objects)
- **BiocManager** (Installs curatedAdipoArray and annotation packages from Bioconductor)

## Examples

```
BiocManager::install("curatedAdipoArray"); library(curatedAdipoArray); se <- curatedAdipoArray::APIArray()
```

## Evaluation signals

- The enrichment result table contains all five reported GO terms: GO:0050873, GO:0045600, GO:0033993, GO:0019395 and GO:0030374
- Each of the five GO terms is significant and biologically consistent with the reported adipogenic phenotype (brown fat cell differentiation, positive regulation of fat cell differentiation, response to lipid, fatty acid oxidation, nuclear receptor coactivator activity)
- The ranked gene list is sorted by decreasing log fold-change and covers all genes from the differential expression analysis
- The output table (go_enrichment_results.csv) contains GO term IDs, descriptions, enrichment scores and adjusted p-values
- Genes are mapped through mouse annotations (Org.Mm.eg.db) with no unmapped-symbol losses inconsistent with the curated, probe-collapsed dataset

## Limitations

- The enrichment is anchored to a specific curated microarray resource; two datasets in the underlying resource were excluded for missing probe annotations, so annotation completeness governs which genes can be mapped
- GO enrichment reflects the microarray platform's gene coverage after probe-to-symbol collapsing; terms absent from the measured genes cannot be detected
- Ranking by fold-change alone (without a statistic-based ranking metric) can overweight large-magnitude but noisy changes

## Evidence

- [intro] The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment.: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [readme] This package is for documenting and distributing a curated dataset of gene expression from MDI-induced 3T3-L1 adipocyte cell model under genetic and pharmacological modification.: "This package is for documenting and distributing a curated dataset of gene expression from MDI-induced 3T3-L1 adipocyte cell model under genetic and pharmacological modification."
- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
