---
name: term-enrichment-ranking
description: Use when a pre-computed CCI dataset of enriched ligand-receptor int-pairs
  must be linked to Gene Ontology and pathway terms, e.g. to build InterCellar's function-verse
  ranked-term annotation table.
when_to_use_negative:
- Input is raw scRNA-seq data or a full interaction reference database rather than
  pre-computed inferred ligand-receptor interactions; InterCellar's workflow builds
  upon pre-computed interactions uploaded as CCI data.
- Goal is classic statistical over-representation analysis (p-values) on a single
  gene list, rather than annotating int-pairs under the all-components-enriched rule.
- Goal is comparing interactions across multiple datasets/conditions to highlight
  condition-specific communication patterns — that is the multiple-condition comparison
  functionality, not function-verse annotation.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: InterCellar
  role: R/Shiny app whose function-verse performs the term-to-int-pair annotation
    and occurrence ranking of GO/pathway terms
  repo: https://github.com/martaint/InterCellar
- name: biomaRt
  role: queries Gene Ontology term-gene annotations through Ensembl
- name: graphite
  role: queries the pathway databases KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther,
    and PharmGKB for term-gene annotations
- name: InterCellar-reproducibility
  role: repository with supplementary data and code to reproduce InterCellar's analyses,
    including the functional-verse results
  repo: https://github.com/martaint/InterCellar-reproducibility
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:ae3aa3f6b4771ee43ef75869dfbc1011461ae798f13c1292a1d460e04609a8b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - InterCellar
  - InterCellar-reproducibility
  - R
  - biomaRt
  - graphite
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# term-enrichment-ranking

## Summary

Annotates enriched ligand-receptor int-pairs from a CCI dataset with GO and pathway terms using biomaRt (Ensembl) and graphite (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB), then ranks terms by occurrence. Core rule: a term is assigned to an int-pair only when ALL components, including multi-subunit complex members, are enriched by that term.

## When to use

Use when a pre-computed CCI dataset of enriched ligand-receptor int-pairs must be linked to Gene Ontology and pathway terms, e.g. to build InterCellar's function-verse ranked-term annotation table.

## When NOT to use

- Input is raw scRNA-seq data or a full interaction reference database rather than pre-computed inferred ligand-receptor interactions; InterCellar's workflow builds upon pre-computed interactions uploaded as CCI data.
- Goal is classic statistical over-representation analysis (p-values) on a single gene list, rather than annotating int-pairs under the all-components-enriched rule.
- Goal is comparing interactions across multiple datasets/conditions to highlight condition-specific communication patterns — that is the multiple-condition comparison functionality, not function-verse annotation.

## Inputs

- Pre-computed, inferred ligand-receptor interactions uploaded as CCI data (output of InterCellar-supported tools: CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR), filtered to enriched int-pairs
- Component list of each int-pair, including members of multi-subunit ligand/receptor complexes

## Outputs

- Ranked term-to-int-pair annotation table written to functional_annotation_ranked_terms.csv, with each term's occurrence (number of annotated int-pairs)
- Function-verse view of functional terms ranked by occurrence for downstream exploration

## How to apply

Load the table of enriched int-pairs from the cell-cell interaction (CCI) dataset into InterCellar's function-verse, then retrieve term-gene annotations: Gene Ontology terms through Ensembl via the biomaRt R package, and pathway terms for KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, and PharmGKB through the graphite R package. Annotate a functional term to an int-pair only when ALL components of the int-pair — including members of multi-subunit complexes — are enriched by that term; this is an all-component match criterion, not an enrichment p-value cutoff. Rank the resulting functional terms by occurrence, i.e. the number of int-pairs annotated with each term. Write the ranked term-to-int-pair annotation table to functional_annotation_ranked_terms.csv and verify every assignment satisfies the all-components-enriched rule.

## Related tools

- **InterCellar** (R/Shiny app whose function-verse performs the term-to-int-pair annotation and occurrence ranking of GO/pathway terms) — https://github.com/martaint/InterCellar
- **biomaRt** (queries Gene Ontology term-gene annotations through Ensembl)
- **graphite** (queries the pathway databases KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, and PharmGKB for term-gene annotations)
- **InterCellar-reproducibility** (repository with supplementary data and code to reproduce InterCellar's analyses, including the functional-verse results) — https://github.com/martaint/InterCellar-reproducibility

## Examples

```
library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- functional_annotation_ranked_terms.csv exists and contains ranked functional terms with their annotated int-pair counts.
- Every int-pair-term assignment satisfies the all-components-enriched rule — including members of multi-subunit complexes; any assignment missing one un-enriched component is an error.
- Term ranks strictly follow occurrence, i.e. the number of int-pairs annotated with each term.
- Both annotation sources are covered: GO terms from Ensembl/biomaRt and pathway terms from all six graphite-backed databases (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB).
- Every int-pair in the output corresponds to an enriched int-pair present in the uploaded CCI table.

## Limitations

- Terms are ranked by occurrence only; no significance/p-value threshold for the term-to-int-pair annotation itself is defined in this method.
- The all-components rule is stringent: a single un-enriched member of a multi-subunit complex blocks annotation of the whole int-pair.
- InterCellar supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work.

## Evidence

- [intro] function-verse annotation of enriched interactions: "For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [methods] graphite pathway sources: [evidence span withheld — anchor status: not_found]
- [methods] all-components enrichment rule: [evidence span withheld — anchor status: not_found]
- [readme] README functional-annotation statement: [evidence span withheld — anchor status: external_doc]
