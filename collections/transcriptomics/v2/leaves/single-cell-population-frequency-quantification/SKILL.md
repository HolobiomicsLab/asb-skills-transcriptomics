---
name: single-cell-population-frequency-quantification
description: Use when you hold a scRNA-seq reference atlas with a cell-to-type identity
  file (e.g., the CCI distribution's data_ref/ cell-type and identity files for healthy
  BMMCs) and must report or verify the percentage composition of each annotated population.
when_to_use_negative:
- The single-cell dataset has no cell-type annotation or identity file, so cells cannot
  be assigned to the 17 hematopoietic populations before tabulation
- The goal is projecting leukemia subclusters onto the reference via CCI scores (BF
  probability) rather than summarizing the reference's own population composition
- Input is already a computed frequency or feature table rather than per-cell identity
  assignments
tools:
- name: CCI (counterpart composite index)
  role: Provides the distribution whose data_ref/ folder stores the healthy BMMC reference
    atlas files (expression matrix, gene list, cell types, cell identities) from which
    frequencies are tabulated
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Used to inspect the data_ref/ reference files and run pandas-style tabulation
    of cell-to-type assignments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:6e8c25d2921b67ef9f25e9a06691a87e2afb26351d6b6c1952e3ffb75e99ad30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-population-frequency-quantification

## Summary

Quantifies the relative abundance (counts and percentages) of annotated cell populations in a single-cell RNA-seq reference atlas, such as the healthy BMMC atlas of 18,751 cells from 4 healthy donors annotated into 17 hematopoietic populations, so that computed composition can be validated against paper-reported frequencies.

## When to use

Use when you hold a scRNA-seq reference atlas with a cell-to-type identity file (e.g., the CCI distribution's data_ref/ cell-type and identity files for healthy BMMCs) and must report or verify the percentage composition of each annotated population.

## When NOT to use

- The single-cell dataset has no cell-type annotation or identity file, so cells cannot be assigned to the 17 hematopoietic populations before tabulation
- The goal is projecting leukemia subclusters onto the reference via CCI scores (BF probability) rather than summarizing the reference's own population composition
- Input is already a computed frequency or feature table rather than per-cell identity assignments

## Inputs

- Cell type annotation file for reference atlas cells (from data_ref/ of the CCI distribution)
- Cell identity file listing each cell in the same order as the expression matrix (from data_ref/)
- Paper-reported population frequencies for cross-checking (e.g., T cells 53.11%, B cells 17.67%, erythroid progenitors 10.15%)

## Outputs

- cell_type_frequencies.csv containing one row per annotated population with computed count, computed percentage, and reported percentage
- Absolute deviations (computed vs reported percentage) per population for the 17 hematopoietic populations

## How to apply

Load the cell identity/type files of the reference atlas from the data_ref/ directory of the CCI distribution (github.com/pengfeeei/cci), inspecting them with bash and tabulating cell-to-type assignments in a pandas-style fashion. Tally the number of cells assigned to each of the 17 annotated hematopoietic populations and convert counts to percentages of the total annotated cells. Decision point: compare each computed percentage against the paper-reported values — T cells 53.11%, B cells 17.67%, erythroid progenitors 10.15% — recording absolute deviations per population; accept matching within rounding tolerance. Write the combined frequency table (computed and reported frequencies per population) to cell_type_frequencies.csv. The rationale is that CCI projects leukemia subclusters onto this reference atlas, so a correctly proportioned reference composition underpins downstream counterpart-mapping interpretation.

## Related tools

- **CCI (counterpart composite index)** (Provides the distribution whose data_ref/ folder stores the healthy BMMC reference atlas files (expression matrix, gene list, cell types, cell identities) from which frequencies are tabulated) — https://github.com/pengfeeei/cci
- **bash** (Used to inspect the data_ref/ reference files and run pandas-style tabulation of cell-to-type assignments)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('data_ref/celltypes.tsv',sep='\t'); f=(df['celltype'].value_counts(normalize=True)*100).round(2); f.to_csv('cell_type_frequencies.csv')"
```

## Evaluation signals

- Each of the 17 annotated hematopoietic populations appears exactly once in the output table
- Computed population percentages sum to approximately 100%
- Computed frequencies for T cells, B cells, and erythroid progenitors match the reported 53.11%, 17.67%, and 10.15% within rounding tolerance
- Rarest population is stromal cells at 0.04% and the most abundant is T cells at 53.11%, consistent with the paper-reported ordering
- cell_type_frequencies.csv contains both computed and reported frequency columns with recorded absolute deviations

## Limitations

- Validation is limited to cross-checking against the three paper-reported percentages (T cells, B cells, erythroid progenitors); the other 14 populations rely only on internal consistency (single occurrence, ~100% sum)
- Percentages are only meaningful if every cell in the identity file is annotated to one of the 17 populations; unlabeled or double-counted cells distort the composition
- CCI projects cell clusters rather than single cells ('due to prevalent droplets in scRNA-seq'), so cluster-level mapping results should not be equated with single-cell population frequencies

## Evidence

- [readme] data_ref/ contents: "Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell"
- [abstract] atlas scale and scope: "we analyzed ~200,000 single-cell transcriptomes of bone marrow mononuclear cells (BMMCs) and its subsets from 23 clinical samples"
- [abstract] reference atlas role: "We constructed a comprehensive cell atlas as hematopoietic reference"
- [readme] cluster-level projection caveat: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq."
- [other] workflow tabulation step: [evidence span withheld — anchor status: not_found]
