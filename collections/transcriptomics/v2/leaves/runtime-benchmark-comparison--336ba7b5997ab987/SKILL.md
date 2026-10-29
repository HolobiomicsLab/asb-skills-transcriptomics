---
name: runtime-benchmark-comparison
description: Use when evaluating or reporting the computational cost of ERgene reference-gene
  screening on an expression matrix (e.g. GSE125792), especially when choosing a computational
  depth or comparing speed to geNorm/Normfinder/Keeper.
when_to_use_negative:
- Input is not a DataFrame-format expression matrix (ERgene.FindERG requires pandas
  DataFrame data)
- The goal is statistical normalization rather than screening/benchmarking — use Ergene.Normalizationdata
  instead, and note single-gene normalization is discouraged per MIQE guidelines unless
  invariable expression is demonstrated
- A very large computational depth is required and a missing/empty result would be
  unacceptable, since too large a depth may return no result
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ERgene
  role: Python library whose FindERG screening function is timed and benchmarked at
    different computational depths
  repo: github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides the DataFrame data structure and computational power used to load
    and screen the expression matrix
- name: Python
  role: Runtime environment in which ERgene benchmarks are executed and timed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:237aab7207e183eec784d511bf4ca2a273010a80505b41e2cf330bf5129e2fdb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ERgene
  - Pandas
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# runtime-benchmark-comparison

## Summary

Benchmarks the runtime of ERgene's reference-gene screening (ERgene.FindERG) on the GSE125792/McLoughlin dataset at different computational depths, comparing efficiency against methods like Normfinder and recording measured wall-clock times alongside candidate gene outputs.

## When to use

Use when evaluating or reporting the computational cost of ERgene reference-gene screening on an expression matrix (e.g. GSE125792), especially when choosing a computational depth or comparing speed to geNorm/Normfinder/BestKeeper.

## When NOT to use

- Input is not a DataFrame-format expression matrix (ERgene.FindERG requires pandas DataFrame data)
- The goal is statistical normalization rather than screening/benchmarking — use Ergene.Normalizationdata instead, and note single-gene normalization is discouraged per MIQE guidelines unless invariable expression is demonstrated
- A very large computational depth is required and a missing/empty result would be unacceptable, since too large a depth may return no result

## Inputs

- Gene expression matrix as a pandas DataFrame (e.g. GSE125792 read from 'gse.txt' with sep='\t', indexed on the first column)
- Computational depth parameter (integer, e.g. 2 or 3)

## Outputs

- Ranked candidate reference gene list (top 20 genes, sorted by variation of relative factor change; depth 3 on GSE125792 yields seven: RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19)
- Measured screening runtime (e.g. 'calculate time:17.3s' at depth 3), optionally written with gene IDs to reference_genes_depth3.csv

## How to apply

Load the expression matrix (e.g. the McLoughlin dataset GSE125792, read as a tab-separated file and indexed on its first column) into a pandas DataFrame, the format required by ERgene's screening function. Call ERgene.FindERG with the DataFrame and a chosen computational depth, timing the run — depth 2 on the 1968-gene test set took 4.58 s, while the depth-3 run reported in the README took 17.3 s. Use depth 3 to reproduce the paper's seven-gene candidate list (RPL13/ASHGV40056316, FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19); version 1.2.0 of ERgene increased computing speed by a factor of 100. Record measured runtime and output genes together (e.g. to reference_genes_depth3.csv) and compare against reported baselines, noting ERgene is nearly 90% faster than Normfinder. Expect efficiency to decline at depth 3 or larger, so treat depth as the key decision point in any runtime benchmark.

## Related tools

- **ERgene** (Python library whose FindERG screening function is timed and benchmarked at different computational depths) — github.com/Starlitnightly/ERgene
- **Pandas** (Provides the DataFrame data structure and computational power used to load and screen the expression matrix)
- **Python** (Runtime environment in which ERgene benchmarks are executed and timed)

## Examples

```
import ERgene, pandas as pd; data=pd.read_csv('gse.txt',sep='\t'); data=data.set_index(data.columns[0]); ERgene.FindERG(data,3)
```

## Evaluation signals

- Depth-3 run on GSE125792 returns exactly seven candidate genes: RPL13 (ASHGV40056316), FTL, RPS3a, SF3B5, RPS4X, TUBA1A, RPL19
- Runtime is recorded and printed by the function (README example reports 'calculate time:17.3s' at depth 3)
- Depth-2 run on the 1968-gene test set completes in ~4.58 s as reported in the paper
- Output genes are ranked by variation of relative factor change sorted from smallest to largest, with the top 20 genes returned
- Results are comparable to reference: ERgene efficiency is nearly 90% higher than Normfinder and found genes similar to NormFinder/geNorm (Table 2)

## Limitations

- When the computational depth is set to 3 or larger, efficiency begins to decline; setting depth too large may produce no result
- Runtime depends on ERgene version — version 1.2.0 increased computing speed by a factor of 100, and no software version number is stated in the article text
- Block calculation (every 1000 genes per block) affects performance on large datasets
- Single reference gene normalization is not acceptable per MIQE guidelines unless invariable expression is demonstrated

## Evidence

- [discussion] Total number of genes tested was 1968; depth 2 screening took 4.58 s: [evidence span withheld — anchor status: not_found]
- [discussion] ERgene efficiency vs Normfinder: "The computational efficiency of ERgene is increased by nearly 90% higher than that of Normfinder"
- [discussion] Efficiency declines at larger depth: "When the computational depth is set to 3 or larger, the efficiency begins to decline. When the computational depth set too large, there may be no result."
- [readme] Version 1.2.0 speedup: "In new Version 1.2.0, the computing speed is increased by a factor of 100"
- [readme] Depth-3 README example with runtime: "calculate time:17.3s
['ASHGV40057862', 'ASHGV40057056', 'ASHGV40025887', 'ASHGV40056316', 'ASHGV40056377', 'ASHGV40057617', 'ASHGV40057263']"
