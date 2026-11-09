---
name: set-intersection-upsetplot-visualization
description: Use when you have per-sample-pair or per-run lists of selected candidate
  reference genes and need to quantify pairwise set overlap and duplication rate,
  flagging genes covered in more than 80% of the pairs.
when_to_use_negative:
- When only a single reference gene is to be normalized without any set-overlap analysis
  — single gene standardization is discouraged and ERgene's normalizationdata is a
  separate concern
- When the goal is differential normalization benchmarking against geNorm/NormFinder/BestKeeper
  rather than visualizing set intersections of ERgene candidate outputs
- When computational depth is set very large — ERgene may return no result, leaving
  no gene sets to intersect
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ERgene
  role: Provides FindERG to screen candidate reference genes per sample pair; renders
    the UpSet plot of set intersections when run in Jupyter
  repo: github.com/Starlitnightly/ERgene
- name: upsetplot
  role: Python dependency of ERgene 1.2.0 used to draw the UpSet plot of pairwise
    gene-set intersections
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Loads the tab-separated GSE125792 expression matrix into the DataFrame format
    required by FindERG
- name: Python
  role: Runtime language for the ERgene-based screening and coverage aggregation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:21990212eac0b1757ac137df0969339267a6e0733d8feb09aef1df7632e5ddfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  - upsetplot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# set-intersection-upsetplot-visualization

## Summary

Visualizes pairwise/multi-way set intersections of candidate internal reference genes screened by ERgene across sample pairs (e.g. 20 candidates over 66 sample pairs from 12 GSE125792 samples) as an UpSet plot, exposing which genes are consistently selected across pairs.

## When to use

Use when you have per-sample-pair or per-run lists of selected candidate reference genes and need to quantify pairwise set overlap and duplication rate, flagging genes covered in more than 80% of the pairs.

## When NOT to use

- When only a single reference gene is to be normalized without any set-overlap analysis — single gene standardization is discouraged and ERgene's normalizationdata is a separate concern
- When the goal is differential normalization benchmarking against geNorm/NormFinder/BestKeeper rather than visualizing set intersections of ERgene candidate outputs
- When computational depth is set very large — ERgene may return no result, leaving no gene sets to intersect

## Inputs

- GSE125792 expression matrix from GEO as tab-separated text (gse.txt, sep='\t') loaded into a pandas DataFrame with the first column as index
- Per-pair top-20 candidate reference gene lists from ERgene.FindERG (66 sample pairs from 12 samples)
- Computational depth parameter (depth 2 in the overlap analysis; depth 3 in the README example)

## Outputs

- UpSet plot image of pairwise set intersections of screened genes (upsetplot.png), auto-rendered in Jupyter notebooks
- Per-gene sample-pair coverage table flagging genes covered in more than 80% of the 66 pairs
- Overall duplication rate across pairs (expected >50%)
- Ranked top-20 candidate reference gene list per pair

## How to apply

Retrieve the GSE125792 expression matrix (12 samples) from GEO as a tab-separated file and load it into a pandas DataFrame, the format ERgene requires. For each of the 66 sample pairs, call ERgene's exported FindERG function on the expression data to obtain the top 20 candidate reference genes ranked by smallest variance of the relative factor change. Aggregate the per-pair gene lists, compute the sample-pair coverage for each candidate gene (flagging genes covered in >80% of the 66 pairs), and compute the overall duplication rate as the frequency of repeated gene selections across pairs, verifying it exceeds 50%. Render the pairwise set intersections of the screened genes as an UpSet plot using the upsetplot library (a dependency of ERgene 1.2.0) and save it, e.g. as upsetplot.png; ERgene automatically generates this UpSet plot when run in a Jupyter notebook.

## Related tools

- **ERgene** (Provides FindERG to screen candidate reference genes per sample pair; renders the UpSet plot of set intersections when run in Jupyter) — github.com/Starlitnightly/ERgene
- **upsetplot** (Python dependency of ERgene 1.2.0 used to draw the UpSet plot of pairwise gene-set intersections) — github.com/Starlitnightly/ERgene
- **Pandas** (Loads the tab-separated GSE125792 expression matrix into the DataFrame format required by FindERG)
- **Python** (Runtime language for the ERgene-based screening and coverage aggregation workflow)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- At least 10 genes show sample-pair coverage of more than 80% in the UpSet plot of the 66 sample-pair overlap
- Overall duplication rate of selected candidate reference genes remains above 50% when computational depth is increased
- upsetplot.png exists and is a readable image file
- Each per-pair call to FindERG returns exactly the top 20 genes sorted by smallest variance of the relative factor change
- Number of sample pairs equals C(12,2) = 66 for a 12-sample expression matrix

## Limitations

- Computational efficiency declines at depth 3 or larger, and when computational depth is set too large there may be no result at all
- Single reference gene normalization is not acceptable per MIQE guidelines unless invariable expression is demonstrated, so UpSet-derived singletons should be used with caution
- The UpSet plot is only auto-generated when ERgene runs in a Jupyter notebook

## Evidence

- [results] Ten genes showed sample-pair coverage of more than 80% in the Upsetplot of the 66 sample-pair overlap: [evidence span withheld — anchor status: not_found]
- [readme] ERgene aggregates per-pair lists and renders UpSet intersections: "if you use jupyter notebook, there is an Upsetplot will be generated."
- [discussion] FindERG returns top-20 genes sorted by variance of relative factor change: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [discussion] Input must be a pandas DataFrame expression matrix: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [readme] ERgene depends on upsetplot for rendering set intersections: "Dependencies
------
- Numpy
- time
- pandas
- itertools
- upsetplot"
