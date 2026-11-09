---
name: gene-ontology-term-overrepresentation
description: Use when you have a gene list from differential expression (e.g. consensus
  DEGs selected by a minimum vote threshold) and need overrepresentation of biological
  processes such as basement membrane or ECM terms.
when_to_use_negative:
- Input is raw counts or a table of counts rather than an already-selected DEG list
  — enrichment should be run on detected DEGs, not on the full expression matrix
- You need pathway-level mapping of many genes rather than overrepresentation of a
  shortlist; the co-expression module detection functionality of ExpHunterSuite is
  the complementary choice in the paper
- The DEG list came from logFC-based ranking whose fold changes are unreliable — the
  paper warns incorrect fold change estimation has 'important implications for downstream
  analyses that use logFC, such as the popular GSEA method'
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose functional Hunter enrichment module converts
    the DEG gene list into overrepresented biological processes combining multiple
    annotation databases and custom term sets
  repo: bioconductor.org/packages/ExpHunterSuite
- name: R
  role: runtime environment in which ExpHunterSuite is run modularly
- name: Bioconductor
  role: distribution channel and dependency ecosystem (e.g. reactome.db annotation
    database) for the enrichment module
  repo: bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:aacf6dad413303ad41426653413a0b2f72f86e325b0da067add3de477a6456b9
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-ontology-term-overrepresentation

## Summary

Run the functional enrichment module of ExpHunterSuite in R/Bioconductor on a differentially expressed gene (DEG) list to convert it into overrepresented biological processes, combining multiple annotation databases and custom term sets. In this workflow it was used to identify basement membrane / extracellular matrix processes among prevalent PMM2-CDG DEGs.

## When to use

Use when you have a gene list from differential expression (e.g. consensus DEGs selected by a minimum vote threshold) and need overrepresentation of biological processes such as basement membrane or ECM terms.

## When NOT to use

- Input is raw counts or a table of counts rather than an already-selected DEG list — enrichment should be run on detected DEGs, not on the full expression matrix
- You need pathway-level mapping of many genes rather than overrepresentation of a shortlist; the co-expression module detection functionality of ExpHunterSuite is the complementary choice in the paper
- The DEG list came from logFC-based ranking whose fold changes are unreliable — the paper warns incorrect fold change estimation has 'important implications for downstream analyses that use logFC, such as the popular GSEA method'

## Inputs

- DEG gene list / DEG results table from upstream differential expression analysis (e.g. prevalent PMM2-CDG DEGs from a minimum vote threshold cut-off)
- Annotation databases available to ExpHunterSuite (e.g. reactome.db) and custom term sets

## Outputs

- functional_enrichment_results.tsv (overrepresented biological process terms, filtered for basement membrane / extracellular matrix terms)
- companion record of CACNA1H log2 fold change from the DEG results table

## How to apply

Install ExpHunterSuite as an R/Bioconductor package (raising the R timeout, e.g. options(timeout=1000), since reactome.db exceeds 400 Mb and the default 60 s timeout). Load the DEG results table produced by the upstream differential expression task and pass the DEG gene list to the functional Hunter enrichment module, which combines multiple annotation databases (including reactome.db) and custom term sets — a capability the paper highlights because 'fewer still combine multiple annotation databases and custom term sets'. Filter the enrichment results for terms of interest (here basement membrane and extracellular matrix / collagen related processes) and write them to functional_enrichment_results.tsv. In the PMM2-CDG application this surfaced basement membrane formation/composition alteration, driven by ECM and collagen processes, alongside the CACNA1H underexpression (negative log2FC) supporting the calcium-channel hypothesis. For non-model organisms, annotation is provided through orthologous translation before enrichment.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose functional Hunter enrichment module converts the DEG gene list into overrepresented biological processes combining multiple annotation databases and custom term sets) — bioconductor.org/packages/ExpHunterSuite
- **R** (runtime environment in which ExpHunterSuite is run modularly)
- **Bioconductor** (distribution channel and dependency ecosystem (e.g. reactome.db annotation database) for the enrichment module) — bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'
```

## Evaluation signals

- Validation per the workflow: confirm the enrichment table contains basement membrane / extracellular matrix related terms (contains_substring check)
- Confirm the extracted CACNA1H log2FC value is negative (value_in_range check), consistent with the calcium-channel underexpression finding
- Enrichment output is a TSV named functional_enrichment_results.tsv with term rows derived from the input DEG list
- Enrichment run must use multiple annotation databases/custom term sets, not a single database, per the ExpHunterSuite design objective
- Reproduction of the published PMM2-CDG finding: functional enrichment points to alteration of basement membrane formation and composition

## Limitations

- Biological findings from the paper are 'currently being studied further', so enrichment hits are hypothesis-generating, not resolved conclusions
- Incorrect fold change estimation upstream can lead to poor target selection and affect downstream logFC analyses such as GSEA
- The paper emphasizes 'the need for the stable implementations of proposed methodologies and workflows'

## Evidence

- [intro] Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge: "Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge"
- [intro] Fewer still combine multiple annotation databases and custom term sets: "Fewer still combine multiple annotation databases and custom term sets"
- [abstract] Application to rare-disease model datasets confirms existing knowledge related to immune cell infiltration and suggest novel hypotheses including the role of calcium channels: "Application to rare-disease model datasets confirms existing knowledge related to immune cell infiltration and suggest novel hypotheses including the role of calcium channels"
- [readme] this scripts uses Autoflow, be sure to has it installed in your system: [evidence span withheld — anchor status: external_doc]
