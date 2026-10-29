---
name: observation-grouped-expression-summarization
description: Use when a Scanpy/AnnData scRNA object already carries gene age class
  annotations (e.g. from an oggmap orthomap) and the goal is a table of relative expression
  per gene age class grouped by an observation annotation like cell type or sampling
  timepoint.
when_to_use_negative:
- The AnnData object has no gene age class annotation yet — run the orthomap extraction
  and ID-matching steps (oggmap of2orthomap / TEI annotation) first; get_rematrix
  assumes ages are already attached
- The goal is a cell-level index rather than group-level summaries — use oggmap's
  TEI or partial TEI profile calculation instead of per-group mean/relative expression
- The observation annotation is continuous and un-binned (e.g. pseudotime) with no
  categorical grouping column — get_rematrix groups by annotated observations, so
  a categorical grouping must exist first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: oggmap
  role: Provides get_rematrix, which extracts mean/relative expression per gene age
    class grouped by an annotated observation, from raw counts or implemented expression
    transformation options
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Loads and manipulates the annotated AnnData scRNA object whose observations
    and expression layers feed get_rematrix
  repo: https://github.com/kullrich/oggmap
- name: Python
  role: Runtime for oggmap (developed with Python 3.10; Python 2.7x or <=3.10 not
    supported for older releases) and for the final relative-share normalization and
    CSV export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:6948fda27573c9185dada79a34ddd9f7c743315a4277db948f9a442fb467f851
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# observation-grouped-expression-summarization

## Summary

Summarizes scRNA-seq expression by aggregating mean expression per gene age class within annotated observation groups (e.g. cell type or sampling timepoint) and normalizing to relative shares, using oggmap's get_rematrix on a TEI-annotated AnnData object.

## When to use

Use when a Scanpy/AnnData scRNA object already carries gene age class annotations (e.g. from an oggmap orthomap) and the goal is a table of relative expression per gene age class grouped by an observation annotation like cell type or sampling timepoint.

## When NOT to use

- The AnnData object has no gene age class annotation yet — run the orthomap extraction and ID-matching steps (oggmap of2orthomap / TEI annotation) first; get_rematrix assumes ages are already attached
- The goal is a cell-level index rather than group-level summaries — use oggmap's TEI or partial TEI profile calculation instead of per-group mean/relative expression
- The observation annotation is continuous and un-binned (e.g. pseudotime) with no categorical grouping column — get_rematrix groups by annotated observations, so a categorical grouping must exist first

## Inputs

- TEI/gene-age-annotated AnnData (h5ad) object from oggmap-based orthomap matching (e.g. zebrafish AnnData with gene age class column)
- Choice of expression matrix: raw counts or an implemented transformation layer (e.g. normalized/log-transformed)
- Annotated observation column (categorical, e.g. cell type or sampling timepoint) to group by

## Outputs

- relative_expression_per_age_class.csv: rows = observation groups (e.g. cell types/timepoints), columns = gene age classes, values = relative shares summing to 1 per row

## How to apply

Load the TEI-annotated AnnData object (gene IDs matched to gene age classes via oggmap) with Scanpy. Choose the expression source — raw counts by default, or one of oggmap's implemented expression transformation options (e.g. a normalized or log-transformed layer) — consistent with get_rematrix input options. Run oggmap's get_rematrix, which 'extracts the relative expression per gene age class, grouped by any annotated observation such as cell-type or sampling timepoint', aggregating mean expression per age class over the chosen observation column. Finally normalize so the age-class values within each observation group sum to 1, and write the table (rows = observation groups, columns = gene age classes) to CSV. The normalization step is the decision point: raw mean expression per class is not comparable across groups with different total expression, so relative shares are the comparable quantity.

## Related tools

- **oggmap** (Provides get_rematrix, which extracts mean/relative expression per gene age class grouped by an annotated observation, from raw counts or implemented expression transformation options) — https://github.com/kullrich/oggmap
- **Scanpy** (Loads and manipulates the annotated AnnData scRNA object whose observations and expression layers feed get_rematrix) — https://github.com/kullrich/oggmap
- **Python** (Runtime for oggmap (developed with Python 3.10; Python 2.7x or <=3.10 not supported for older releases) and for the final relative-share normalization and CSV export)

## Evaluation signals

- The output CSV exists and contains one column per gene age class present in the orthomap
- Within each observation group (row), the relative expression shares sum to 1, with each individual value in (0, 1)
- Row identifiers match the levels of the chosen annotated observation column (e.g. the same set of cell types or sampling timepoints present in adata.obs)
- Mean expression values before normalization are reproducible when the same expression source (raw counts vs. transformed layer) is re-selected
- Gene age classes assigned in prior orthomap-matching steps are consistent between the AnnData and the summary table (no dropped or renamed classes)

## Limitations

- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or <=3.10, so environment setup must match
- Relative shares depend on which expression transformation is chosen (raw counts vs. normalized/log-transformed layers); mixing sources across runs changes the result
- The discussion/changelog section of the article provides no limitations content, so failure modes beyond input requirements are not documented here

## Evidence

- [methods] calculate mean/relative expression per gene age class: "- calculate mean/relative expression per gene age class."
- [abstract] oggmap results usable for weighted expression from scRNA objects: "which can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [readme] README: orthomap results feed weighted expression calculation: [evidence span withheld — anchor status: external_doc]
- [full_text] Source card finding on get_rematrix grouping behavior: [evidence span withheld — anchor status: not_found]
