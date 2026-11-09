---
name: adipocyte-differentiation-gene-signature-interpretation
description: Use when a curated 3T3-L1 adipocyte dataset (e.g. from curatedAdipoArray)
  has differential-expression results available and the goal is to identify enriched
  GO terms among genes ranked by fold-change rather than test a single gene list.
when_to_use_negative:
- Input is raw probe-level intensities that have not been mapped and collapsed to
  gene symbols — collapse probes first (e.g. with collapseRows) before ranking genes.
- The goal is calling individual differentially expressed genes rather than interpreting
  a coordinated expression signature via gene set enrichment.
- The data are RNA-seq or ChIP-seq samples (curatedAdipoRNA / curatedAdipoChIP) rather
  than the curatedAdipoArray microarray SummarizedExperiment.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3308
tools:
- name: curatedAdipoArray
  role: Source of the curated 3T3-L1 microarray SummarizedExperiment data used to
    derive the fold-change-ranked gene list
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Access point for the clean and processed SummarizedExperiment data objects
- name: clusterProfiler
  role: Runs gene set enrichment analysis (enrichGO/GSEA) of the ranked gene list
    against the Gene Ontology
- name: Org.Mm.eg.db
  role: Mouse gene-to-GO annotation backend used to map gene identifiers to GO terms
- name: GO.db
  role: Gene Ontology term definitions and ontology structure for the BP/CC/MF tests
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:c1db89741033d29c49455d763b96d0fe11fb47a2b43b7f3bad3affc833b786ef
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adipocyte-differentiation-gene-signature-interpretation

## Summary

Interpret gene-expression changes in the MDI-induced 3T3-L1 adipocyte model by running gene set enrichment (GSEA) on a fold-change-ranked gene list against the Gene Ontology, recovering adipogenic signatures such as GO:0050873 (brown fat cell differentiation) under Pparg knockdown.

## When to use

Use when a curated 3T3-L1 adipocyte dataset (e.g. from curatedAdipoArray) has differential-expression results available and the goal is to identify enriched GO terms among genes ranked by fold-change rather than test a single gene list.

## When NOT to use

- Input is raw probe-level intensities that have not been mapped and collapsed to gene symbols — collapse probes first (e.g. with collapseRows) before ranking genes.
- The goal is calling individual differentially expressed genes rather than interpreting a coordinated expression signature via gene set enrichment.
- The data are RNA-seq or ChIP-seq samples (curatedAdipoRNA / curatedAdipoChIP) rather than the curatedAdipoArray microarray SummarizedExperiment.

## Inputs

- Processed 3T3-L1 microarray SummarizedExperiment object from curatedAdipoArray accessed via ExperimentHub
- Differential expression results (gene-level log fold-changes) from the Pparg-knockdown comparison
- Mouse Gene Ontology annotations from Org.Mm.eg.db and GO.db

## Outputs

- Ranked gene list ordered by decreasing log fold-change
- GSEA enrichment result table with GO term IDs, descriptions, enrichment scores and adjusted p-values
- go_enrichment_results.csv containing the five significant GO terms (GO:0050873, GO:0045600, GO:0033993, GO:0019395, GO:0030374)

## How to apply

Load the processed SummarizedExperiment object of the 3T3-L1 microarray data through ExperimentHub from the curatedAdipoArray Bioconductor package. Take the differential expression results and rank all genes by decreasing log fold-change to build the ranked gene list required by GSEA. Run clusterProfiler GSEA against the Gene Ontology using the mouse annotation packages Org.Mm.eg.db and GO.db, testing the standard GO ontologies (BP, CC, MF). Extract significant terms and confirm the adipogenic signature — GO:0050873, GO:0045600, GO:0033993, GO:0019395 and GO:0030374 — is present with its enrichment score. The rationale is that GSEA uses the full ranked list rather than a significance-thresholded gene list, so moderate but coordinated fold-changes in adipogenesis genes still contribute to the enrichment statistic. Write the result table (GO IDs, descriptions, enrichment scores, adjusted p-values) to go_enrichment_results.csv.

## Related tools

- **curatedAdipoArray** (Source of the curated 3T3-L1 microarray SummarizedExperiment data used to derive the fold-change-ranked gene list) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Access point for the clean and processed SummarizedExperiment data objects)
- **clusterProfiler** (Runs gene set enrichment analysis (enrichGO/GSEA) of the ranked gene list against the Gene Ontology)
- **Org.Mm.eg.db** (Mouse gene-to-GO annotation backend used to map gene identifiers to GO terms)
- **GO.db** (Gene Ontology term definitions and ontology structure for the BP/CC/MF tests)

## Examples

```
BiocManager::install("curatedAdipoArray"); library(ExperimentHub); eh <- ExperimentHub(); query(eh, "curatedAdipoArray")
```

## Evaluation signals

- go_enrichment_results.csv contains all five expected GO terms: GO:0050873, GO:0045600, GO:0033993, GO:0019395 and GO:0030374
- Each of the five terms is significant (adjusted p-value below significance threshold) with a reported enrichment score
- The enriched terms are consistent with reported adipogenic biology (brown fat cell differentiation, positive regulation of fat cell differentiation, response to lipid, fatty acid oxidation, nuclear receptor coactivator activity)
- The ranked gene list is sorted by decreasing log fold-change from the differential expression results
- The analysis used the processed (batch-corrected, normalized) SummarizedExperiment from curatedAdipoArray, not raw GEO probe intensities

## Limitations

- Two datasets were excluded for missing probe intensities and probe annotations, so availability of the perturbation condition in curatedAdipoArray must be checked before running the analysis.
- The curated data rely on probe-to-gene-symbol collapsing and cross-study normalization with limma and batch-effect removal with sva; residual processing artifacts could affect enrichment scores.
- Results depend on the perturbation analyzed — the five-term adipogenic signature is reported for Pparg knockdown in differentiating 3T3-L1 pre-adipocytes and may not generalize to other drugs or gene manipulations in the resource.

## Evidence

- [intro] curatedAdipoArray supports downstream enrichment analysis: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [readme] Data access via ExperimentHub: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [readme] Dataset description: "A curated dataset of Microarrays samples. The samples are MDI-induced pre-adipocytes (3T3-L1) at different time points/stage of differentiation"
- [readme] Package installation: "The `curatedAdipoArray` package can be installed from Bioconductor using 
`BiocManager`."
