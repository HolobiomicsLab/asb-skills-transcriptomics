---
name: differential-expression-fold-change-ranking
description: Use when you have differential expression results (gene-level log fold-changes)
  from perturbed 3T3-L1 adipocyte microarray data and need a fold-change-ranked gene
  list as input to clusterProfiler GSEA for GO term enrichment.
when_to_use_negative:
- Input is RNA-seq count data (use curatedAdipoRNA) rather than the curated 3T3-L1
  microarray dataset in curatedAdipoArray
- You need an over-representation (ORA) test on a pre-selected significant gene list
  rather than GSEA on a full fold-change-ranked list
- Datasets were excluded during curation for missing probe annotations — ranking is
  invalid for genes lacking probe-to-gene-symbol mapping
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: curatedAdipoArray
  role: Source of the curated, gene-collapsed 3T3-L1 microarray dataset and differential
    expression results used to build the ranked list
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Provides access to the clean and processed SummarizedExperiment data objects
- name: clusterProfiler
  role: Runs the GSEA of the fold-change-ranked gene list against GO (BP, CC, MF)
- name: Org.Mm.eg.db
  role: Mouse gene-to-GO annotation used to map gene identifiers for enrichment testing
- name: GO.db
  role: GO term definitions and ontology structure for the tested terms
- name: limma
  role: Used upstream to log-transform and normalize gene intensities across studies,
    underpinning the fold-change estimates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:ec743e3cf2581454992415c04755818fbd519d18e6c810bf0eb7c120d0f09dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - GO.db
  - Org.Mm.eg.db
  - R/Bioconductor
  - clusterProfiler
  - curatedAdipoArray
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Differential expression fold-change ranking for GSEA

## Summary

Ranks all genes from a differential expression analysis by decreasing log fold-change to produce the ranked gene list required by GSEA, as demonstrated for MDI-induced 3T3-L1 microarray data under Pparg knockdown using the curatedAdipoArray Bioconductor package.

## When to use

Use when you have differential expression results (gene-level log fold-changes) from perturbed 3T3-L1 adipocyte microarray data and need a fold-change-ranked gene list as input to clusterProfiler GSEA for GO term enrichment.

## When NOT to use

- Input is RNA-seq count data (use curatedAdipoRNA) rather than the curated 3T3-L1 microarray dataset in curatedAdipoArray
- You need an over-representation (ORA) test on a pre-selected significant gene list rather than GSEA on a full fold-change-ranked list
- Datasets were excluded during curation for missing probe annotations — ranking is invalid for genes lacking probe-to-gene-symbol mapping

## Inputs

- Processed 3T3-L1 microarray SummarizedExperiment object from curatedAdipoArray (accessed through ExperimentHub)
- Gene-level differential expression results with log fold-changes (e.g. Pparg knockdown vs. control)
- Mouse GO annotation packages Org.Mm.eg.db and GO.db

## Outputs

- Gene list ranked by decreasing log fold-change (GSEA input)
- Enrichment result table with GO term IDs, descriptions, enrichment scores and adjusted p-values (go_enrichment_results.csv)

## How to apply

Load the processed SummarizedExperiment object for the curated 3T3-L1 microarray dataset from ExperimentHub via the curatedAdipoArray package (installed with BiocManager). Take the precomputed differential expression results — computed on data that was log-transformed, normalized across studies with limma, and batch-corrected (data series and platform) with sva — and sort all genes by decreasing log fold-change to build the ranked gene list. Feed this ranked list to clusterProfiler's GSEA against the Gene Ontology (testing BP, CC and MF ontologies) using Org.Mm.eg.db and GO.db for mouse annotation. Rationale: GSEA needs the full gene universe ordered by a biologically meaningful statistic, and fold-change ranking directly reflects the direction of the perturbation (e.g. Pparg knockdown) without imposing an arbitrary significance cutoff. Extract significant enriched terms with their enrichment scores and adjusted p-values and export them as a table (e.g. go_enrichment_results.csv).

## Related tools

- **curatedAdipoArray** (Source of the curated, gene-collapsed 3T3-L1 microarray dataset and differential expression results used to build the ranked list) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Provides access to the clean and processed SummarizedExperiment data objects)
- **clusterProfiler** (Runs the GSEA of the fold-change-ranked gene list against GO (BP, CC, MF))
- **Org.Mm.eg.db** (Mouse gene-to-GO annotation used to map gene identifiers for enrichment testing)
- **GO.db** (GO term definitions and ontology structure for the tested terms)
- **limma** (Used upstream to log-transform and normalize gene intensities across studies, underpinning the fold-change estimates)

## Examples

```
BiocManager::install("curatedAdipoArray"); library(ExperimentHub); eh <- ExperimentHub(); se <- query(eh, "curatedAdipoArray")[[1]]
```

## Evaluation signals

- The output enrichment table contains all five expected GO terms: GO:0050873 (brown fat cell differentiation), GO:0045600 (positive regulation of fat cell differentiation), GO:0033993 (response to lipid), GO:0019395 (fatty acid oxidation) and GO:0030374 (nuclear receptor transcription coactivator activity)
- The five terms show significant enrichment consistent with the reported adipogenic biology
- The ranked gene list is sorted in decreasing log fold-change order and covers all genes, not only significant ones (GSEA requirement)
- The enrichment table includes GO term IDs, descriptions, enrichment scores and adjusted p-values for every reported term
- The input data are gene-level (probe-collapsed via collapseRows) and batch-corrected with sva, not raw probe intensities

## Limitations

- Two datasets were excluded during curation for missing probe annotations, so ranking quality depends on complete probe-to-gene-symbol mapping
- Fold-changes come from microarray data processed with limma normalization across studies and sva batch removal (data series and platform); residual batch artifacts would distort the ranking
- The package contains no R functions itself — ranking and GSEA must be implemented by the user in downstream scripts

## Evidence

- [intro] The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment.: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [readme] Access to the clean and the processed data SummarizedExperiment objects through ExperimentHub.: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [methods] the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
