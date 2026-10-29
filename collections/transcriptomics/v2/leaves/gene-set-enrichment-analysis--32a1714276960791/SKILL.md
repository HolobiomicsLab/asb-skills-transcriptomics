---
name: gene-set-enrichment-analysis
description: Use when you have differential expression results from the curatedAdipoArray
  3T3-L1 microarray data (e.g., Pparg knockdown) and need to rank all genes by decreasing
  log fold-change to identify significantly enriched GO terms via GSEA rather than
  overlap-based ORA.
when_to_use_negative:
- The input is raw probe intensities or unprocessed microarray data rather than differential
  expression results — data curation/normalization (limma, sva batch removal) must
  happen first.
- The analysis target is the RNA-Seq (curatedAdipoRNA) or ChIP-Seq (curatedAdipoChIP)
  datasets, whose RangedSummarizedExperiment objects require differential expression/peak-calling
  before enrichment.
- You need over-representation analysis (ORA) on a small pre-defined DE gene list
  rather than GSEA on a full fold-change-ranked list.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: clusterProfiler
  role: Runs the GSEA enrichment against GO for the fold-change-ranked gene list
- name: curatedAdipoArray
  role: Provides the curated 3T3-L1 microarray dataset and differential expression
    input
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: Org.Mm.eg.db
  role: Mouse gene-to-GO annotation mapping used as TERM2GENE/OrgDb for enrichment
- name: GO.db
  role: Supplies GO term definitions and ontology structure for BP, CC, MF testing
- name: ExperimentHub
  role: Access point for the processed SummarizedExperiment data object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:670e18f8e4d161433b7ec7e82319b18b0194f5358e227aeea54e84949f20f732
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-set-enrichment-analysis

## Summary

Runs Gene Set Enrichment Analysis (GSEA) on a fold-change-ranked gene list from MDI-induced 3T3-L1 adipocyte microarray data to detect enriched Gene Ontology terms under perturbation, using clusterProfiler with mouse GO annotations.

## When to use

Use when you have differential expression results from the curatedAdipoArray 3T3-L1 microarray data (e.g., Pparg knockdown) and need to rank all genes by decreasing log fold-change to identify significantly enriched GO terms via GSEA rather than overlap-based ORA.

## When NOT to use

- The input is raw probe intensities or unprocessed microarray data rather than differential expression results — data curation/normalization (limma, sva batch removal) must happen first.
- The analysis target is the RNA-Seq (curatedAdipoRNA) or ChIP-Seq (curatedAdipoChIP) datasets, whose RangedSummarizedExperiment objects require differential expression/peak-calling before enrichment.
- You need over-representation analysis (ORA) on a small pre-defined DE gene list rather than GSEA on a full fold-change-ranked list.

## Inputs

- Differential expression results (log fold-change per gene) from the curatedAdipoArray SummarizedExperiment object of MDI-induced 3T3-L1 samples
- Gene Ontology gene sets (BP, CC, MF) via Org.Mm.eg.db and GO.db mouse annotation

## Outputs

- go_enrichment_results.csv table with GO term IDs, descriptions, enrichment scores, and adjusted p-values
- GSEA result object from clusterProfiler listing enriched GO terms

## How to apply

Load the curated 3T3-L1 microarray SummarizedExperiment object from curatedAdipoArray through ExperimentHub in R/Bioconductor. Rank all genes by decreasing log fold-change from the differential expression results — GSEA needs the full ranked list, not a thresholded DE gene set. Run clusterProfiler GSEA against Gene Ontology (BP, CC, MF) using Org.Mm.eg.db and GO.db as the mouse annotation backend. Extract enriched terms and confirm the adipogenic biology is recovered: GO:0050873 (brown fat cell differentiation), GO:0045600 (positive regulation of fat cell differentiation), GO:0033993 (response to lipid), GO:0019395 (fatty acid oxidation), and GO:0030374 (nuclear receptor coactivator activity). Write GO term IDs, descriptions, enrichment scores and adjusted p-values to go_enrichment_results.csv.

## Related tools

- **clusterProfiler** (Runs the GSEA enrichment against GO for the fold-change-ranked gene list)
- **curatedAdipoArray** (Provides the curated 3T3-L1 microarray dataset and differential expression input) — https://github.com/MahShaaban/curatedAdipoArray
- **Org.Mm.eg.db** (Mouse gene-to-GO annotation mapping used as TERM2GENE/OrgDb for enrichment)
- **GO.db** (Supplies GO term definitions and ontology structure for BP, CC, MF testing)
- **ExperimentHub** (Access point for the processed SummarizedExperiment data object)

## Examples

```
BiocManager::install("curatedAdipoArray"); gsea_res <- GSEA(ranked_genes, TERM2GENE = go_bp, OrgDb = "org.Mm.eg.db", ont = "BP")
```

## Evaluation signals

- go_enrichment_results.csv contains all five expected GO terms: GO:0050873, GO:0045600, GO:0033993, GO:0019395, GO:0030374
- Each of the five terms is reported with significant adjusted p-value and its enrichment score
- The ranked gene list is sorted by decreasing log fold-change covering all genes, not only significant DE hits
- Enriched terms are biologically consistent with adipogenesis (brown fat cell differentiation, fat cell differentiation, response to lipid, fatty acid oxidation, nuclear receptor coactivator activity)

## Limitations

- The input data are microarray-derived and were normalized across studies with limma and batch-corrected with sva (data series and platform batch effects), which shapes the fold-change ranking fed to GSEA.
- The curated resource excluded two studies for missing probe annotations, so coverage of perturbations depends on dataset completeness.
- GO enrichment is restricted to standard GO ontologies (BP, CC, MF) via Org.Mm.eg.db; non-GO gene sets are not covered here.

## Evidence

- [intro] Access to the clean and the processed data SummarizedExperiment objects through ExperimentHub: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [readme] The curated data support downstream analyses such as differential expression and gene set enrichment: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [methods] Curated dataset of microarray samples of MDI-induced pre-adipocytes: "A curated dataset of Microarrays samples. The samples are MDI-induced pre- adipocytes (3T3-L1) at different time points/stage of differentiation"
