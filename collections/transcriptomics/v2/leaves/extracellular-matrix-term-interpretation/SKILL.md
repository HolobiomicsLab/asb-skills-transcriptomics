---
name: extracellular-matrix-term-interpretation
description: Use when functional enrichment results from a DEG list (e.g. PMM2-CDG
  DEGs selected by a minimum vote threshold of three) must be filtered and interpreted
  for basement membrane, extracellular matrix, or collagen-related biological process
  terms.
when_to_use_negative:
- The input is raw RNA-seq data or count tables rather than an already-computed DEG
  results table — use the upstream DEgenes Hunter differential expression pipeline
  instead
- The biological question concerns co-expression modules or DEG method consensus voting
  rather than interpreting overrepresented biological process terms
- The organism is non-model and requires orthologous translation before functional
  annotation is meaningful for ECM/basement membrane terms
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ExpHunter Suite
  role: Runs the functional Hunter enrichment module that combines multiple annotation
    databases and custom term sets on the PMM2-CDG DEG gene list
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: R
  role: Runtime environment in which the enrichment module is run and from which the
    CACNA1H logFC record is extracted and written to TSV
- name: Bioconductor
  role: Package ecosystem hosting ExpHunterSuite used for the functional enrichment
    step
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:47ef6be236427026127c4e860107fad8fce32ff80b9d5cabf4e5a5d86b2b8b13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - Bioconductor
  - ExpHunter Suite
  - ExpHunterSuite
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# extracellular-matrix-term-interpretation

## Summary

Interprets functional enrichment results from the ExpHunterSuite functional Hunter module applied to prevalent PMM2-CDG differentially expressed genes, filtering for basement membrane and extracellular matrix (ECM)/collagen terms to support the finding that basement membrane formation and composition are altered in PMM2-CDG.

## When to use

Use when functional enrichment results from a DEG list (e.g. PMM2-CDG DEGs selected by a minimum vote threshold of three) must be filtered and interpreted for basement membrane, extracellular matrix, or collagen-related biological process terms.

## When NOT to use

- The input is raw RNA-seq data or count tables rather than an already-computed DEG results table — use the upstream DEgenes Hunter differential expression pipeline instead
- The biological question concerns co-expression modules or DEG method consensus voting rather than interpreting overrepresented biological process terms
- The organism is non-model and requires orthologous translation before functional annotation is meaningful for ECM/basement membrane terms

## Inputs

- Prevalent PMM2-CDG DEG results table from upstream differential expression analysis (DEGs identified with minimum vote threshold of three)
- ExpHunterSuite functional Hunter enrichment output table
- DEG gene list for PMM2-CDG

## Outputs

- functional_enrichment_results.tsv containing basement membrane / extracellular matrix enrichment terms
- Companion record with the CACNA1H log2 fold change value from the DEG results table

## How to apply

Load the prevalent PMM2-CDG DEG results table produced by the upstream differential expression task into R and run the functional Hunter enrichment module of ExpHunterSuite, which combines multiple annotation databases and custom term sets rather than a single database. From the resulting enrichment table, filter for basement membrane and extracellular matrix related terms (e.g. collagen-related processes) to determine which biological processes are overrepresented. In parallel, extract the CACNA1H log2 fold change record from the DEG table to connect the ECM signature to the calcium channel hypothesis reported for PMM2-CDG. Write the enrichment results to functional_enrichment_results.tsv with a companion CACNA1H logFC record, and validate that the enrichment table contains basement membrane/ECM terms and that the CACNA1H logFC is negative.

## Related tools

- **ExpHunter Suite** (Runs the functional Hunter enrichment module that combines multiple annotation databases and custom term sets on the PMM2-CDG DEG gene list) — https://github.com/seoanezonjic/ExpHunterSuite
- **R** (Runtime environment in which the enrichment module is run and from which the CACNA1H logFC record is extracted and written to TSV)
- **Bioconductor** (Package ecosystem hosting ExpHunterSuite used for the functional enrichment step) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'
```

## Evaluation signals

- The enrichment table contains at least one term matching basement membrane, extracellular matrix, or collagen substrings (contains_substring check)
- The CACNA1H log2 fold change value extracted from the DEG results table is negative (value_in_range check)
- functional_enrichment_results.tsv is produced and parses as a valid TSV with enrichment term columns populated
- The DEG input list corresponds to genes passing the minimum vote threshold of three from the upstream consensus voting step
- Interpreted ECM/basement membrane terms are consistent with the reported finding that basement membrane formation and composition are altered in PMM2-CDG

## Limitations

- The biological findings (including ECM/basement membrane alteration and calcium channel involvement) are still being studied further and are not fully resolved
- The vote threshold used to define prevalent DEGs is dataset-dependent; the authors hesitate to extrapolate optimal thresholds to other organisms, tissues, or experimental designs
- Incorrect fold change estimation in the upstream DEG step could propagate to downstream logFC-based analyses such as GSEA and lead to poor target selection

## Evidence

- [abstract] Application to rare-disease model datasets confirms existing knowledge related to immune cell infiltration and suggest novel hypotheses including the role of calcium channels: "Application to rare-disease model datasets confirms existing knowledge related to immune cell infiltration and suggest novel hypotheses including the role of calcium channels"
- [intro] Fewer still combine multiple annotation databases and custom term sets: "Fewer still combine multiple annotation databases and custom term sets"
- [intro] Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge: "Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge"
- [readme] Protocol to undertake the key steps in the analysis of transcriptomics experiments, including differentially expressed gene detection, a co-expression analysis and the functional enrichment.: [evidence span withheld — anchor status: external_doc]
