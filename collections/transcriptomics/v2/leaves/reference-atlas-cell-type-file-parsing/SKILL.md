---
name: reference-atlas-cell-type-file-parsing
description: Use when you have the CCI reference atlas files (expression matrix, gene
  list, cell types, cell identity) in data_ref/ and need per-population cell counts
  or a healthy BMMC composition table matching reported values (e.g., T cells 53.11%).
when_to_use_negative:
- Your input is a target scRNA-seq expression matrix for clustering and CCI projection
  (that belongs in data_obj/, not data_ref/) — this skill only tabulates the already-annotated
  reference atlas.
- You need de novo cell-type annotation of unlabeled cells; the reference atlas files
  already contain curated cell-to-type assignments.
- The reference atlas is not the healthy BMMC atlas (e.g., a non-hematopoietic carcinoma
  reference), so the 17-population composition and reported percentages do not apply.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CCI (counterpart composite index)
  role: Provides the data_ref/ folder containing the reference atlas cell type and
    identity files that this skill parses; the atlas serves as the healthy reference
    for CCI projection.
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Used to inspect the data_ref/ files from the CCI distribution and drive the
    parsing workflow.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:9a94a5eb9083ff3a3deb636738e18cbb1098ee1aa6bdc54aa0e3cab7f9a244ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reference-atlas-cell-type-file-parsing

## Summary

Parses the reference-atlas cell type and cell identity files shipped in the data_ref/ folder of the pengfeeei/cci distribution (a comprehensive normal BMMC atlas from 4 healthy donors) to tally the 17 annotated hematopoietic cell populations and compute their relative frequencies.

## When to use

Use when you have the CCI reference atlas files (expression matrix, gene list, cell types, cell identity) in data_ref/ and need per-population cell counts or a healthy BMMC composition table matching reported values (e.g., T cells 53.11%).

## When NOT to use

- Your input is a target scRNA-seq expression matrix for clustering and CCI projection (that belongs in data_obj/, not data_ref/) — this skill only tabulates the already-annotated reference atlas.
- You need de novo cell-type annotation of unlabeled cells; the reference atlas files already contain curated cell-to-type assignments.
- The reference atlas is not the healthy BMMC atlas (e.g., a non-hematopoietic carcinoma reference), so the 17-population composition and reported percentages do not apply.

## Inputs

- data_ref/ reference atlas files from github.com/pengfeeei/cci: expression matrix (cells as rows, genes as columns), gene list file, cell types file, and per-cell identity file
- Healthy BMMC reference atlas of 18,751 cells from 4 healthy donors annotated into 17 cell populations

## Outputs

- Per-population cell count tallies for the 17 annotated hematopoietic populations
- cell_type_frequencies.csv containing computed and paper-reported percentage frequencies per population

## How to apply

Inspect the four data_ref/ files (expression matrix with cells as rows and genes as columns, gene list, cell types, and per-cell identity) using bash and pandas-style tabulation to confirm the identity column aligns with the expression matrix rows. Tally the number of cells assigned to each of the 17 annotated hematopoietic populations (the healthy BMMC atlas built from 18,751 cells of 4 healthy donors) and convert counts to percentages of the total annotated cells. Record absolute deviations between computed percentages and paper-reported values (T cells 53.11%, B cells 17.67%, erythroid progenitors 10.15%, NK cells 11.62%, stromal cells rarest at 0.04%). Write the combined table of computed and reported frequencies to cell_type_frequencies.csv. This reference composition matters downstream because CCI projects target leukemia cell clusters onto this healthy reference atlas to identify healthy counterparts.

## Related tools

- **CCI (counterpart composite index)** (Provides the data_ref/ folder containing the reference atlas cell type and identity files that this skill parses; the atlas serves as the healthy reference for CCI projection.) — https://github.com/pengfeeei/cci
- **bash** (Used to inspect the data_ref/ files from the CCI distribution and drive the parsing workflow.)

## Examples

```
python -c "import pandas as pd; ct=pd.read_csv('data_ref/cell_types.txt',sep='\t'); f=ct['cell_type'].value_counts(normalize=True)*100; f.to_csv('cell_type_frequencies.csv')"
```

## Evaluation signals

- Each of the 17 annotated hematopoietic populations appears exactly once in the frequency table.
- Computed percentages sum to approximately 100% of the 18,751 annotated reference cells.
- Computed frequencies for T cells, B cells, and erythroid progenitors match the reported 53.11%, 17.67%, and 10.15% within rounding tolerance.
- Rarest population (stromal cells) is at 0.04% and NK cells at 11.62%, consistent with the paper-reported composition.
- The cell identity file rows align one-to-one with the rows of the expression matrix in data_ref/.

## Limitations

- Frequency composition is specific to the healthy BMMC atlas of 4 donors and 17 populations; it is not a general hematopoiesis composition estimate.
- CCI projects cell clusters rather than single cells, so the reference files must remain formatted exactly like the toy data for downstream compatibility.
- The distribution relies on the user formatting reference files correctly (expression matrix, gene list, cell types, identity); malformed files will break tabulation or later CCI runs.

## Evidence

- [readme] Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell as in expression matrix.: "Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell as in"
- [abstract] We constructed a comprehensive cell atlas as hematopoietic reference: "We constructed a comprehensive cell atlas as hematopoietic reference"
- [readme] CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq.: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq."
- [abstract] we analyzed ~200,000 single-cell transcriptomes of bone marrow mononuclear cells (BMMCs) and its subsets from 23 clinical samples: "we analyzed ~200,000 single-cell transcriptomes of bone marrow mononuclear cells (BMMCs) and its subsets from 23 clinical samples"
