---
name: relative-correction-factor-matrix-computation
description: Use when a gene expression matrix (Pandas DataFrame, genes × samples)
  must be screened for stable endogenous reference genes by replicating ERgene's inverse
  internal-reference calculation rather than treating the library as a black box.
when_to_use_negative:
- The goal is gene expression normalization of data by an already-identified reference
  gene — use Ergene.Normalizationdata(Data, Ergname) instead of re-deriving the F1/F2
  matrices
- The dataset is very small and shallow reference-gene screening via the ERgene library
  directly (FindERG) is sufficient, making a reimplementation unnecessary
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ERgene
  role: Python library whose exported FindERG function implements the same inverse
    internal-reference screening; used as ground-truth comparison for the reimplemented
    top-20 list
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame format required by the ERgene screening interface and
    the matrix/variance operations for the F1, F2, and ΔF computation
- name: Python
  role: Implementation language for the reimplemented correction-factor-matrix algorithm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:f975ddaf7cd48a98db4cd80058d5757e91264990a63cecf897d5b5c5bb8e2aea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# relative-correction-factor-matrix-computation

## Summary

Reimplements the inverse-of-internal-reference screening at the core of the ERgene Python library: per-sample relative correction factor matrices F1 and F2 are built, their difference ΔF is computed, and per-gene variance of ΔF ranks candidate endogenous reference genes (top 20 returned).

## When to use

Use when a gene expression matrix (Pandas DataFrame, genes × samples) must be screened for stable endogenous reference genes by replicating ERgene's inverse internal-reference calculation rather than treating the library as a black box.

## When NOT to use

- The goal is gene expression normalization of data by an already-identified reference gene — use Ergene.Normalizationdata(Data, Ergname) instead of re-deriving the F1/F2 matrices
- The dataset is very small and shallow reference-gene screening via the ERgene library directly (FindERG) is sufficient, making a reimplementation unnecessary

## Inputs

- Gene expression matrix as a Pandas DataFrame (e.g. a tab-separated GSE file such as gse.txt loaded with pd.read_csv(sep='\t') and set_index on the first column)
- Computational depth parameter (Depth, e.g. 2 or 3) determining the number of samples selected for the computation

## Outputs

- Ranked list of the top 20 candidate internal reference genes sorted by ascending ΔF variance
- Ranked top-20 list written to top20_reference_genes.csv for comparison against ERgene's FindERG output

## How to apply

Load the expression matrix into a Pandas DataFrame (index set on the first column, as ERgene Ver1.2 requires) and reconstruct the inverse-of-internal-reference calculation: build the per-sample relative correction factor matrices F1 and F2 and compute their difference ΔF. Compute the per-gene variance of ΔF across samples as the stability measure, applying the robust block computation in blocks of 1000 genes for large datasets. Sort genes by ΔF variance from smallest to largest and take the top 20 as candidate reference genes; validate by running ERgene.Ergene.Finderg(Data, Depth) (exported as FindERG in v1.2.0) on the identical input and diffing the ranked lists. The computational Depth parameter controls how many samples are paired for the computation — depth 2 screened 1968 genes in 4.58 s, while depth ≥ 3 slows efficiency and very large depths may return no result. Write the ranked top-20 list to a CSV (e.g. top20_reference_genes.csv) for comparison.

## Related tools

- **ERgene** (Python library whose exported FindERG function implements the same inverse internal-reference screening; used as ground-truth comparison for the reimplemented top-20 list) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame format required by the ERgene screening interface and the matrix/variance operations for the F1, F2, and ΔF computation)
- **Python** (Implementation language for the reimplemented correction-factor-matrix algorithm)

## Examples

```
import ERgene, pandas as pd; data = pd.read_csv('gse.txt', sep='\t').set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- The reimplemented top-20 list matches the output of ERgene.Ergene.Finderg / ERgene.FindERG run on the identical input DataFrame
- Genes are ordered by ΔF variance ascending and exactly 20 genes are returned
- For large gene sets, the block computation processes genes in blocks of 1000 as specified
- Runtime is consistent with reported performance (e.g. depth 2 on 1968 genes took ~4.58 s)
- Output CSV (top20_reference_genes.csv) contains the ranked gene list and parses correctly

## Limitations

- When the computational depth is set too large, there may be no result; efficiency begins to decline at depth 3 or larger
- Single-gene normalization based on any one returned reference gene is not acceptable per MIQE guidelines unless invariable expression is demonstrated — ERgene's own normalizationdata function must be used with caution
- A result may not occur at too large a depth, and efficiency of the matrix block approach can be sensitive to dataset size

## Evidence

- [discussion] Block calculation: every 1000 genes taken as a block for large datasets: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [discussion] Compute variance of relative factor change and return top 20 genes: "then sort the results of variation from smallest to largest. The program will return the top 20 genes as a result."
- [abstract] Inverse internal-reference principle and robust block operation: "The principle is based on the inverse process of the internal reference method, and the robust matrix block operation makes the selection of internal reference genes faster than any other method."
- [discussion] Computational depth behavior: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [readme] README invocation of FindERG on a TSV-loaded DataFrame: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792"
