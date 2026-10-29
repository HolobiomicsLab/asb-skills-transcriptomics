---
name: block-matrix-computation-for-large-gene-datasets
description: Use when a gene expression matrix (Pandas DataFrame) is too large to
  screen endogenous reference genes in a single pass and needs to be split into 1000-gene
  blocks for variance-of-ΔF computation.
when_to_use_negative:
- The gene dataset is small enough to screen in a single pass without block computation
- The goal is data normalization for a known reference gene (use ERgene.normalizationdata
  instead of screening)
- A single internal reference gene is to be used without evidence of invariable expression,
  which MIQE guidelines deem unacceptable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: ERgene
  role: Python library implementing the inverse internal-reference screening with
    robust matrix block operation (1000-gene blocks) via FindERG
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame data structures on which the block computation and
    variance calculations are performed
- name: Python
  role: Execution environment for the ERgene library and the block computation workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:d102ede2c55e3967bff59b6946a37cc55a6e8c7e4d943ece17df7d134e4d709a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# block-matrix-computation-for-large-gene-datasets

## Summary

A robust block computation strategy from the ERgene library that processes oversized gene expression datasets in blocks of 1000 genes at a time while screening internal reference genes via the inverse internal-reference method.

## When to use

Use when a gene expression matrix (Pandas DataFrame) is too large to screen endogenous reference genes in a single pass and needs to be split into 1000-gene blocks for variance-of-ΔF computation.

## When NOT to use

- The gene dataset is small enough to screen in a single pass without block computation
- The goal is data normalization for a known reference gene (use ERgene.normalizationdata instead of screening)
- A single internal reference gene is to be used without evidence of invariable expression, which MIQE guidelines deem unacceptable

## Inputs

- Gene expression matrix as a Pandas DataFrame (genes × samples), e.g. loaded from a tab-separated file such as gse.txt

## Outputs

- Ranked list of the top 20 candidate endogenous reference genes (e.g. written to top20_reference_genes.csv)
- Per-gene variance of ΔF across samples as the stability measure

## How to apply

Load the expression matrix as a Pandas DataFrame and, if the number of genes is too large, split genes into blocks of 1000 for the block calculation. Within each block, build the per-sample relative correction factor matrices F1 and F2, compute their difference ΔF, and calculate the per-gene variance of ΔF across samples as the stability measure. Concatenate results across blocks, sort genes by ΔF variance from smallest to largest, and return the top 20 genes as candidate reference genes. The rationale is that robust matrix block operation makes internal reference gene selection faster than other methods while keeping memory usage tractable.

## Related tools

- **ERgene** (Python library implementing the inverse internal-reference screening with robust matrix block operation (1000-gene blocks) via FindERG) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame data structures on which the block computation and variance calculations are performed)
- **Python** (Execution environment for the ERgene library and the block computation workflow)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- Top-20 gene list from the block computation matches the output of ERgene.FindERG on the same input matrix
- Results are sorted by ΔF variance from smallest to largest before selecting the top 20 genes
- Each block contains at most 1000 genes, consistent with the stated block size
- Output is a ranked list of exactly 20 candidate reference genes for sufficiently large input datasets
- Runtime remains tractable (e.g. depth 2 screening of 1968 genes took 4.58 s), indicating block computation is effective

## Limitations

- Computational depth set too large may yield no result; efficiency declines at depth 3 or larger
- Version 1.2.0 of ERgene increased computing speed by a factor of 100, so older versions may be much slower
- Upsetplot visualization is only generated in Jupyter notebook environments

## Evidence

- [discussion] Block calculation: every 1000 genes taken as a block for large datasets: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] Top 20 genes returned: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [abstract] ERgene is based on the inverse process of the internal reference method: "The principle is based on the inverse process of the internal reference method, and the robust matrix block operation makes the selection of internal reference genes faster than any other method."
- [readme] Screening call with DataFrame input: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
	data=data.set_index(data.columns[0])#This is Ver1.2
	ERgene.FindERG(data,3)"
- [readme] Speed improvement in version 1.2.0: "In new Version 1.2.0, the computing speed is increased by a factor of 100"
