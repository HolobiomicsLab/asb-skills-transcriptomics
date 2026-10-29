---
name: gene-ontology-annotation-via-biomart
description: Use when a pre-computed cell-cell interaction (CCI) dataset of enriched
  int-pairs must be annotated with Gene Ontology functional terms, as a prerequisite
  for ranking terms by annotated int-pair occurrence.
when_to_use_negative:
- The interaction table has not been pre-computed/enriched (InterCellar builds on
  pre-computed, inferred ligand-receptor interactions uploaded as CCI data).
- You need de novo GO enrichment statistics on a plain gene list rather than annotation
  of already-enriched int-pairs by partner-gene overlap.
- The analysis goal is pathway-only annotation without Gene Ontology — use the graphite-based
  pathway resources instead of biomaRt GO queries.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0605
tools:
- name: biomaRt
  role: Queries Gene Ontology through Ensembl to retrieve GO term-gene annotations
    for int-pair partner genes
- name: graphite
  role: Complementary R package used to retrieve pathway term-gene annotations (KEGG,
    Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB)
- name: InterCellar
  role: R/Shiny platform whose function-verse consumes the annotations to assign and
    rank functional terms per int-pair
  repo: https://github.com/martaint/InterCellar
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:178f1d39e262d71b45946005c512a06b2682933fa64f22c3f87ef0abca8d6448
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - InterCellar
  - R
  - biomaRt
  - graphite
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# gene-ontology-annotation-via-biomart

## Summary

Retrieves Gene Ontology term-gene annotations from Ensembl using the biomaRt R package so that functional terms can be assigned to predicted ligand-receptor interaction pairs (int-pairs) in InterCellar's function-verse.

## When to use

Use when a pre-computed cell-cell interaction (CCI) dataset of enriched int-pairs must be annotated with Gene Ontology functional terms, as a prerequisite for ranking terms by annotated int-pair occurrence.

## When NOT to use

- The interaction table has not been pre-computed/enriched (InterCellar builds on pre-computed, inferred ligand-receptor interactions uploaded as CCI data).
- You need de novo GO enrichment statistics on a plain gene list rather than annotation of already-enriched int-pairs by partner-gene overlap.
- The analysis goal is pathway-only annotation without Gene Ontology — use the graphite-based pathway resources instead of biomaRt GO queries.

## Inputs

- Table of enriched ligand-receptor interaction pairs (int-pairs) from a CCI dataset, e.g. produced by CellChat, CellPhoneDB, ICELLNET, or SingleCellSignalR
- Ensembl Gene Ontology annotations accessed via the biomaRt R package

## Outputs

- functional_annotation_ranked_terms.csv: ranked term-to-int-pair annotation table with functional terms ordered by number of annotated int-pairs (occurrence)

## How to apply

Load the table of enriched int-pairs from the CCI dataset into the function-verse of InterCellar. Query Gene Ontology through Ensembl using biomaRt to retrieve GO term-gene annotations (pathway term-gene annotations from KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, and PharmGKB are retrieved separately via graphite). Annotate a functional term to an int-pair only when ALL components of the int-pair — including members of multi-subunit complexes — are enriched by that term. Rank functional terms by the number of int-pairs annotated with each term (occurrence) and write the ranked table to functional_annotation_ranked_terms.csv.

## Related tools

- **biomaRt** (Queries Gene Ontology through Ensembl to retrieve GO term-gene annotations for int-pair partner genes)
- **graphite** (Complementary R package used to retrieve pathway term-gene annotations (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB))
- **InterCellar** (R/Shiny platform whose function-verse consumes the annotations to assign and rank functional terms per int-pair) — https://github.com/martaint/InterCellar

## Examples

```
BiocManager::install("InterCellar"); InterCellar::run_app(reproducible = TRUE)  # then load enriched int-pairs CCI data in the function-verse to annotate via biomaRt GO queries
```

## Evaluation signals

- functional_annotation_ranked_terms.csv exists and contains functional terms ranked by their annotated int-pair counts (occurrence).
- Every int-pair-term assignment satisfies the all-components-enriched rule: no term is assigned when any partner gene (including multi-subunit complex members) lacks that term.
- Multi-subunit complex partners are fully expanded and each subunit checked against the GO term-gene mapping, not only the representative gene.
- Retrieved GO term-gene mappings are consistent with current Ensembl release (re-query biomaRt to confirm no stale/offline annotations).
- Term ranking is monotone: terms with more annotated int-pairs appear higher in the output table.

## Limitations

- The annotation requires that ALL components of an int-pair, including multi-subunit complex members, be enriched by the term, so complexes with many subunits may fail annotation.
- InterCellar supports only a limited set of input CCI tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); results depend on the reference database and statistical assumptions of the upstream prediction method.
- biomaRt queries depend on live Ensembl access and current GO releases, so results are not frozen across sessions unless reproducibility is ensured (run_app(reproducible = TRUE)).

## Evidence

- [methods] Query Gene Ontology through Ensembl using the biomaRt R package to retrieve GO term-gene annotations.: [evidence span withheld — anchor status: not_found]
- [results] A functional term is annotated to an int-pair only when the term is enriched in both genes (or gene complexes) that are partners of the interaction.: [evidence span withheld — anchor status: not_found]
- [intro] In the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions.: "in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions"
- [intro] The workflow of InterCellar builds upon pre-computed, inferred ligand-receptor interactions, which can be uploaded in the first step as a CCI dataset.: "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data)"
- [readme] Starting from pre-computed ligand-receptor interactions, InterCellar provides filtering options, annotations and multiple visualizations based on GO and pathway databases.: [evidence span withheld — anchor status: external_doc]
