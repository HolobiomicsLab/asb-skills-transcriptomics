---
name: library-output-parity-verification
description: Use when a task requires reimplementing an algorithm from a paper's description
  and validating it against the official library's exported function (e.g. ERgene.FindERG)
  on the same input matrix, expecting identical ranked results.
when_to_use_negative:
- When the goal is only to obtain reference genes using the official library — call
  ERgene.FindERG directly rather than reimplementing and diffing
- When the goal is normalizing expression with known reference genes (Ergene.Normalizationdata
  / normalizationdata) rather than screening candidates
- When the input is not a raw expression matrix but already-normalized data or a gene
  ranking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: ERgene
  role: Reference library whose exported FindERG function provides the ground-truth
    top-20 gene ranking the reimplementation must match
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Used to load and hold the expression matrix in DataFrame format, the required
    input type for both the reimplementation and ERgene.FindERG
- name: Python
  role: Language in which the inverse internal-reference algorithm is reimplemented
    and compared against the library output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:a5b64d776f39316afc79666a6ff3a1a300154efdb2ceee3fd41dadab1c85c115
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# library-output-parity-verification

## Summary

Verify a from-scratch reimplementation of an algorithm (here, ERgene's inverse internal-reference screening via correction factor matrices F1/F2, their difference ΔF, and per-gene ΔF variance) by reproducing a reference library's exported function on identical input and diffing the ranked outputs.

## When to use

Use when a task requires reimplementing an algorithm from a paper's description and validating it against the official library's exported function (e.g. ERgene.FindERG) on the same input matrix, expecting identical ranked results.

## When NOT to use

- When the goal is only to obtain reference genes using the official library — call ERgene.FindERG directly rather than reimplementing and diffing
- When the goal is normalizing expression with known reference genes (Ergene.Normalizationdata / normalizationdata) rather than screening candidates
- When the input is not a raw expression matrix but already-normalized data or a gene ranking

## Inputs

- Gene expression matrix as a Pandas DataFrame (e.g. read from a tab-separated file like gse.txt via pd.read_csv with the first column as index)
- Computational depth parameter (integer, e.g. 2 or 3) controlling how many samples are selected for the computation

## Outputs

- Reimplemented ranked top-20 candidate internal reference gene list
- ERgene.FindERG reference top-20 list for comparison
- Ranked gene list written to top20_reference_genes.csv

## How to apply

Load the same expression matrix (e.g. a tab-separated GSE file) into a Pandas DataFrame with the first column set as index, which is the format ERgene expects. Reimplement the inverse internal-reference computation: build per-sample relative correction factor matrices F1 and F2, compute their difference ΔF, take the per-gene variance of ΔF across samples, sort genes by that variance from smallest to largest, and take the top 20 genes. For large gene sets, mirror the reference implementation's robust block calculation, processing genes in blocks of 1000. Run ERgene.FindERG(data, depth) on the identical DataFrame with the same computational depth, then compare the two top-20 gene lists element-wise; parity (identical ranked lists) confirms the reimplementation is faithful. Write the verified ranked list to a CSV such as top20_reference_genes.csv.

## Related tools

- **ERgene** (Reference library whose exported FindERG function provides the ground-truth top-20 gene ranking the reimplementation must match) — github.com/Starlitnightly/ERgene
- **Pandas** (Used to load and hold the expression matrix in DataFrame format, the required input type for both the reimplementation and ERgene.FindERG)
- **Python** (Language in which the inverse internal-reference algorithm is reimplemented and compared against the library output)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- Reimplemented top-20 gene list is element-wise identical to the output of ERgene.FindERG run on the same DataFrame and depth
- Both implementations return exactly 20 candidate genes, sorted by ΔF variance from smallest to largest
- For large gene sets, block processing in units of 1000 genes yields the same result as unblocked computation
- Computational depth matches the reference call (e.g. depth 3 reproduces the README example output list); depth set too large yields no result in both implementations
- Output CSV (top20_reference_genes.csv) contains a single ranked column of 20 gene identifiers matching the verified list

## Limitations

- Computational efficiency declines at depth 3 or larger, and setting the depth too large may produce no result at all
- Single reference gene normalization via normalizationdata is not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated — this skill covers screening, not single-gene normalization
- ERgene results depend on the DataFrame being indexed correctly (data.set_index(data.columns[0]) in version 1.2); mis-indexing breaks parity checks

## Evidence

- [discussion] Sort-and-return contract for parity check: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] Block computation for large datasets: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] Input format requirement for FindERG: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [readme] Concrete invocation from README example: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2
ERgene.FindERG(data,3)"
- [discussion] Cross-method agreement as plausibility signal: "the internal reference genes found by ERgene, NormFinder and geNorm were similar"
