---
name: internal-reference-gene-normalization
description: Use when a raw microarray or gene-expression DataFrame (e.g., GEO series
  such as GSE4786) needs an empirically selected endogenous reference gene for single-gene
  normalization and before/after distribution comparison.
when_to_use_negative:
- Single-gene normalization is not acceptable per MIQE guidelines unless invariable
  expression is clearly demonstrated — do not skip the FindERG screening step or cite
  single-gene results without evidence
- Do not set computational depth too large (depth 3+ declines efficiency; too large
  a depth may return no result)
- Not applicable when the goal is multi-reference-gene normalization pipelines like
  geNorm, NormFinder, or BestKeeper — this skill covers ERgene-based screening and
  single-gene standardization only
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ERgene
  role: Python library providing FindERG for screening internal reference genes and
    normalizationdata for single-reference-gene standardization
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Used to load and index the probe-level expression matrix before screening
- name: Python
  role: Runtime environment; ERgene depends on Numpy, pandas, itertools, upsetplot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:15202290313e0b6d6a421b24c45fc7767024e2831eac2e9a524ca8f067e50792
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# internal-reference-gene-normalization

## Summary

Screen endogenous internal reference genes from a probe-level expression matrix with the ERgene Python library, then standardize the matrix to the selected reference probe (e.g., 1439423_x_at on GSE4786) to narrow sample spectral differences.

## When to use

Use when a raw microarray or gene-expression DataFrame (e.g., GEO series such as GSE4786) needs an empirically selected endogenous reference gene for single-gene normalization and before/after distribution comparison.

## When NOT to use

- Single-gene normalization is not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated — do not skip the FindERG screening step or cite single-gene results without evidence
- Do not set computational depth too large (depth 3+ declines efficiency; too large a depth may return no result)
- Not applicable when the goal is multi-reference-gene normalization pipelines like geNorm, NormFinder, or BestKeeper — this skill covers ERgene-based screening and single-gene standardization only

## Inputs

- Probe-level gene expression matrix as a pandas DataFrame (e.g., tab-separated GEO file like gse.txt from GSE4786 or GSE125792, first column set as index)
- Computational depth parameter (e.g., Depth=2, as used in the GSE4786 workflow)
- Selected reference gene/probe name (e.g., '1439423_x_at') for normalizationdata

## Outputs

- Top-20 internal reference gene list/table from FindERG (plus an Upsetplot in Jupyter)
- Normalized expression matrix returned by ERgene.normalizationdata
- Before/after boxplot and density comparison figures (e.g., normalization_comparison.png)

## How to apply

Load the probe-level expression matrix into a pandas DataFrame (set the first column as index). Run ERgene.FindERG(Data, Depth=2); for large matrices the function applies robust block calculation with every 1000 genes as one block, sorts variance of relative factor change from smallest to largest, and returns the top 20 candidate reference genes. Confirm the expected probe appears in the FindERG output (e.g., 1439423_x_at for the Someya mouse cochlea dataset GSE4786). Standardize with ERgene.normalizationdata(Data, Ergname='<selected probe>'). Generate boxplots and density plots comparing raw vs. normalized per-sample distributions to verify sample spectral differences narrowed; use Depth=3 only with caution since efficiency declines at depth 3+ and no result may occur at too large depth.

## Related tools

- **ERgene** (Python library providing FindERG for screening internal reference genes and normalizationdata for single-reference-gene standardization) — github.com/Starlitnightly/ERgene
- **Pandas** (Used to load and index the probe-level expression matrix before screening)
- **Python** (Runtime environment; ERgene depends on Numpy, pandas, itertools, upsetplot)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t').set_index(data.columns[0]); ERgene.FindERG(data, 2); ans = ERgene.normalizationdata(data, '1439423_x_at')
```

## Evaluation signals

- The expected reference probe (e.g., 1439423_x_at for GSE4786) appears in the FindERG top-20 output
- Boxplots and density plots show narrower per-sample spectral differences after normalizationdata compared to raw distributions (cf. Fig. 2a,b before vs. Fig. 2c,d after)
- FindERG returns exactly the top 20 genes sorted by smallest variance of relative factor change
- For large matrices, block calculation triggered with every 1000 genes per block; runtime should remain tractable (e.g., 1968 genes at depth 2 took 4.58 s)
- Normalized matrix dimensions match the input DataFrame and contain values scaled relative to the chosen reference probe

## Limitations

- Single reference gene normalization is not acceptable per MIQE guidelines unless invariable expression is demonstrated; the README warns 'this method should be used with Caution'
- When computational depth is set to 3 or larger, efficiency declines, and at too large a depth there may be no result
- No ERgene software version stated in the article text (README mentions v1.2.0 with ~100x speed increase); installation instructions and dataset accession statements are absent from the article body

## Evidence

- [discussion] FindERG output for GSE4786: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] block calculation: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] depth efficiency: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [discussion] single-gene caution: "it is not acceptable to normalize a single internal reference gene unless the investigator provides clear evidence to the reviewer"
- [readme] README normalization usage: "ans=ERgene.normalizationdata(data,'ASHGV40056316')"
- [readme] README FindERG usage: "ERgene.FindERG(data,3)"
