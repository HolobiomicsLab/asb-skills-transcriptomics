---
name: variance-based-candidate-gene-ranking
description: Use when a gene expression matrix (DataFrame, e.g. tab-separated GSE
  text or lab mouse brain SHED vs salt data) needs a stably expressed internal reference
  gene for normalization and no Python-based screening method has been applied yet.
when_to_use_negative:
- Single-gene normalization is not acceptable per MIQE guidelines unless the investigator
  provides clear evidence of invariable expression — do not rely on this alone for
  final qPCR normalization without such justification
- Computational depth set too large (depth 3 or larger shows declining efficiency;
  very large depths may yield no result) — avoid oversetting depth on large datasets
- Data not in DataFrame format or index not set (FindERG expects DataFrame input with
  gene identifiers indexed, required for ERgene version 1.2+)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3572
tools:
- name: ERgene
  role: Python library providing FindERG for screening internal reference genes and
    normalizationdata for standardizing with a single reference gene
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame format expected by FindERG and normalizationdata for
    loading expression data
- name: Python
  role: Runtime environment; ERgene is installed via pip install ERgene (version 1.2.0
    gives ~100x speed increase)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:2f837deae2b6d986ae9ea35c75ef85343315e53fd52e5762aa94046eb30304f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# variance-based-candidate-gene-ranking

## Summary

Screen candidate endogenous reference genes by computing the variation of the relative factor with ERgene's FindERG, ranking results smallest-to-largest, and returning the top 20 genes, then normalizing expression data with the selected single reference gene via normalizationdata.

## When to use

Use when a gene expression matrix (DataFrame, e.g. tab-separated GSE text or lab mouse brain SHED vs salt data) needs a stably expressed internal reference gene for normalization and no Python-based screening method has been applied yet.

## When NOT to use

- Single-gene normalization is not acceptable per MIQE guidelines unless the investigator provides clear evidence of invariable expression — do not rely on this alone for final qPCR normalization without such justification
- Computational depth set too large (depth 3 or larger shows declining efficiency; very large depths may yield no result) — avoid oversetting depth on large datasets
- Data not in DataFrame format or index not set (FindERG expects DataFrame input with gene identifiers indexed, required for ERgene version 1.2+)

## Inputs

- Gene expression matrix as a Pandas DataFrame (genes x samples), e.g. read from a tab-separated file with data.set_index(data.columns[0])
- Computational depth parameter (integer; README example uses depth 3)
- Name of the selected reference gene for normalization (e.g. 'ASHGV40056316' or 'Atp1a3')

## Outputs

- Ranked list of top 20 candidate reference genes (variation of relative factor sorted smallest to largest)
- Optional Upsetplot visualization when run in Jupyter notebook
- Normalized expression DataFrame returned by normalizationdata
- Screening ranking and normalization verification results (e.g. normalization_results.csv)

## How to apply

Load the expression data into a Pandas DataFrame and set the first column as index (required from ERgene version 1.2). Run ERgene.FindERG(data, depth), optionally choosing a computational depth (e.g. 3, as in the README example on GSE125792); the function computes the variation of the relative factor, sorts results from smallest to largest, and returns the top 20 candidate reference genes. Inspect the ranked list to confirm the expected candidate (e.g. Atp1a3 for the lab mouse brain dataset). Then call ERgene.normalizationdata(data, 'GeneName') to standardize with the single reference gene and compare between-group differences (e.g. SHED vs salt injection spectral density) before and after normalization to verify reduction. Note the article's caution: single-gene normalization is not acceptable per MIQE guidelines unless invariable expression is demonstrated, and block calculation (every 1000 genes per block) applies to very large datasets.

## Related tools

- **ERgene** (Python library providing FindERG for screening internal reference genes and normalizationdata for standardizing with a single reference gene) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame format expected by FindERG and normalizationdata for loading expression data)
- **Python** (Runtime environment; ERgene is installed via pip install ERgene (version 1.2.0 gives ~100x speed increase))

## Examples

```
import ERgene; import pandas as pd; data=pd.read_csv('gse.txt',sep='\t'); data=data.set_index(data.columns[0]); ERgene.FindERG(data,3)
```

## Evaluation signals

- FindERG's ranked output contains the expected candidate reference gene (e.g. Atp1a3 for the lab mouse brain SHED vs salt dataset)
- normalizationdata yields a smaller between-group (SHED vs salt) sample/spectral-density difference than the raw data, verifiable by before/after boxplots and density plots
- FindERG returns a ranked list capped at the top 20 genes, sorted by smallest variation of the relative factor
- Runtime is consistent with reported efficiency: screening 1968 genes at depth 2 took 4.58 s; depth 3 on GSE125792 took ~17.3 s
- In Jupyter, an Upsetplot of overlapping candidate sets is generated, providing a visual sanity check

## Limitations

- When computational depth is set to 3 or larger, efficiency declines; setting depth too large may produce no result
- Single reference gene normalization is not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated; the README reiterates this should be used with caution
- No ERgene software version stated in the article (README notes v1.2.0 changed indexing and ~100x speed); installation/data-access details are not in the article text
- When the gene dataset is too large, block calculation (every 1000 genes per block) is adopted, which may affect behavior on very large inputs

## Evidence

- [methods] variation ranking: "### Exported symbols
- `FindERG`
- `normalizationdata`"
- [discussion] depth efficiency: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [discussion] top 20 output: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] MIQE caution: "it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer"
- [readme] README usage: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2
ERgene.FindERG(data,3)"
- [discussion] DataFrame input: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
