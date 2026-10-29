---
name: relative-factor-variation-ranking
description: Use when a gene expression matrix (pandas DataFrame) must be screened
  for stable internal reference genes, and candidate genes should be ranked by smallest
  variance of relative factor change with the top 20 returned.
when_to_use_negative:
- 'The goal is single-gene normalization: single reference gene standardization is
  not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated
  — use Ergene.normalizationdata with caution, not as the screening endpoint'
- 'Input data is probe-based rather than raw expression values: the paper''s test
  data ''are not converted by a probe'', so applicability to probe-converted matrices
  is untested'
- 'A very large computational depth is required on a large gene set: efficiency declines
  at depth 3 or larger and FindERG may return no result'
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ERgene
  role: Python library providing FindERG, which screens internal reference genes by
    ranking variance of relative factor change and returning the top 20 candidates
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame format required as input by ERgene for the expression
    matrix
- name: upsetplot
  role: Dependency used to render the UpSet plot of pairwise set intersections of
    screened candidate genes
  repo: github.com/Starlitnightly/ERgene
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:f98ba47c6bd6214d7f335c924351d8ac41414051072e43881d73f1d35faa2b31
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - upsetplot
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# relative-factor-variation-ranking

## Summary

A screening method from the ERgene Python library that ranks candidate endogenous reference genes by the variance of the relative factor change, returning the top 20 genes with the smallest variation. It exploits the inverse process of the internal reference method with robust matrix block operation, making selection faster than geNorm, Normfinder or BestKeeper.

## When to use

Use when a gene expression matrix (pandas DataFrame) must be screened for stable internal reference genes, and candidate genes should be ranked by smallest variance of relative factor change with the top 20 returned.

## When NOT to use

- The goal is single-gene normalization: single reference gene standardization is not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated — use Ergene.normalizationdata with caution, not as the screening endpoint
- Input data is probe-based rather than raw expression values: the paper's test data 'are not converted by a probe', so applicability to probe-converted matrices is untested
- A very large computational depth is required on a large gene set: efficiency declines at depth 3 or larger and FindERG may return no result

## Inputs

- Gene expression matrix in pandas DataFrame format (e.g. GSE125792 read from gse.txt with sep='\t', first column set as index)
- Computational depth parameter (integer, e.g. 2 or 3)

## Outputs

- Ranked list of top 20 candidate internal reference genes by smallest variance of relative factor change (e.g. ['ASHGV40057862', 'ASHGV40057056', ...])
- UpSet plot (upsetplot.png) of pairwise set intersections when run in Jupyter notebook
- For multi-pair screening: per-gene sample-pair coverage fractions and an overall duplication rate

## How to apply

Load the expression matrix (e.g. GSE125792 gse.txt, tab-separated, gene IDs as index) into a pandas DataFrame, since ERgene requires DataFrame format input. Call ERgene.FindERG(Data, Depth) with a computational depth (depth 2 screened 1968 genes in 4.58 s; depth 3 took 17.3 s on GSE125792); the program sorts the variance of the relative factor change from smallest to largest and returns the top 20 genes. For very large gene sets, use block calculation taking every 1000 genes as a block; avoid depths of 3 or larger when speed matters, and do not set the depth too large or there may be no result. For multi-sample-pair screening (e.g. 66 sample pairs over 12 GSE125792 samples), invoke FindERG per pair, then aggregate per-pair lists to compute sample-pair coverage (flag genes covered in >80% of pairs) and an overall duplication rate of repeated selections (>50% is the expected invariant). Validate the pairwise set intersections visually with an Upsetplot, which FindERG generates automatically in a Jupyter notebook.

## Related tools

- **ERgene** (Python library providing FindERG, which screens internal reference genes by ranking variance of relative factor change and returning the top 20 candidates) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame format required as input by ERgene for the expression matrix)
- **upsetplot** (Dependency used to render the UpSet plot of pairwise set intersections of screened candidate genes) — github.com/Starlitnightly/ERgene

## Examples

```
import ERgene, pandas as pd; data=pd.read_csv('gse.txt',sep='\t'); data=data.set_index(data.columns[0]); ERgene.FindERG(data,3)
```

## Evaluation signals

- FindERG returns exactly the top 20 genes as a Python list, sorted by smallest-to-largest variance of relative factor change
- Output gene IDs match the input matrix's index (e.g. ASHGV* accession-style IDs for GSE125792)
- On the GSE125792 benchmark (12 samples, 66 sample pairs), 10 genes show >80% sample-pair coverage and the overall duplication rate of repeated selections exceeds 50%
- upsetplot.png is produced and readable as an image when run in Jupyter notebook
- Runtime stays bounded: 4.58 s for 1968 genes at depth 2; ~17.3 s for GSE125792 at depth 3 (v1.2.0 claims ~100x speedup)

## Limitations

- Computational efficiency declines at computational depth 3 or larger; when the depth is set too large, there may be no result
- Single reference gene normalization via normalizationdata is not desirable and should be used with caution
- No ERgene software version number, DOI, or data accession statement is given in the article body; the README indicates behavior differs from Ver1.2 onward (set_index on first column required)

## Evidence

- [discussion] then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result.: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] This function is used to screen internal reference genes. The parameter data are in DataFrame format: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [discussion] When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block.: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result.: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [readme] data=pd.read_csv('gse.txt',sep='\t')#GSE125792
	data=data.set_index(data.columns[0])#This is Ver1.2
	ERgene.FindERG(data,3): "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
	data=data.set_index(data.columns[0])#This is Ver1.2
	ERgene.FindERG(data,3)"
