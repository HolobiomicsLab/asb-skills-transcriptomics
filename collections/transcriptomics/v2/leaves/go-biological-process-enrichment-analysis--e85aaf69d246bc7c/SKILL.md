---
name: go-biological-process-enrichment-analysis
description: Use when you have a list of differentially expressed genes or WGCNA co-expression
  module members and need to identify over-represented GO Biological Process terms
  linking gene sets to phenotypic traits.
when_to_use_negative:
- Input is a raw count matrix with no upstream DEG or co-expression gene lists — enrichment
  requires gene lists as input
- The goal is DEG detection itself rather than biological interpretation of an already-computed
  gene list
- Only a single annotation database is desired and combining multiple databases/custom
  term sets is not required
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: ExpHunter Suite
  role: Runs functional enrichment combining multiple annotation databases and custom
    term sets on DEG and co-expression module gene lists
  repo: bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: Produces the consensus DEG gene lists that serve as enrichment input
  repo: github.com/seoanezonjic/ExpHunterSuite
- name: R/Bioconductor
  role: Environment in which the enrichment workflow is executed
  repo: bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:e98aaae53ea3d1c4c9b454b311f8004d687cf88c39ecbdaa128189045722076d
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# GO Biological Process enrichment analysis

## Summary

Functional enrichment of gene lists (DEGs and co-expression modules) against GO Biological Process terms, combining multiple annotation databases as implemented in ExpHunter Suite, to convert gene lists into biological knowledge about processes such as immune/inflammatory response in Lafora disease modules.

## When to use

Use when you have a list of differentially expressed genes or WGCNA co-expression module members and need to identify over-represented GO Biological Process terms linking gene sets to phenotypic traits.

## When NOT to use

- Input is a raw count matrix with no upstream DEG or co-expression gene lists — enrichment requires gene lists as input
- The goal is DEG detection itself rather than biological interpretation of an already-computed gene list
- Only a single annotation database is desired and combining multiple databases/custom term sets is not required

## Inputs

- List of differentially expressed genes from DEgenes Hunter (DESeq2, NOISeq, limma consensus)
- Gene lists of WGCNA co-expression modules correlated with a phenotypic trait (e.g. mutant vs wild-type)
- Gene identifier annotations; orthologous translation tables for non-model organisms

## Outputs

- enrichment_results.csv with GO Biological Process terms per gene list/module
- module_trait_correlations.csv linking module eigen-genes to phenotypic traits
- Integrated ExpHunter Suite HTML report including enrichment sections

## How to apply

Run functional enrichment on the gene lists produced by upstream steps — the consensus DEG list from DEgenes Hunter (DESeq2, NOISeq, limma combined by vote threshold) and the WGCNA co-expression modules correlated with the mutant/wild-type phenotypic trait. Combine results from multiple annotation databases and custom term sets rather than a single source, focusing on GO Biological Process terms. Restrict interpretation to modules with strong module-eigen-gene correlation with the phenotype of interest (e.g. mutant animals in the Lafora disease RNA-seq dataset). ExpHunter Suite can also translate identifiers via orthologs to support non-model organisms. Export the enrichment results table and inspect whether enriched BP terms reproduce known biology (e.g. immune system and inflammatory processes in Lafora disease) as a sanity check.

## Related tools

- **ExpHunter Suite** (Runs functional enrichment combining multiple annotation databases and custom term sets on DEG and co-expression module gene lists) — bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (Produces the consensus DEG gene lists that serve as enrichment input) — github.com/seoanezonjic/ExpHunterSuite
- **R/Bioconductor** (Environment in which the enrichment workflow is executed) — bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e 'devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'; launch_DEgenesHunter.sh
```

## Evaluation signals

- enrichment_results.csv exists and contains GO Biological Process terms for each submitted module/DEG list
- Enriched immune/inflammatory GO BP terms appear specifically in modules with high correlation to mutant animals in the Lafora disease dataset
- The integrated ExpHunter Suite HTML report is generated and includes the enrichment section
- Gene identifiers in module lists are successfully mapped to annotation terms (orthologous translation applied for non-model organisms)
- Enrichment results reproduce known biology of the disorder (immune cell infiltration) as reported in the paper

## Limitations

- Few tools combine multiple annotation databases and custom term sets, so results depend on database choice
- Enrichment downstream of DEG results is affected by logFC estimation errors, which have implications for downstream analyses such as GSEA
- Optimal analysis parameters (e.g. vote thresholds upstream) may not generalize across organisms, tissues and experimental designs

## Evidence

- [abstract] functional enrichment combining multiple annotation databases: "Detected DEGs serve as input for functional enrichment analysis, in which lists of genes are converted into biological knowledge"
- [intro] enrichment input from DEGs and modules: "Fewer still combine multiple annotation databases and custom term sets"
- [intro] co-expression enrichment in Lafora disease: "Co-expression analysis, which searches for groups of co-expressed genes (CEGs) that correlate with phenotypic data11"
- [other] validation of enrichment output: [evidence span withheld — anchor status: external_doc]
