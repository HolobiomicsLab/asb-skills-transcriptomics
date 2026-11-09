---
name: gene-overlap-across-sample-pairs
description: Use when you have ranked candidate reference gene lists from ERgene FindERG
  over multiple sample pairs of one dataset (e.g., 20 genes per pair) and need to
  quantify which genes recur across pairs.
when_to_use_negative:
- When the task is single-dataset reference gene ranking only, without robustness
  analysis across sample pairs or samples — a single FindERG call suffices.
- When normalizing already-selected reference genes; use ERgene.normalizationdata
  instead, and note single-gene normalization is discouraged per MIQE guidelines.
- When the input is raw sequencing or probe-unconverted data ERgene was not designed
  to ingest without conversion to an expression matrix.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ERgene
  role: Provides the exported FindERG function used per sample pair to return the
    top 20 candidate reference genes ranked by smallest variance of relative factor
    change; generates an UpSet plot in Jupyter
  repo: github.com/Starlitnightly/ERgene
- name: Python
  role: Execution environment for the ERgene library and the overlap/duplication-rate
    computation scripts
- name: Pandas
  role: Loads the GEO expression matrix into the DataFrame format ERgene requires
    (index set to the first column, per ERgene v1.2)
- name: upsetplot
  role: ERgene dependency used to render the pairwise set intersections of screened
    genes as upsetplot.png
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:d93d8425864d42d0754341de236088fd1249e641840fe5ce8c074f07c8065c72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  - upsetplot
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-overlap-across-sample-pairs

## Summary

Assess robustness of ERgene-screened candidate internal reference genes by computing each gene's coverage across all sample pairs (e.g., 66 pairs from 12 GSE125792 samples), visualizing pairwise set intersections as an UpSet plot, and checking that >80% coverage in ≥10 genes and an overall duplication rate >50%.

## When to use

Use when you have ranked candidate reference gene lists from ERgene FindERG over multiple sample pairs of one dataset (e.g., 20 genes per pair) and need to quantify which genes recur across pairs.

## When NOT to use

- When the task is single-dataset reference gene ranking only, without robustness analysis across sample pairs or samples — a single FindERG call suffices.
- When normalizing already-selected reference genes; use ERgene.normalizationdata instead, and note single-gene normalization is discouraged per MIQE guidelines.
- When the input is raw sequencing or probe-unconverted data ERgene was not designed to ingest without conversion to an expression matrix.

## Inputs

- Gene expression matrix in tab-separated text format (e.g., gse.txt from GSE125792), first column used as index, loaded into a pandas DataFrame
- Enumerated sample pairs from the dataset (e.g., 66 pairs from 12 samples)
- ERgene computational depth parameter (e.g., depth=2 or 3)

## Outputs

- Per-sample-pair ranked list of top 20 candidate internal reference genes
- Per-gene sample-pair coverage fractions across the 66 pairs
- Overall duplication rate of repeated gene selections across pairs
- UpSet plot image (upsetplot.png) of pairwise set intersections of screened genes

## How to apply

Retrieve the expression matrix (e.g., GSE125792, 12 samples) from GEO and load it into a pandas DataFrame, the format required by ERgene. For each of the 66 sample pairs, invoke ERgene's exported FindERG function with the chosen computational depth (e.g., depth 2 or 3) to obtain the top 20 candidate reference genes ranked by smallest variance of relative factor change. Aggregate per-pair gene lists, compute each gene's sample-pair coverage, and flag genes covered in more than 80% of the pairs; also compute the overall duplication rate as the frequency of repeated gene selections across pairs and verify it exceeds 50%. Render the pairwise set intersections with the upsetplot library (a dependency of ERgene) and save as upsetplot.png for inspection.

## Related tools

- **ERgene** (Provides the exported FindERG function used per sample pair to return the top 20 candidate reference genes ranked by smallest variance of relative factor change; generates an UpSet plot in Jupyter) — github.com/Starlitnightly/ERgene
- **Python** (Execution environment for the ERgene library and the overlap/duplication-rate computation scripts)
- **Pandas** (Loads the GEO expression matrix into the DataFrame format ERgene requires (index set to the first column, per ERgene v1.2))
- **upsetplot** (ERgene dependency used to render the pairwise set intersections of screened genes as upsetplot.png)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t').set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- At least 10 genes show sample-pair coverage above 80% in the UpSet plot of the 66 sample-pair overlap.
- The overall duplication rate of selected candidate reference genes remains above 50% when computational depth is increased.
- upsetplot.png exists and is a readable image file.
- Each per-pair FindERG call returns exactly the top 20 genes as documented for the ERgene workflow.
- Recomputed gene lists are consistent with per-pair ERgene output ordering by smallest variance of the relative factor change.

## Limitations

- Computational efficiency declines at depth 3 or larger, and no result may be produced when the computational depth is set too large.
- ERgene may be slow on very large gene sets unless block calculation (every 1000 genes per block) is used.
- Single reference gene normalization is not acceptable per MIQE guidelines unless invariable expression is clearly demonstrated.
- The example pipeline was demonstrated on a specific dataset (GSE125792); coverage and duplication thresholds (>80%, >50%) were validated there and may not transfer directly to other datasets.

## Evidence

- [results] Ten genes showed sample-pair coverage of more than 80% in the Upsetplot of the 66 sample-pair overlap: [evidence span withheld — anchor status: not_found]
- [results] the duplication rate of selected candidate internal reference genes remained above 50%: [evidence span withheld — anchor status: not_found]
- [results] Retrieve the GSE125792 expression matrix (12 samples) from GEO and load it into a pandas DataFrame: [evidence span withheld — anchor status: not_found]
- [discussion] sort the results of variation from smallest to largest. The program will return the top 20 genes: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [readme] data=pd.read_csv('gse.txt',sep='\t')#GSE125792
	data=data.set_index(data.columns[0]): "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
	data=data.set_index(data.columns[0])#This is Ver1.2"
