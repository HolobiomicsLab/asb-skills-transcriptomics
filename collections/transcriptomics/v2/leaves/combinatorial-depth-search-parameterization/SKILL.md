---
name: combinatorial-depth-search-parameterization
description: Use when running ERgene.FindERG on a pandas DataFrame expression matrix
  and you must choose a computational depth (e.g. 2 vs 3) to balance runtime against
  reference-gene search thoroughness, especially for datasets like GSE125792.
when_to_use_negative:
- You only need single-gene normalization rather than screening — ERgene.normalizationdata
  exists but the literature says single gene standardization is not desirable and
  should be used with caution
- The depth is set too large for the dataset size — the paper warns that at very large
  computational depth there may be no result
- You require an existing R/Excel workflow such as geNorm, Normfinder or BestKeeper
  rather than the Python ERgene pipeline
tools:
- name: ERgene
  role: Python library whose FindERG function performs depth-parameterized robust
    matrix block screening of reference genes
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame input format required by ERgene.FindERG and the computational
    power for normalization and screening
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:f26950128d2de3ca39f3b8b14d0eb56c3b99130e04795ae8528e07ff4d16777b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# combinatorial-depth-search-parameterization

## Summary

Parameterizes the combinatorial search depth in ERgene.FindERG, which screens endogenous reference genes via the inverse process of the internal reference method with robust matrix block operations (blocks of 1000 genes). Depth controls the trade-off between speed and coverage: depth 2 was fastest on a 1968-gene dataset (4.58 s), while depth 3 (17.3 s on GSE125792) yields deeper candidate sets but with declining efficiency.

## When to use

Use when running ERgene.FindERG on a pandas DataFrame expression matrix and you must choose a computational depth (e.g. 2 vs 3) to balance runtime against reference-gene search thoroughness, especially for datasets like GSE125792.

## When NOT to use

- You only need single-gene normalization rather than screening — ERgene.normalizationdata exists but the literature says single gene standardization is not desirable and should be used with caution
- The depth is set too large for the dataset size — the paper warns that at very large computational depth there may be no result
- You require an existing R/Excel workflow such as geNorm, Normfinder or BestKeeper rather than the Python ERgene pipeline

## Inputs

- Gene expression matrix as a pandas DataFrame (e.g. GSE125792 loaded from a tab-separated file with the first column set as index)
- Integer computational depth parameter passed to ERgene.FindERG (e.g. 2 or 3)

## Outputs

- Ranked list of candidate reference genes (variation of relative factor change sorted smallest to largest), e.g. seven genes at depth 3 for GSE125792
- Reported runtime of the depth-N run (e.g. 'calculate time:17.3s')
- Optional Upsetplot visualization when run in a Jupyter notebook
- Summary table such as reference_genes_depth3.csv pairing gene symbols with measured runtime

## How to apply

Load the expression matrix (e.g. GSE125792 read as a tab-separated file) into a pandas DataFrame and set the first column as the index before calling ERgene.FindERG(data, depth). Start with depth 2, which on 1968 genes took only 4.58 s; note that when depth is set to 3 or larger, efficiency begins to decline, so measure the runtime of each depth-3 run. For large datasets, ERgene automatically adopts block calculation, taking every 1000 genes as a block, then sorts the variation of relative factor change from smallest to largest and returns the top genes. At depth 3 on GSE125792 the run completed in 17.3 s and returned seven candidates (RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19), which you should record with the measured runtime (e.g. in reference_genes_depth3.csv). Do not set the depth too large, because the paper warns there may be no result.

## Related tools

- **ERgene** (Python library whose FindERG function performs depth-parameterized robust matrix block screening of reference genes) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame input format required by ERgene.FindERG and the computational power for normalization and screening)

## Examples

```
import ERgene, pandas as pd; data=pd.read_csv('gse.txt',sep='\t'); data=data.set_index(data.columns[0]); ERgene.FindERG(data,3)
```

## Evaluation signals

- Depth-3 run on GSE125792 returns exactly seven candidates (RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19) and a reported runtime near 17.3 s
- Depth-2 run on a 1968-gene dataset completes in roughly 4.58 s, confirming the expected efficiency regime
- Output list is ordered by variation of relative factor change from smallest to largest
- Results at comparable depths should be similar to those found by NormFinder and geNorm (per Table 2 comparison in the paper)
- No 'no result' outcome, which indicates the depth was not set too large for the dataset

## Limitations

- When the computational depth is set to 3 or larger, efficiency begins to decline; setting the depth too large may yield no result
- Runtime scales with depth (17.3 s at depth 3 vs 4.58 s at depth 2 for ~1968 genes), so deep searches are costly
- Single-gene normalization downstream of screening is cautioned against by the literature and MIQE guidelines unless invariable expression is demonstrated
- No ERgene software version beyond the README's mention of v1.2.0 (100x speed increase) is stated in the paper text

## Evidence

- [discussion] Total number of genes tested was 1968; depth 2 screening took 4.58 s: [evidence span withheld — anchor status: not_found]
- [discussion] Efficiency declines at depth 3 or larger; too large a depth may yield no result: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [discussion] Block calculation with 1000-gene blocks and sorting of variation: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [readme] Depth-3 invocation and runtime on GSE125792: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2
ERgene.FindERG(data,3)"
- [abstract] Inverse-process principle and robust matrix block operation: "The principle is based on the inverse process of the internal reference method, and the robust matrix block operation makes the selection of internal reference genes faster"
