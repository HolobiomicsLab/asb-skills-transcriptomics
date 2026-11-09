---
name: int-pair-functional-annotation
description: Use when a pre-computed CCI dataset (e.g. from CellChat, CellPhoneDB,
  ICELLNET, SingleCellSignalR) has been loaded into InterCellar's function-verse and
  enriched int-pairs need GO/pathway functional annotation.
when_to_use_negative:
- Input is raw scRNA-seq expression data rather than pre-computed, inferred ligand-receptor
  interactions — InterCellar's workflow builds on pre-computed CCI data
- The CCI output comes from a tool outside InterCellar's supported set (CellChat,
  CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is
  planned future work
- You need condition-specific int-pair comparison or module-level data-driven analysis
  rather than GO/pathway annotation of individual int-pairs
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: R/Shiny platform hosting the function-verse where int-pairs are annotated
    and ranked by occurrence
  repo: https://github.com/martaint/InterCellar-reproducibility
- name: biomaRt
  role: Queries Gene Ontology through Ensembl to retrieve GO term-gene annotations
    for int-pair components
- name: graphite
  role: Queries pathway databases (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther,
    PharmGKB) to retrieve pathway term-gene annotations
- name: R
  role: Implementation language of InterCellar (Bioconductor package); two lines of
    code install and launch the app
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:0a30305f7cc8eaa2cf2b86205d685c42c1173e7ccefc2baa4cef38f92ad998c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar-reproducibility
  tools:
  - InterCellar
  - R
  - biomaRt
  - graphite
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# int-pair-functional-annotation

## Summary

Annotate each ligand-receptor int-pair in InterCellar's function-verse with Gene Ontology and pathway terms (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB) retrieved via biomaRt and graphite, applying the all-components-enriched rule and ranking terms by int-pair occurrence.

## When to use

Use when a pre-computed CCI dataset (e.g. from CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR) has been loaded into InterCellar's function-verse and enriched int-pairs need GO/pathway functional annotation.

## When NOT to use

- Input is raw scRNA-seq expression data rather than pre-computed, inferred ligand-receptor interactions — InterCellar's workflow builds on pre-computed CCI data
- The CCI output comes from a tool outside InterCellar's supported set (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work
- You need condition-specific int-pair comparison or module-level data-driven analysis rather than GO/pathway annotation of individual int-pairs

## Inputs

- Table of enriched ligand-receptor int-pairs from a CCI dataset loaded into InterCellar's function-verse
- GO term-gene annotations queried from Ensembl via biomaRt
- Pathway term-gene annotations from KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB via graphite
- Gene symbols for all int-pair components, including multi-subunit complex members

## Outputs

- functional_annotation_ranked_terms.csv — ranked functional terms with annotated int-pair counts (occurrence)
- Ranked term-to-int-pair annotation table for linking biological pathways to ligand-receptor interactions

## How to apply

Load the table of enriched int-pairs from the CCI dataset into InterCellar's function-verse, then query Gene Ontology through Ensembl using biomaRt and query pathway databases (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB) through graphite to build term-gene annotation maps. Annotate a functional term to an int-pair only when ALL components of the int-pair — including members of multi-subunit ligand/receptor complexes — are enriched by that term, which is the strict all-components-enriched rule that prevents partial-complex assignments. Rank functional terms by the number of int-pairs annotated with each term (occurrence), and write the ranked table to functional_annotation_ranked_terms.csv. Run InterCellar locally with `reproducible = TRUE` so results are reproducible across R sessions, and use the app's download options to collect the ranked term-to-int-pair annotation table for downstream verification.

## Related tools

- **InterCellar** (R/Shiny platform hosting the function-verse where int-pairs are annotated and ranked by occurrence) — https://github.com/martaint/InterCellar-reproducibility
- **biomaRt** (Queries Gene Ontology through Ensembl to retrieve GO term-gene annotations for int-pair components)
- **graphite** (Queries pathway databases (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB) to retrieve pathway term-gene annotations)
- **R** (Implementation language of InterCellar (Bioconductor package); two lines of code install and launch the app)

## Examples

```
BiocManager::install("InterCellar"); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- functional_annotation_ranked_terms.csv exists and contains ranked functional terms with their annotated int-pair counts
- Every int-pair-term assignment satisfies the all-components-enriched rule: ALL components of the int-pair, including multi-subunit complex members, are enriched by the term
- Terms are ordered by occurrence (number of annotated int-pairs), and a term's count matches the number of int-pairs it is assigned to
- The int-pairs annotated match those in the loaded enriched int-pair table from the CCI dataset
- Results are reproducible across R sessions when run_app(reproducible = TRUE) is used

## Limitations

- InterCellar supports only a limited set of input tools; extension to other published tools is planned future work
- Annotated interactions are predictions awaiting experimental validation (e.g., smFISH or flow cytometry co-occurrence)
- Multi-subunit complexes make annotation strict: a term is dropped unless every complex member is enriched, which can leave int-pairs sparsely annotated

## Evidence

- [intro] function-verse GO/pathway annotation: "For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [methods] biomaRt GO query step: [evidence span withheld — anchor status: not_found]
- [methods] all-components-enriched rule: [evidence span withheld — anchor status: not_found]
- [methods] output artifact: [evidence span withheld — anchor status: not_found]
- [abstract] pathway linking rationale: "InterCellar guides the biological interpretation through customized analysis steps, multiple visualization options, and the possibility to link biological pathways to ligand-receptor interactions."
- [readme] README annotation basis: [evidence span withheld — anchor status: external_doc]
