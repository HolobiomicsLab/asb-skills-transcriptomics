---
name: matrix-block-computation
description: Use when screening endogenous reference genes over a large gene expression
  matrix (thousands of genes, e.g. GSE125792) and full-matrix pairwise computation
  would be too slow or depth-3+ screening risks no result.
when_to_use_negative:
- Normalization of a specific known gene is the goal — use ERgene.normalizationdata
  instead; single-gene standardization is documented as undesirable and should be
  used with caution per MIQE guidance
- The expression matrix is small enough that full-matrix screening is fast — blocking
  adds no benefit and depth 3+ already declines in efficiency
- Computational depth is set too large — the paper warns there may be no result at
  all
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ERgene
  role: Python library implementing the robust matrix block operation and inverse
    internal-reference screening (FindERG)
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame input format required by FindERG and the computational
    power underlying the library
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:5488621fc6b9c34d811f4df61eded5553283f37eb6ada6d5c1c0121b68b8c617
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# matrix-block-computation

## Summary

ERgene's robust matrix block operation partitions large gene expression DataFrames into blocks of 1000 genes so the inverse internal-reference screening (variance of relative factor change) stays computationally efficient. It is the mechanism that lets FindERG screen datasets like GSE125792 (1968 genes) in seconds at computational depth 2 or 3.

## When to use

Use when screening endogenous reference genes over a large gene expression matrix (thousands of genes, e.g. GSE125792) and full-matrix pairwise computation would be too slow or depth-3+ screening risks no result.

## When NOT to use

- Normalization of a specific known gene is the goal — use ERgene.normalizationdata instead; single-gene standardization is documented as undesirable and should be used with caution per MIQE guidance
- The expression matrix is small enough that full-matrix screening is fast — blocking adds no benefit and depth 3+ already declines in efficiency
- Computational depth is set too large — the paper warns there may be no result at all

## Inputs

- Gene expression matrix as a pandas DataFrame (e.g. GSE125792 / McLoughlin dataset, 1968 genes, tab-separated with first column as gene index)
- Computational depth parameter (integer, e.g. 2 or 3)

## Outputs

- Ranked list of candidate reference genes sorted by smallest variation of relative factor change (top 20 at depth 2, e.g. RPL19, RPS4X, RPL13, TUBA1A, SF3B5, RPS3a, ...; top 7 at depth 3, e.g. RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19)
- Measured runtime report (e.g. 'calculate time:17.3s')
- An Upsetplot visualization when run in a Jupyter notebook

## How to apply

Load the expression matrix (e.g. GSE125792, 1968 genes) into a pandas DataFrame with the first column as index, then call ERgene.FindERG with a computational depth parameter (2 or 3; depth 2 gave the 20-gene panel, depth 3 the 7-gene panel on GSE125792). When the gene dataset is too large, ERgene automatically adopts block calculation, taking every 1000 genes as a block, and applies the inverse process of the internal reference method to compute the variance of relative factor change across genes. Results (variation of relative factor change) are sorted from smallest to largest and the top 20 genes (or top 7 at depth 3 per the source cards) are returned as the candidate reference panel. Set depth deliberately: efficiency begins to decline at depth 3 or larger, and a too-large depth may produce no result. Version 1.2.0 increased computing speed by a factor of 100; measure runtime (e.g. 4.58 s at depth 2, 17.3 s at depth 3 in the README example) as a sanity check.

## Related tools

- **ERgene** (Python library implementing the robust matrix block operation and inverse internal-reference screening (FindERG)) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame input format required by FindERG and the computational power underlying the library)

## Examples

```
import ERgene; import pandas as pd; data = pd.read_csv('gse.txt', sep='\t'); data = data.set_index(data.columns[0]); ERgene.FindERG(data, 3)
```

## Evaluation signals

- Depth-2 run on GSE125792 (1968 genes) reproduces the 20-gene panel: RPL19, RPS4X, RPL13, TUBA1A, SF3B5, RPS3a, RPL9, RPL39L, LDHA, RPS8, RPL31, FTL, RPS3, RPL22, PINLYP, CAPG, UQCRH, RPS5, RPSAP58, RPL36
- Depth-3 run reproduces the 7-gene panel: RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19
- Runtime is in seconds (e.g. 4.58 s at depth 2 per the article; 17.3 s at depth 3 per the README), consistent with the ~90% efficiency gain over Normfinder
- Returned genes are sorted by smallest variation of relative factor change and limited to the top 20
- Input DataFrame must be indexed on its first column (set_index(data.columns[0]) in ERgene v1.2)

## Limitations

- When the computational depth is set to 3 or larger, efficiency begins to decline; at too large a depth there may be no result
- Block calculation with every 1000 genes as a block is adopted only when the gene dataset is too large
- No ERgene software version number (other than README v1.2.0), no data accession statements, and installation instructions are not stated in the article text

## Evidence

- [discussion] block calculation abstract: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
- [abstract] robust block rationale: "The principle is based on the inverse process of the internal reference method, and the robust matrix block operation makes the selection of internal reference genes faster than any other method."
- [discussion] depth tradeoff: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [readme] speed gain: "In new Version 1.2.0, the computing speed is increased by a factor of 100"
- [discussion] depth2 runtime: [evidence span withheld — anchor status: not_found]
