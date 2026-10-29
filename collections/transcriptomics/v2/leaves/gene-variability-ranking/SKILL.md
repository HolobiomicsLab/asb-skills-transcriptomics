---
name: gene-variability-ranking
description: Use when you have a gene expression matrix (e.g. GSE125792, 1968 genes)
  and need to screen or rank stable endogenous reference genes for qPCR normalization
  in Python.
when_to_use_negative:
- Normalization of already-selected reference genes — use ERgene.normalizationdata
  instead; single gene standardization is not desirable per the literature and MIQE
  guidance unless invariable expression is clearly demonstrated
- Computational depth is set too large (3 or larger declines in efficiency; a very
  large depth may return no result)
- You need an R or Excel workflow — geNorm (qbase+/Excel 2003) and Normfinder (R/Excel
  2003) cover that; ERgene is the Python-specific choice
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ERgene
  role: Python library whose exported FindERG function performs the inverse internal-reference
    screening and variance ranking
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame input format and computational power underlying ERgene's
    screening
- name: Python
  role: Language environment in which ERgene is loaded and run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:83370d0f708d679d7d9834078bd74f94baa9102e74d97bdca47397828178d953
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-variability-ranking

## Summary

Ranks candidate endogenous reference genes by computing the variance of relative factor change across genes with the ERgene Python library, which inverts the internal-reference method and uses robust matrix block computation (every 1000 genes as a block) to return the top 20 most stable genes.

## When to use

Use when you have a gene expression matrix (e.g. GSE125792, 1968 genes) and need to screen or rank stable endogenous reference genes for qPCR normalization in Python.

## When NOT to use

- Normalization of already-selected reference genes — use ERgene.normalizationdata instead; single gene standardization is not desirable per the literature and MIQE guidance unless invariable expression is clearly demonstrated
- Computational depth is set too large (3 or larger declines in efficiency; a very large depth may return no result)
- You need an R or Excel workflow — geNorm (qbase+/Excel 2003) and Normfinder (R/Excel 2003) cover that; ERgene is the Python-specific choice

## Inputs

- Gene expression matrix in pandas DataFrame format (e.g. tab-separated GSE125792 gse.txt with gene IDs in the first column set as index)
- Computational depth parameter (integer, e.g. 2)

## Outputs

- Ranked list of the top 20 candidate reference genes sorted by smallest variance of relative factor change
- UpSet plot (in Jupyter notebook) of candidate reference genes
- Optionally a ranked gene list written to candidate_reference_genes.csv

## How to apply

Load the gene expression matrix (e.g. the 1968-gene GSE125792 table) into a pandas DataFrame with the first column set as the index, and explicitly attach dependencies such as pandas since the ERgene package does not attach them itself. Call the exported ERgene function `FindERG(Data, Depth)` — note the exact symbol casing declared in the package API surface — with the computational depth set (depth 2 completed the 1968-gene McLoughlin dataset in 4.58 s). ERgene applies the inverse internal-reference procedure with robust matrix block computation, taking every 1000 genes as a block for large datasets, and computes the variance of relative factor change across genes. The results of variation are sorted from smallest to largest and the top 20 genes are returned as the candidate reference panel (e.g. RPL19, RPS4X, RPL13, TUBA1A, SF3B5, RPS3a, RPL9, RPL39L, LDHA, RPS8, RPL31, FTL, RPS3, RPL22, PINLYP, CAPG, UQCRH, RPS5, RPSAP58, RPL36 at depth 2 on GSE125792). Optionally write the ranked list to a CSV and confirm it reproduces the reported genes; in Jupyter an UpSet plot is generated. Correctness can be judged by comparing the ranked output against known stable genes from ERgene, NormFinder and geNorm, which were reported to be similar.

## Related tools

- **ERgene** (Python library whose exported FindERG function performs the inverse internal-reference screening and variance ranking) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame input format and computational power underlying ERgene's screening)
- **Python** (Language environment in which ERgene is loaded and run)

## Examples

```
import ERgene, pandas as pd; data=pd.read_csv('gse.txt',sep='\t'); data=data.set_index(data.columns[0]); ERgene.FindERG(data,2)
```

## Evaluation signals

- Output is a ranked list whose top 20 genes match the reported panel for depth 2 on GSE125792 (RPL19, RPS4X, RPL13, TUBA1A, SF3B5, RPS3a, RPL9, RPL39L, LDHA, RPS8, RPL31, FTL, RPS3, RPL22, PINLYP, CAPG, UQCRH, RPS5, RPSAP58, RPL36)
- Depth-2 run on the 1968-gene GSE125792 matrix completes in seconds (article reports 4.58 s); large deviation suggests misconfiguration
- The function invoked is the exported symbol `FindERG` with the exact casing from the package API surface, taking (DataFrame, depth) arguments
- Ranked results are sorted from smallest to largest variance of relative factor change before selecting the top 20
- For the McLoughlin dataset, stable genes such as RPL9 and RPL31 appear in the output, consistent with genes ERgene found similar to NormFinder and geNorm (Table 2)

## Limitations

- Computational efficiency begins to decline at depth 3 or larger, and a very large computational depth may produce no result
- Computational depth set too large may yield no result at all
- Single reference gene normalization (normalizationdata) is not acceptable per MIQE guidelines unless clear evidence of invariable expression is provided
- No ERgene software version or pip/conda installation instructions are stated in the article text (README indicates version 1.2.0 and `pip install ERgene`)

## Evidence

- [discussion] Screen internal reference genes with ERgene.Ergene.Finderg(Data, Depth): "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [discussion] Block and ranking mechanism: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] Top-20 selection: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] Depth-2 runtime on 1968 genes: [evidence span withheld — anchor status: not_found]
- [discussion] Cross-method agreement: "the internal reference genes found by ERgene, NormFinder and geNorm were similar (Table 2)"
- [readme] README example invocation: "import ERgene
import pandas as pd
data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2
ERgene.FindERG(data,3)"
