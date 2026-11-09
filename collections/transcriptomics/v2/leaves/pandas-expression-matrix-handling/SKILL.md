---
name: pandas-expression-matrix-handling
description: Use when a raw tab-delimited gene expression matrix (e.g. a GSE export
  like gse.txt) must be shaped into an indexed Pandas DataFrame as input to ERgene's
  FindERG or normalizationdata functions.
when_to_use_negative:
- When the goal is reference-gene screening with R or Excel tools (geNorm, Normfinder,
  BestKeeper) rather than a Python Pandas workflow — ERgene was built because no Python
  package existed for this task.
- When the matrix is already indexed or preprocessed for a downstream tool that does
  not require Pandas DataFrame input with the first column as index.
- When a single reference gene suffices — the literature states single-gene normalization
  is not desirable, so normalizationdata should be used with caution.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ERgene
  role: Consumes the Pandas DataFrame via FindERG(data, depth) to screen internal
    reference genes and via normalizationdata for single-gene normalization
  repo: https://github.com/Starlitnightly/ERgene
- name: Pandas
  role: Loads and shapes the expression matrix (read_csv with sep='\t', set_index
    to first column) into the DataFrame format ERgene requires
- name: Python
  role: Runtime environment for the ERgene/Pandas workflow (pip install ERgene)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:f70c114ea80bc8c217c07090c7820d000c539e710ad50af5e63991ab6373513e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Starlitnightly/ERgene
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pandas-expression-matrix-handling

## Summary

Prepare a gene expression matrix as a Pandas DataFrame with genes indexed by the first column so it can be consumed by ERgene's FindERG screening, which computes inverse internal-reference correction factor variances to rank candidate reference genes.

## When to use

Use when a raw tab-delimited gene expression matrix (e.g. a GSE export like gse.txt) must be shaped into an indexed Pandas DataFrame as input to ERgene's FindERG or normalizationdata functions.

## When NOT to use

- When the goal is reference-gene screening with R or Excel tools (geNorm, Normfinder, BestKeeper) rather than a Python Pandas workflow — ERgene was built because no Python package existed for this task.
- When the matrix is already indexed or preprocessed for a downstream tool that does not require Pandas DataFrame input with the first column as index.
- When a single reference gene suffices — the literature states single-gene normalization is not desirable, so normalizationdata should be used with caution.

## Inputs

- Tab-delimited gene expression matrix file (e.g. gse.txt from GSE125792)
- Pandas DataFrame with one row per gene, first column used as gene index
- Integer computational depth parameter (e.g. 2 or 3)

## Outputs

- Ranked list of top 20 candidate internal reference genes (e.g. ['ASHGV40057862', 'ASHGV40057056', ...])
- Upsetplot figure when run in a Jupyter notebook
- Normalized expression values from ERgene.normalizationdata(data, gene_name)

## How to apply

Load the expression file with Pandas (e.g. pd.read_csv('gse.txt', sep='\t')), then reset the index to the first column with data.set_index(data.columns[0]) as required by ERgene version 1.2. Pass this DataFrame to ERgene.FindERG(data, depth), where the depth parameter (e.g. 3) sets the computational depth and controls how many samples are used per computation. For large gene sets, ERgene internally uses robust block computation taking every 1000 genes as a block. The function returns the top 20 genes sorted by smallest variance of the relative correction factor change (ΔF across samples); a single gene can then be normalized via ERgene.normalizationdata(data, gene).

## Related tools

- **ERgene** (Consumes the Pandas DataFrame via FindERG(data, depth) to screen internal reference genes and via normalizationdata for single-gene normalization) — https://github.com/Starlitnightly/ERgene
- **Pandas** (Loads and shapes the expression matrix (read_csv with sep='\t', set_index to first column) into the DataFrame format ERgene requires)
- **Python** (Runtime environment for the ERgene/Pandas workflow (pip install ERgene))

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t').set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- DataFrame check: the expression matrix must have its first column set as index (data.set_index(data.columns[0]) in ERgene 1.2) before calling FindERG.
- FindERG returns a list of top 20 genes sorted by smallest ΔF variance — verify output length is 20 for gene sets with ≥20 candidates.
- Runtime check: the function prints calculate time (e.g. 17.3 s at depth 3 for 1968 genes); the article reports 4.58 s at depth 2 — large deviations may indicate depth too large (no result possible at too-large depth).
- Cross-check: reimplemented top-20 list should match ERgene's FindERG output on the identical input matrix; write the ranked list to top20_reference_genes.csv.
- Dependencies present: Numpy, time, pandas, itertools, upsetplot (required for the Jupyter Upsetplot output).

## Limitations

- Computational depth 3 or larger degrades efficiency; a too-large depth may yield no result at all.
- Single-gene normalization via normalizationdata is explicitly discouraged — single gene standardization is not desirable per the literature and MIQE guidance.
- ERgene's test data were not converted by a probe, and no data availability/accession statements for the test datasets are provided.
- No ERgene version number beyond the README's 1.2.0 is stated in the article, and installation instructions are not in the article text (README: pip install ERgene).

## Evidence

- [readme] README example invocation: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2
ERgene.FindERG(data,3)"
- [discussion] DataFrame input requirement: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [discussion] Block computation for large datasets: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] Top-20 output and sorting: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] Depth parameter effects: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
