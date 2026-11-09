---
name: python-bioinformatics-package-execution
description: Use when a task requires running an installed Python package's exported
  functions (e.g., ERgene.FindERG / ERgene.normalizationdata) on gene expression data
  loaded into a Pandas DataFrame and verifying the outputs.
when_to_use_negative:
- The data are not gene expression values in DataFrame format, or the goal is normalization
  validation per MIQE guidelines where single-gene normalization is unacceptable without
  clear invariability evidence
- The computational depth is set too large (≥3 or higher for big datasets), where
  ERgene's efficiency declines and it may return no result
- The task requires an R- or Excel-based tool such as geNorm (qbase+/Excel), Normfinder
  (R/Excel), or BestKeeper instead of a Python package
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
tools:
- name: ERgene
  role: The executed package; its exported FindERG and normalizationdata functions
    screen reference genes and standardize expression data
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Used to load the expression dataset into the DataFrame format required by
    FindERG
- name: Python
  role: Runtime environment in which the package is installed via pip and invoked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:cded92f9a761aaa872410fe4d05b3b60bc16d91e0e32b077618d1df2e0630872
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# python-bioinformatics-package-execution

## Summary

Execute a pip-installable Python bioinformatics package (ERgene, built on Pandas) from a research task to screen candidate endogenous reference genes and perform single-reference-gene normalization on gene expression DataFrames.

## When to use

Use when a task requires running an installed Python package's exported functions (e.g., ERgene.FindERG / ERgene.normalizationdata) on gene expression data loaded into a Pandas DataFrame and verifying the outputs.

## When NOT to use

- The data are not gene expression values in DataFrame format, or the goal is normalization validation per MIQE guidelines where single-gene normalization is unacceptable without clear invariability evidence
- The computational depth is set too large (≥3 or higher for big datasets), where ERgene's efficiency declines and it may return no result
- The task requires an R- or Excel-based tool such as geNorm (qbase+/Excel), Normfinder (R/Excel), or BestKeeper instead of a Python package

## Inputs

- Gene expression dataset as a Pandas DataFrame (rows=genes with first column as index, columns=samples), e.g., tab-separated GSE file read via pd.read_csv('gse.txt', sep='\t')
- Computational depth parameter for ERgene.FindERG (integer, e.g., 2 or 3)
- Name of a candidate reference gene for normalization (e.g., 'Atp1a3')

## Outputs

- Ranked list of up to 20 candidate endogenous reference genes (smallest variation of relative factor first), e.g., ['ASHGV40057862', ...]
- Normalized expression DataFrame returned by ERgene.normalizationdata
- UpSet plot of candidate genes when run in Jupyter notebook
- Results written to a summary CSV (e.g., normalization_results.csv)

## How to apply

Install the package from PyPI (e.g., `pip install ERgene`, version 1.2.0+ with ~100x speedup) along with its dependencies (Numpy, pandas, itertools, upsetplot). Load the expression data (e.g., the aging mouse brain SHED vs salt-injection example from the ERgene repository's example folder, or a GSE tab-separated file) into a Pandas DataFrame and set the first column as index, since FindERG expects DataFrame input. Call the package's exported screening function (ERgene.FindERG(data, depth)), which computes the variation of the relative factor, sorts results from smallest to largest, and returns the top 20 candidate reference genes; use depth 2 for speed (4.58 s for 1968 genes) and depth ~3 for larger datasets (17.3 s on GSE125792), since depth ≥3 begins to decline in efficiency. Then call the exported normalization function (ERgene.normalizationdata(data, gene_name)) with an identified reference gene (e.g., Atp1a3 for the mouse brain dataset) and compare before/after spectral density or boxplots to confirm normalization reduced between-group differences.

## Related tools

- **ERgene** (The executed package; its exported FindERG and normalizationdata functions screen reference genes and standardize expression data) — github.com/Starlitnightly/ERgene
- **Pandas** (Used to load the expression dataset into the DataFrame format required by FindERG)
- **Python** (Runtime environment in which the package is installed via pip and invoked)

## Examples

```
import ERgene, pandas as pd; data=pd.read_csv('gse.txt',sep='\t'); data=data.set_index(data.columns[0]); ERgene.FindERG(data,3); ans=ERgene.normalizationdata(data,'Atp1a3')
```

## Evaluation signals

- FindERG's ranked output contains the expected candidate reference gene (e.g., Atp1a3 for the aging mouse brain SHED vs salt dataset) within the top 20 returned genes
- Screening prints a runtime line (README example: 'calculate time:17.3s' at depth 3 on GSE125792) and returns a Python list of gene identifiers
- normalizationdata yields between-group (SHED vs salt) spectral-density/boxplot differences smaller than in the unnormalized data (Fig. 1a-d pattern)
- Input DataFrame has its first column set as the index (required in ERgene version 1.2+)
- No empty/missing result when using an appropriate depth; depth ≥3 on large datasets is the known failure mode to check

## Limitations

- Single-gene standardization is not desirable per the literature and MIQE guidelines unless invariable expression is clearly demonstrated; use normalizationdata with caution
- Computational efficiency declines at depth 3 or larger, and setting the depth too large may produce no result
- No ERgene software version number is stated in the article text; the README references version 1.2.0 for the 100x speedup and the set_index requirement
- For very large datasets, block calculation is adopted with every 1000 genes taken as a block

## Evidence

- [discussion] FindERG expects DataFrame input and returns top 20 genes: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [discussion] Sorting and top-20 return behavior: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] Depth 2 runtime benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] Single reference gene caution (MIQE): "it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer"
- [readme] README invocation pattern with index requirement: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
	data=data.set_index(data.columns[0])#This is Ver1.2
	ERgene.FindERG(data,3)"
- [readme] README normalization call: "ans=ERgene.normalizationdata(data,'ASHGV40056316')"
- [full_text] Article workflow validation (Atp1a3 candidate and reduced spectral density difference): [evidence span withheld — anchor status: not_found]
