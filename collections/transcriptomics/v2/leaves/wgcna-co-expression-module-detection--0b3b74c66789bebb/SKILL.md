---
name: wgcna-co-expression-module-detection
description: Use when you have an RNA-seq table of counts plus sample phenotype metadata
  (e.g., mutant/wild-type Lafora disease samples) and want to find co-expressed gene
  groups (CEGs) that correlate with phenotypic data.
when_to_use_negative:
- Input is raw reads or FASTQ rather than a processed table of counts — the count
  matrix must already be produced by upstream RNA-seq initial data analysis.
- The analysis goal is only differential expression detection without phenotypic correlation
  of gene groups — use the DEgenes Hunter DEG combination workflow instead.
- No sample phenotype/trait metadata exists to correlate modules against, since the
  method searches for CEGs that correlate with phenotypic data.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose modular co-expression analysis step performs
    co-expression module detection and eigen-gene–trait correlation plus integrated
    reporting
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: ExpHunter Suite pipeline producing the DEG context and handling the count-table
    input preceding co-expression analysis; run via launch_DEgenesHunter.sh in the
    Lafora/PMM2 examples
  repo: https://github.com/seoanezonjic/ExpHunterSuite
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:a680b4508b17e8d7dc1cfa8583e27122a2e3e33e095fb9e66d56d4cdddd6cbc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - limma
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# wgcna-co-expression-module-detection

## Summary

Detect co-expression modules in RNA-seq data via WGCNA within the ExpHunter Suite and correlate module eigen-genes with a phenotypic trait (e.g., mutant vs wild-type in the Lafora disease dataset) to link gene networks to phenotype, followed by functional enrichment of module gene lists.

## When to use

Use when you have an RNA-seq table of counts plus sample phenotype metadata (e.g., mutant/wild-type Lafora disease samples) and want to find co-expressed gene groups (CEGs) that correlate with phenotypic data.

## When NOT to use

- Input is raw reads or FASTQ rather than a processed table of counts — the count matrix must already be produced by upstream RNA-seq initial data analysis.
- The analysis goal is only differential expression detection without phenotypic correlation of gene groups — use the DEgenes Hunter DEG combination workflow instead.
- No sample phenotype/trait metadata exists to correlate modules against, since the method searches for CEGs that correlate with phenotypic data.

## Inputs

- RNA-seq table of counts (count matrix) from the Lafora disease (or PMM2-CDG) experiment in github.com/fmjabato/ExpHunterSuiteExamples
- Sample metadata with phenotypic trait labels (mutant vs wild-type)
- R environment with ExpHunterSuite installed (R version 4, devtools::install_github from seoanezonjic/ExpHunterSuite)

## Outputs

- Co-expression modules (groups of co-expressed genes, CEGs)
- Module eigen-gene vs phenotypic trait correlation table (module_trait_correlations.csv)
- Functional enrichment results per module (enrichment_results.csv, GO BP terms)
- Integrated ExpHunter Suite HTML results report

## How to apply

Run co-expression module detection with WGCNA as a modular step of the ExpHunter Suite on the table of counts produced by upstream RNA-seq analysis (DEgenes Hunter accepts the count matrix plus sample metadata from the ExpHunterSuiteExamples repository). Correlate the resulting module eigen-genes with the phenotypic trait vector (mutant vs wild-type) to produce a module-trait correlation table (module_trait_correlations.csv). For modules highly correlated with the phenotype, run functional enrichment on their gene lists, combining multiple annotation databases with a focus on GO Biological Process terms. Rationale: the paper positions co-expression analysis as a key ExpHunter Suite component often overlooked in RNA-seq protocols, and applying it to rare-disease model datasets confirmed immune cell infiltration knowledge and suggested novel hypotheses such as calcium channel roles in PMM2-CDG.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose modular co-expression analysis step performs co-expression module detection and eigen-gene–trait correlation plus integrated reporting) — https://bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (ExpHunter Suite pipeline producing the DEG context and handling the count-table input preceding co-expression analysis; run via launch_DEgenesHunter.sh in the Lafora/PMM2 examples) — https://github.com/seoanezonjic/ExpHunterSuite

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'; bash launch_DEgenesHunter.sh
```

## Evaluation signals

- The ExpHunter Suite HTML results report file has been generated after execution (README: 'A new HTML file with the results report must has been generated').
- The module-trait correlation table (module_trait_correlations.csv) contains module eigen-gene correlation fields.
- Modules highly correlated with mutant animals show enrichment for immune system and inflammatory GO Biological Process terms.
- Installability check: R environment configured per seoanezonjic/ExpHunterSuite instructions with R version 4 and dependencies=TRUE.
- In the Lafora disease experiment, cross-check DEG method behaviour (DESeq2 was the most conservative method) to contextualise which genes enter module detection.

## Limitations

- The biological findings (e.g., immune cell infiltration, calcium channel hypotheses) are 'currently being studied further' and not fully resolved.
- Co-expression analysis 'is also often overlooked in RNA-seq data analysis', so downstream interpretation of module-trait correlations may lack established benchmarks.
- Replicating the Lafora/PMM2 experiments requires a specifically configured R environment; synthetic experiments additionally require Autoflow installed on the system.

## Evidence

- [intro] Co-expression module detection on RNA-seq counts correlated with phenotype: "Co-expression analysis, which searches for groups of co-expressed genes (CEGs) that correlate with phenotypic data"
- [abstract] Co-expression module detection and data-fusion integration in ExpHunter Suite: "including co-expression module detection and result integration via data-fusion"
- [discussion] Co-expression as a key, often-overlooked ExpHunter Suite component: "A key component of the ExpHunter Suite, and a step overlooked in many protocols"
- [readme] Replication of Lafora/PMM2 experiments via launch script: [evidence span withheld — anchor status: external_doc]
- [readme] HTML results report generated after pipeline execution: [evidence span withheld — anchor status: external_doc]
