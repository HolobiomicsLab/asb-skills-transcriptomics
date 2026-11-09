---
name: sample-distribution-boxplot-comparison
description: Use when you have probe-level expression matrices (e.g. GEO datasets
  like GSE4786) and need to visually confirm that normalization to an internal reference
  gene reduced per-sample distribution differences.
when_to_use_negative:
- Data is single-cell or already quantile/RMA-normalized so per-sample spectral differences
  are not expected to exist
- The goal is qPCR normalization under MIQE guidelines requiring multiple validated
  reference genes, since single-gene standardization is not acceptable without clear
  evidence of invariable expression
edam_operation: http://edamontology.org/operation_0575
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ERgene
  role: Provides FindERG to select the internal reference probe and normalizationdata
    to standardize the expression matrix before and after which the boxplot/density
    comparison is made
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Holds the probe-level expression matrix as a DataFrame used as input to ERgene
    functions and to per-sample plotting
- name: Python
  role: Runtime environment for ERgene and for generating the pre/post boxplots and
    density plots
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:d4071ca57784e36e748691689c562e78bb7a0c0e53f2bdb030759269cd4872af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_0575
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sample-distribution-boxplot-comparison

## Summary

Verifies microarray normalization quality by generating per-sample boxplots and density plots of probe-level expression before and after standardizing to an ERgene-selected internal reference gene (e.g. 1439423_x_at on GSE4786). Correct normalization narrows the sample spectral differences visible across the plots.

## When to use

Use when you have probe-level expression matrices (e.g. GEO datasets like GSE4786) and need to visually confirm that normalization to an internal reference gene reduced per-sample distribution differences.

## When NOT to use

- Data is single-cell or already quantile/RMA-normalized so per-sample spectral differences are not expected to exist
- The goal is qPCR normalization under MIQE guidelines requiring multiple validated reference genes, since single-gene standardization is not acceptable without clear evidence of invariable expression

## Inputs

- Unnormalized GEO microarray probe-level expression matrix (e.g. GSE4786) as a pandas DataFrame with probes as rows and samples as columns
- Normalized expression matrix produced by ERgene.normalizationdata with a single reference probe (e.g. Ergname='1439423_x_at')

## Outputs

- Per-sample boxplots of expression distributions before and after normalization (pre: Fig. 2a-style; post: Fig. 2c-style)
- Per-sample density plots before and after normalization (pre: Fig. 2b-style; post: Fig. 2d-style)
- Combined comparison figure (e.g. normalization_comparison.png) plus the FindERG top-20 reference gene table

## How to apply

Load the raw probe-level expression matrix (e.g. from GEO GSE4786 mouse cochlea microarray data) into a pandas DataFrame and generate per-sample boxplots and density plots to record the pre-normalization spectral spread (analogous to Fig. 2a,b). After running ERgene.FindERG to select the reference probe (e.g. 1439423_x_at), standardize the matrix with ERgene.normalizationdata(Data, Ergname='1439423_x_at'). Regenerate the boxplots and density plots on the normalized data (Fig. 2c,d) and compare pre/post pairs: success is judged when the across-sample spectral difference visibly narrows, i.e. sample medians and distributions converge. Save the FindERG top-20 reference gene table and the comparison plots (e.g. as normalization_comparison.png) as evidence of the verification.

## Related tools

- **ERgene** (Provides FindERG to select the internal reference probe and normalizationdata to standardize the expression matrix before and after which the boxplot/density comparison is made) — github.com/Starlitnightly/ERgene
- **Pandas** (Holds the probe-level expression matrix as a DataFrame used as input to ERgene functions and to per-sample plotting)
- **Python** (Runtime environment for ERgene and for generating the pre/post boxplots and density plots)

## Examples

```
import ERgene, pandas as pd; data=pd.read_csv('gse4786.txt',sep='\t').set_index(data.columns[0]); ERgene.FindERG(data,2); norm=ERgene.normalizationdata(data,'1439423_x_at')
```

## Evaluation signals

- Across-sample spectral difference narrows after normalization: post-normalization boxplots show converging sample medians and distributions versus the pre-normalization spread
- Density plots shift to aligned, near-overlapping per-sample curves after standardization to the reference probe
- The reference probe used (e.g. 1439423_x_at) appears in the FindERG top-20 output table
- Comparison figure saved (e.g. normalization_comparison.png) with matched pre/post panels corresponding to Fig. 2a-d ordering
- Normalized matrix returned by normalizationdata has the same probes/samples dimensions as the input DataFrame

## Limitations

- Single reference gene normalization is not acceptable per MIQE guidelines unless the investigator provides clear evidence of invariable expression
- Comparison is visual (boxplot/density inspection); no quantitative threshold for 'narrowed spectral difference' is defined in the article
- Computational depth set too large in FindERG may yield no result, preventing downstream normalization and comparison

## Evidence

- [other] Using the ERgene-selected reference gene 1439423_x_at to normalize GSE4786 narrowed the sample spectral difference, as shown by comparing boxplots and density plots: [evidence span withheld — anchor status: not_found]
- [discussion] This function is used for the standardization of a single internal reference gene: "This function is used for the standardization of a single internal reference gene"
- [discussion] it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer: "it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer"
- [readme] You only need to input the gene to normalizate.: "You only need to input the gene to normalizate"
- [methods] FindERG export: "### Exported symbols
- `FindERG`
- `normalizationdata`"
