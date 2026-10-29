---
name: gene-age-class-count-summarization
description: Use when you already have a gene-to-gene-age-class orthomap table for
  a focal species (e.g. from oggmap of2orthomap on OrthoFinder output) and need per-age-class
  gene counts for quality checking or downstream plotting.
when_to_use_negative:
- You need gene-age-weighted expression per cell or transcriptome evolutionary index
  (TEI) on scRNA objects — that requires oggmap's Scanpy-based downstream steps, not
  simple per-class counts.
- The orthogroup-to-age assignments do not yet exist — run oggmap of2orthomap (or
  eggnog/plaza equivalents) on the gene family results first; this skill only summarizes
  the finished orthomap.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3299
- http://edamontology.org/topic_0091
tools:
- name: oggmap
  role: Produces the orthomap (of2orthomap sub-command assigns LCA gene age classes
    per orthogroup) that this skill summarizes
  repo: https://github.com/kullrich/oggmap
- name: OrthoFinder
  role: Generates the orthologous groups from which the orthomap and hence the age-class
    counts are derived
  repo: https://github.com/davidemms/OrthoFinder
- name: Python
  role: pandas is used to aggregate the orthomap and count genes per age class
  repo: https://pypi.org/project/oggmap/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:2b5e9c7d5c1766aad0f74cf23add4a81ff7b7380626f296d740bf76e15eb3330
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - oggmap
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3299
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gene-age-class-count-summarization

## Summary

Aggregate an extracted orthomap (gene-to-gene-age-class assignments, e.g. danio_rerio_orthomap.tsv produced by oggmap of2orthomap) with pandas to count the number of genes per gene age class, writing a per-class count table such as danio_rerio_age_class_counts.csv. This summarizes the evolutionary age composition of a focal species' gene set.

## When to use

Use when you already have a gene-to-gene-age-class orthomap table for a focal species (e.g. from oggmap of2orthomap on OrthoFinder output) and need per-age-class gene counts for quality checking or downstream plotting.

## When NOT to use

- You need gene-age-weighted expression per cell or transcriptome evolutionary index (TEI) on scRNA objects — that requires oggmap's Scanpy-based downstream steps, not simple per-class counts.
- The orthogroup-to-age assignments do not yet exist — run oggmap of2orthomap (or eggnog/plaza equivalents) on the gene family results first; this skill only summarizes the finished orthomap.

## Inputs

- Orthomap TSV for the query species (e.g. danio_rerio_orthomap.tsv: gene-to-gene-age-class rows, lowest common ancestor per orthogroup)
- pandas environment for aggregation

## Outputs

- Per-age-class gene count table (danio_rerio_age_class_counts.csv: gene age class vs. number of genes)
- Summary statistics for validating per-class gene counts against plausible ranges

## How to apply

First ensure the orthomap exists and parses as tabular rows of gene-to-age-class assignments — in the oggmap case study this is danio_rerio_orthomap.tsv, extracted by running of2orthomap on the OrthoFinder output directory with Danio rerio as the query species, where each orthologous group's lowest common ancestor (LCA) is assigned as the gene age class. Load the TSV with pandas, group or count observations per age-class value, and write the resulting counts (one row per gene age class, one column of gene counts) to danio_rerio_age_class_counts.csv. Validate that every gene in the orthomap maps to exactly one age class and that per-age-class gene counts fall within plausible ranges for a zebrafish gene set; these counts correspond to the per-class gene numbers highlighted in the article's Fig. 1c. Treat outlier or empty classes as signals to re-check the upstream OrthoFinder run (e.g. the '-S last' search setting) rather than adjusting counts manually.

## Related tools

- **oggmap** (Produces the orthomap (of2orthomap sub-command assigns LCA gene age classes per orthogroup) that this skill summarizes) — https://github.com/kullrich/oggmap
- **OrthoFinder** (Generates the orthologous groups from which the orthomap and hence the age-class counts are derived) — https://github.com/davidemms/OrthoFinder
- **Python** (pandas is used to aggregate the orthomap and count genes per age class) — https://pypi.org/project/oggmap/

## Examples

```
python -c "import pandas as pd; om = pd.read_csv('danio_rerio_orthomap.tsv', sep='\t'); om.value_counts(om.columns[-1]).to_csv('danio_rerio_age_class_counts.csv')"
```

## Evaluation signals

- danio_rerio_age_class_counts.csv exists and contains one row per gene age class with a positive integer gene count
- Sum of per-class gene counts equals the number of genes in danio_rerio_orthomap.tsv (no genes dropped or double-counted)
- Per-age-class gene counts fall within plausible ranges for a zebrafish gene set
- No genes with a missing/empty age-class value remain in the count table
- Counts are consistent with the gene-age-class numbers highlighted in the article's Fig. 1c

## Limitations

- Counts only reflect the LCA assignments of the upstream orthology run — changing the OrthoFinder search setting (e.g. -S last vs -S diamond_ultra_sens) or Ensembl release changes the age-class composition
- Simple counts do not weight expression; they cannot substitute for TEI or partial-TEI analyses on scRNA data
- The oggmap paper provides no data availability statements or versioned release details in the discussion section, so count validation ranges must be justified externally

## Evidence

- [results] Setting zebrafish as the focal species, the authors extracted the corresponding orthomap (Fig. 1b) and highlighted the number of genes per gene age class (Fig. 1c).: [evidence span withheld — anchor status: not_found]
- [methods] extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species.: "extract and assign lowest common ancestor (LCA) from orthologous groups as gene age class for a target species."
- [abstract] which allows for the easy extraction of an orthomap (gene ages per orthogroup) for any given query species: "which allows for the easy extraction of an orthomap (gene ages per orthogroup) for any given query species"
- [readme] The following code extracts the `orthomap` for `Danio rerio` based on pre-calculated OrthoFinder results and ensembl release-113:: "The following code extracts the `orthomap` for `Danio rerio` based on pre-calculated 
OrthoFinder results and ensembl release-113:"
