---
name: module-eigengene-trait-correlation-analysis
description: Use when RNA-seq count data with sample-level phenotypic metadata (e.g.,
  mutant/wild-type labels from the Lafora disease dataset) is available and you want
  to find groups of co-expressed genes whose expression patterns correlate with the
  phenotype.
when_to_use_negative:
- Input is already a module-trait correlation or enrichment table rather than raw
  counts plus phenotype metadata
- The experiment lacks sample-level phenotypic data to correlate eigen-genes against
  (no mutant/wild-type or other trait labels)
- The goal is DEG detection alone without co-expression analysis, since WGCNA module
  detection is a distinct, optional step of the pipeline
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package performing co-expression module detection, module-trait
    correlation, functional enrichment and integrated HTML report generation
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: Upstream pipeline that combines DESeq2, NOISeq and limma results by consensus/vote
    threshold to supply DEGs feeding co-expression analysis
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: R/Bioconductor
  role: Runtime environment for ExpHunter Suite (R version 4 required per README)
  repo: https://github.com/seoanezonjic/ExpHunterSuite
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:42779a68786093b30d4b1d1325c3a3dacbe0842e3d80e64633057cb63aafc53d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - R/Bioconductor
  - limma
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# module-eigengene-trait-correlation-analysis

## Summary

Detects co-expression modules with WGCNA and correlates module eigen-genes with phenotypic traits (e.g., mutant vs wild-type in the Lafora disease RNA-seq dataset) to identify gene modules driving a disease phenotype, then links them to biology via functional enrichment. This step is part of the ExpHunter Suite methodology for rare glucide metabolism disorders.

## When to use

Use when RNA-seq count data with sample-level phenotypic metadata (e.g., mutant/wild-type labels from the Lafora disease dataset) is available and you want to find groups of co-expressed genes whose expression patterns correlate with the phenotype.

## When NOT to use

- Input is already a module-trait correlation or enrichment table rather than raw counts plus phenotype metadata
- The experiment lacks sample-level phenotypic data to correlate eigen-genes against (no mutant/wild-type or other trait labels)
- The goal is DEG detection alone without co-expression analysis, since WGCNA module detection is a distinct, optional step of the pipeline

## Inputs

- RNA-seq count matrix (table of counts) from the Lafora disease experiment in github.com/fmjabato/ExpHunterSuiteExamples
- Sample metadata with phenotypic trait labels (mutant vs wild-type)
- Differentially expressed gene results from the DEgenes Hunter pipeline (DESeq2, NOISeq, limma combined by consensus/vote threshold)

## Outputs

- module_trait_correlations.csv (module-eigen-gene vs trait correlation table)
- enrichment_results.csv (functional enrichment of module gene lists, GO Biological Process terms)
- Integrated ExpHunter Suite HTML report

## How to apply

After obtaining DEGs via the DEgenes Hunter pipeline (DESeq2, NOISeq and limma combined by consensus/vote threshold), run co-expression module detection with WGCNA on the count matrix. Correlate each module eigen-gene with the phenotypic trait (mutant/wild-type) to produce the module-trait correlation table; the paper notes this step is 'a key component of the ExpHunter Suite, and a step overlooked in many protocols'. Then run functional enrichment on the module gene lists, combining multiple annotation databases with focus on GO Biological Process terms. Finally, generate the integrated ExpHunter Suite HTML report and export module_trait_correlations.csv and enrichment_results.csv. Validation: check the report exists, the correlation table contains module-eigen-gene correlation fields, and modules correlated with mutants show immune/inflammatory GO BP terms.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package performing co-expression module detection, module-trait correlation, functional enrichment and integrated HTML report generation) — https://bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (Upstream pipeline that combines DESeq2, NOISeq and limma results by consensus/vote threshold to supply DEGs feeding co-expression analysis) — https://github.com/seoanezonjic/ExpHunterSuite
- **R/Bioconductor** (Runtime environment for ExpHunter Suite (R version 4 required per README)) — https://github.com/seoanezonjic/ExpHunterSuite

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'; bash launch_DEgenesHunter.sh
```

## Evaluation signals

- The integrated ExpHunter Suite HTML report file exists after running launch_DEgenesHunter.sh (README: 'A new HTML file with the results report must has been generated')
- module_trait_correlations.csv contains module-eigen-gene correlation fields (columns mapping each WGCNA module to the mutant/wild-type trait)
- enrichment_results.csv for modules highly correlated with mutant animals contains immune system / inflammatory GO Biological Process terms
- Module eigen-gene correlation values with the phenotypic trait are computed for every detected module, matching the paper's workflow step 3
- Replication requires configuring launch.sh/get_results.sh with the environment load and results folder as described in the README

## Limitations

- Optimal vote threshold for the upstream DEG step may not generalize: 'this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule'
- The biological findings (immune cell infiltration, calcium channel hypotheses) are 'currently being studied further' and not fully resolved
- Co-expression analysis is often overlooked but depends on upstream DEG method choice, since widely-used methods 'can behave quite differently depending on the properties of the data'
- reactome.db (>400 Mb) can exceed R's 60-second download timeout during installation, requiring options(timeout=1000) or source installation

## Evidence

- [abstract] co-expression module detection and result integration via data-fusion: "We present methodology for the comprehensive analysis of expression data, including co-expression module detection and result integration via data-fusion"
- [intro] co-expression analysis correlates groups of genes with phenotypic data: "Co-expression analysis, which searches for groups of co-expressed genes (CEGs) that correlate with phenotypic data11, is also often overlooked in RNA-seq data analysis"
- [discussion] key component of ExpHunter Suite: "A key component of the ExpHunter Suite, and a step overlooked in many protocols"
- [readme] README replication instructions for Lafora/PMM2: [evidence span withheld — anchor status: external_doc]
- [readme] README install command: [evidence span withheld — anchor status: external_doc]
