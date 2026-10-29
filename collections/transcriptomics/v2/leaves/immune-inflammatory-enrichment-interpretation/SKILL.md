---
name: immune-inflammatory-enrichment-interpretation
description: Use when WGCNA module eigengenes (or DEG lists) from a rare-disease RNA-seq
  experiment correlate with a phenotypic trait and you must translate the module gene
  lists into GO Biological Process terms to confirm immune/infiltration biology or
  nominate targets.
when_to_use_negative:
- Input is raw reads or FASTQ that has not been converted to a count table — the workflow
  requires count tables as the starting point for DEgenes Hunter
- The modules of interest do not correlate with the disease phenotypic trait — enrichment
  of uncorrelated modules will not support the immune/inflammatory phenotype interpretation
- The organism/annotation has no orthologous translation to supported databases —
  the article's functional analysis depends on orthologous annotation; non-model organisms
  without this support will yield empty enrichment
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package that runs the co-expression module detection, module-trait
    correlation, multi-database functional enrichment, and generates the integrated
    HTML report
  repo: bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: Upstream pipeline producing the DEG consensus (DESeq2, NOISeq, limma) whose
    gene universe feeds co-expression and enrichment
  repo: github.com/seoanezonjic/ExpHunterSuite
- name: WGCNA
  role: Detects co-expression modules and eigengenes that are correlated with the
    mutant/wild-type trait before enrichment
- name: R
  role: Runtime environment for the ExpHunter Suite (R/Bioconductor, R version 4 required)
  repo: github.com/seoanezonjic/ExpHunterSuite
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:0fd2fe9b31da0db4836c527519b2c6a127720b436926f2dd4f2007129c6a757c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - WGCNA
  - limma
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# immune-inflammatory-enrichment-interpretation

## Summary

Interprets co-expression modules correlated with a disease phenotype (e.g. mutant animals in the Lafora disease RNA-seq dataset) by running functional enrichment on module gene lists with multiple annotation databases and checking for immune/inflammatory GO Biological Process terms, confirming known biology and surfacing candidate therapeutic targets.

## When to use

Use when WGCNA module eigengenes (or DEG lists) from a rare-disease RNA-seq experiment correlate with a phenotypic trait and you must translate the module gene lists into GO Biological Process terms to confirm immune/infiltration biology or nominate targets.

## When NOT to use

- Input is raw reads or FASTQ that has not been converted to a count table — the workflow requires count tables as the starting point for DEgenes Hunter
- The modules of interest do not correlate with the disease phenotypic trait — enrichment of uncorrelated modules will not support the immune/inflammatory phenotype interpretation
- The organism/annotation has no orthologous translation to supported databases — the article's functional analysis depends on orthologous annotation; non-model organisms without this support will yield empty enrichment

## Inputs

- Lafora disease RNA-seq count matrix and sample metadata from github.com/fmjabato/ExpHunterSuiteExamples
- WGCNA co-expression module gene lists correlated with the mutant/wild-type phenotypic trait (module-trait correlation table)
- DEG list from DEgenes Hunter (DESeq2, NOISeq, limma consensus/vote-threshold calls)
- Annotation databases (GO Biological Process, reactome.db) accessed through ExpHunter Suite

## Outputs

- module_trait_correlations.csv (module-eigengene vs. mutant/wild-type correlation table)
- enrichment_results.csv (GO Biological Process / Reactome terms per module with p-values)
- Integrated ExpHunter Suite HTML results report

## How to apply

After DEgenes Hunter differential expression (DESeq2, NOISeq, limma combined by consensus/vote threshold) and WGCNA module detection, correlate module eigengenes with the mutant/wild-type phenotypic trait to build the module-trait correlation table. Run functional enrichment on the gene lists of modules correlated with the disease trait, combining multiple annotation databases (GO, Reactome via reactome.db) and focusing on GO Biological Process terms, as implemented in the ExpHunter Suite HTML report. Interpret enrichment against prior disease knowledge: for Lafora disease, modules correlated with mutants are expected to be enriched for immune system and inflammatory processes consistent with immune cell infiltration. The vote-threshold filter step (optimum threshold 2 by F-measure on spike-in data) controls which DEGs feed downstream co-expression and enrichment, but the authors caution this optimum 'may differ for other organisms, tissues and experimental designs'. Treat enriched terms as hypotheses: the article notes the biological findings 'are currently being studied further'.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package that runs the co-expression module detection, module-trait correlation, multi-database functional enrichment, and generates the integrated HTML report) — bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (Upstream pipeline producing the DEG consensus (DESeq2, NOISeq, limma) whose gene universe feeds co-expression and enrichment) — github.com/seoanezonjic/ExpHunterSuite
- **WGCNA** (Detects co-expression modules and eigengenes that are correlated with the mutant/wild-type trait before enrichment)
- **R** (Runtime environment for the ExpHunter Suite (R/Bioconductor, R version 4 required)) — github.com/seoanezonjic/ExpHunterSuite

## Examples

```
bash launch_DEgenesHunter.sh   # in the Lafora (or PMM2) example folder of fmjabato/ExpHunterSuiteExamples, after installing: Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'
```

## Evaluation signals

- The ExpHunter Suite HTML report file exists after launching launch_DEgenesHunter.sh (README: 'A new HTML file with the results report must has been generated')
- module_trait_correlations.csv contains module-eigengene correlation fields for the mutant/wild-type trait
- enrichment_results.csv contains GO Biological Process terms, and modules correlated with mutant animals include immune system / inflammatory process terms
- The DEG calls used as input come from the consensus vote-threshold step (minimum vote threshold; optimum was 2 by F-measure on spike-in data), not a single-method call
- Enriched immune/inflammatory terms are consistent with the paper's own conclusion that the analysis 'confirms existing knowledge related to immune cell infiltration'

## Limitations

- The optimal DEG vote threshold may not generalize: 'this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule'
- The biological findings are still hypotheses: 'The biological findings shown here are currently being studied further'
- Method behaviour is dataset-dependent: 'some widely-used expression analysis methods can behave quite differently depending on the properties of the data' (e.g. DESeq2 most conservative for Lafora but least conservative for PMM2-CDG)
- reactome.db is >400 Mb and can exceed the default 60-second R download timeout during installation

## Evidence

- [abstract] Application to rare-disease model datasets confirms existing knowledge related to immune cell infiltration and suggest novel hypotheses including the role of calcium channels: "Application to rare-disease model datasets confirms existing knowledge related to immune cell infiltration and suggest novel hypotheses including the role of calcium channels"
- [intro] Co-expression analysis, which searches for groups of co-expressed genes (CEGs) that correlate with phenotypic data11, is also often overlooked in RNA-seq data analysis: "Co-expression analysis, which searches for groups of co-expressed genes (CEGs) that correlate with phenotypic data11, is also often overlooked in RNA-seq data analysis"
- [intro] Fewer still combine multiple annotation databases and custom term sets: "Fewer still combine multiple annotation databases and custom term sets"
- [discussion] for the spike-in data the optimum threshold was 2 in terms of F-measure: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [readme] Protocol to undertake the key steps in the analysis of transcriptomics experiments, including differentially expressed gene detection, a co-expression analysis and the functional enrichment.: [evidence span withheld — anchor status: external_doc]
