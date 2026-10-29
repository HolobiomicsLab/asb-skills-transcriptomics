---
name: r-bioconductor-expression-reporting
description: Use when you have a prevalent DEG results table from a multi-method differential
  expression vote and need functional enrichment across multiple annotation databases
  plus specific gene log2FC extractions.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) rather than a DEG table — first run the clean&map
  / count table steps described in the ExpHunterSuiteExamples README
- The research question requires co-expression module detection or DEG method combination
  rather than functional enrichment of an already-derived gene list
- The organism lacks annotation and orthologous translation setup has not been configured
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose functional Hunter enrichment module runs the
    enrichment and report generation on the DEG list
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: ExpHunterSuite
  role: GitHub installation source for the DEgenes Hunter command line scripts (installed
    via install_DEgenes_hunter into a folder added to PATH)
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: R
  role: Runtime environment in which the enrichment module is executed modularly
- name: Bioconductor
  role: Distribution channel for the ExpHunterSuite workflow package, including the
    hunter vignettes
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:f07961e7ce34a33dbab3b7d945ff69ea3f4c1d49725a6907c9e97c9ff7096b38
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - ExpHunter Suite
  - ExpHunterSuite
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# r-bioconductor-expression-reporting

## Summary

Run the ExpHunter Suite's functional Hunter enrichment module in R/Bioconductor on a consensus DEG table (e.g. PMM2-CDG DEGs with a minimum vote threshold of three) and write the enrichment results plus gene-specific logFC records for downstream reporting and validation.

## When to use

Use when you have a prevalent DEG results table from a multi-method differential expression vote and need functional enrichment across multiple annotation databases plus specific gene log2FC extractions.

## When NOT to use

- Input is raw sequencing data (FASTQ) rather than a DEG table — first run the clean&map / count table steps described in the ExpHunterSuiteExamples README
- The research question requires co-expression module detection or DEG method combination rather than functional enrichment of an already-derived gene list
- The organism lacks annotation and orthologous translation setup has not been configured

## Inputs

- Prevalent PMM2-CDG DEG results table from upstream differential expression task (task_006, vote threshold 3)
- Count tables produced by DEgenes Hunter expression analysis (as run for the Lafora / PMM2 experiments)
- Annotation databases and custom term sets supported by ExpHunterSuite

## Outputs

- functional_enrichment_results.tsv (filtered for basement membrane / extracellular matrix terms)
- Companion record of CACNA1H log2 fold change from the DEG results table

## How to apply

Load the prevalent DEG results table produced by the upstream differential expression vote (task_006, minimum vote threshold of three) into R. Run the functional Hunter enrichment module of ExpHunterSuite, which converts the DEG gene list into biological knowledge by combining multiple annotation databases and custom term sets. Filter the enrichment table for basement membrane and extracellular matrix related terms, then extract gene-specific records such as the CACNA1H log2 fold change directly from the DEG table. Write the enrichment results to functional_enrichment_results.tsv and the gene record to a companion file, validating that the enrichment table contains basement membrane / extracellular matrix substrings and that the CACNA1H logFC value is negative. Note the article's caution that incorrect fold change estimation has downstream implications for logFC-based analyses such as GSEA.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose functional Hunter enrichment module runs the enrichment and report generation on the DEG list) — https://bioconductor.org/packages/ExpHunterSuite
- **ExpHunterSuite** (GitHub installation source for the DEgenes Hunter command line scripts (installed via install_DEgenes_hunter into a folder added to PATH)) — https://github.com/seoanezonjic/ExpHunterSuite
- **R** (Runtime environment in which the enrichment module is executed modularly)
- **Bioconductor** (Distribution channel for the ExpHunterSuite workflow package, including the hunter vignettes) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e "options(timeout=1000); devtools::install_github('seoanezonjic/ExpHunterSuite', dependencies=TRUE)"
```

## Evaluation signals

- functional_enrichment_results.tsv contains basement membrane / extracellular matrix related terms (contains_substring check)
- CACNA1H logFC record is negative (value_in_range check)
- Enrichment was run on the prevalent DEG list defined by the minimum vote threshold of three
- Output files exist at the expected paths (functional_enrichment_results.tsv plus companion gene record)
- The reported HTML results report is generated, as expected when following the ExpHunterSuiteExamples launch/get_results workflow

## Limitations

- The minimum vote threshold cut-off may not generalize: the paper notes 'this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule'
- Incorrect fold change estimation could lead to poor qPCR target selection and affects downstream logFC analyses such as GSEA
- The biological findings (e.g. calcium channel involvement in PMM2-CDG) are still under investigation, not fully resolved

## Evidence

- [abstract] ExpHunter Suite is implemented as an R/Bioconductor package available from https://bioconductor.org/packages/ExpHunterSuite: "ExpHunter Suite is implemented as an R/Bioconductor package available from https://bioconductor.org/packages/ExpHunterSuite"
- [intro] Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge: "Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge"
- [discussion] we would suggest the use of a minimum vote threshold cut-off where possible: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [readme] Rscript -e "ExpHunterSuite::install_DEgenes_hunter('install_folder')": "Rscript -e "ExpHunterSuite::install_DEgenes_hunter('install_folder')""
- [readme] Execute file launch_DEgenesHunter.sh stored in specific folder.: [evidence span withheld — anchor status: external_doc]
