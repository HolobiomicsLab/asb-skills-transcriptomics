---
name: protein-interaction-benchmarking-against-kegg-biogrid
description: Use when you have ranked protein-pair interaction predictions from one
  or more network-inference methods on scRNA-seq datasets and need a reference-standard
  comparison against KEGG Release 95 or BioGRID Release 4.4.217.
when_to_use_negative:
- Predictions are not a ranked interaction list (e.g., already an adjacency matrix
  with no ranking to truncate)
- Ground-truth evaluation should avoid curated databases because the goal is to measure
  literature bias rather than replicate it (the paper emphasizes existing networks
  are heavily biased by data availability)
- The dataset cannot be processed by a competing method (e.g., scLink could not process
  the large PBMC dataset), so ratios are not comparable across methods
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0605
tools:
- name: favapy
  role: Produces the FAVA ranked protein-pair list (latent-space Pearson correlation
    from 1 to -1) whose top 10000 interactions are benchmarked; -n controls the number
    of output interactions
  repo: https://github.com/mikelkou/fava
- name: scLink
  role: Baseline method generating comparable ranked interaction lists on the five
    datasets
- name: hdWGCNA
  role: Baseline method generating comparable ranked interaction lists on the five
    datasets
- name: PCC
  role: Plain Pearson correlation baseline generating ranked interaction lists on
    the same datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:7ef43540d36982f13499598b4465426a4382aae6ee9bfd7c5ccc1cdf02d3b3fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - PCC
  - Python
  - favapy
  - hdWGCNA
  - scLink
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# protein-interaction-benchmarking-against-kegg-biogrid

## Summary

Benchmark ranked protein-protein interaction lists (e.g., from FAVA, scLink, hdWGCNA, PCC) by truncating each to the top 10000 interactions and scoring them against KEGG and BioGRID reference networks as ground truth via true-positive/false-positive counts and TP/FP ratios.

## When to use

Use when you have ranked protein-pair interaction predictions from one or more network-inference methods on scRNA-seq datasets and need a reference-standard comparison against KEGG Release 95 or BioGRID Release 4.4.217.

## When NOT to use

- Predictions are not a ranked interaction list (e.g., already an adjacency matrix with no ranking to truncate)
- Ground-truth evaluation should avoid curated databases because the goal is to measure literature bias rather than replicate it (the paper emphasizes existing networks are heavily biased by data availability)
- The dataset cannot be processed by a competing method (e.g., scLink could not process the large PBMC dataset), so ratios are not comparable across methods

## Inputs

- Ranked protein-pair interaction lists (top-N truncated, e.g. top 10000) from each method (favapy output, scLink, hdWGCNA, PCC rankings)
- KEGG Release 95 reference interaction set
- BioGRID Release 4.4.217 reference interaction set
- Five scRNA-seq expression matrices as tsv/csv files (GBM, lung SCC, small PBMC, pancreatic tumor, large PBMC)

## Outputs

- TP and FP counts per method x dataset x reference database
- TP/FP ratio table (e.g., benchmark_tpfp_ratios.csv)
- Method comparison summary (FAVA vs scLink, hdWGCNA, PCC)

## How to apply

Truncate each method's ranked interaction list to the top 10000 pairs, counting both orientations (proteinA-proteinB and proteinB-proteinA) as separate entries; the favapy -n option controls this output count. Match predicted pairs against KEGG Release 95 and BioGRID Release 4.4.217 reference networks, counting true positives (predicted pairs present in the reference) and false positives (predicted pairs absent), then compute TP/FP ratios per method per dataset. Compare methods on the same five scRNA-seq datasets (GBM, lung SCC, small PBMC, pancreatic tumor, large PBMC) and verify results against published values, e.g., FAVA 3.9 (4588/1168) vs scLink 1.0 (1878/1860) on small PBMC vs KEGG, and FAVA 0.55 (KEGG) / 0.04 (BioGRID) on large PBMC.

## Related tools

- **favapy** (Produces the FAVA ranked protein-pair list (latent-space Pearson correlation from 1 to -1) whose top 10000 interactions are benchmarked; -n controls the number of output interactions) — https://github.com/mikelkou/fava
- **scLink** (Baseline method generating comparable ranked interaction lists on the five datasets)
- **hdWGCNA** (Baseline method generating comparable ranked interaction lists on the five datasets)
- **PCC** (Plain Pearson correlation baseline generating ranked interaction lists on the same datasets)

## Examples

```
favapy <path-to-data-file> <path-to-save-output> -n 10000 -c 1
```

## Evaluation signals

- Small PBMC vs KEGG: FAVA TP/FP = 4588/1168 = 3.9, scLink = 1878/1860 = 1.0 (FAVA 3.9x better than second-best)
- Large PBMC: FAVA TP/FP = 0.55 (KEGG) and 0.04 (BioGRID); scLink absent on large PBMC
- Each method's list is truncated to exactly 10000 interactions before counting TP/FP
- Same top-10000 lists are scored against both KEGG Release 95 and BioGRID Release 4.4.217 so ratio tables are complete over method x dataset x reference
- Reproduced ratios are written to benchmark_tpfp_ratios.csv and match reported values

## Limitations

- TP/FP ratios are only available against databases containing interactions for well-studied proteins; existing high-quality networks are heavily biased by data availability
- Reference sets differ in coverage (KEGG vs BioGRID give very different ratios, e.g. 0.55 vs 0.04 for FAVA on large PBMC), so cross-database comparisons are not interchangeable
- Some baselines may not scale to all datasets — scLink was expected to be unable to process the large PBMC dataset

## Evidence

- [methods] favapy ranked list truncation: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [abstract] FAVA accuracy claim: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods, across a diverse collection of real as well as simulated datasets"
- [intro] latent-space correlation scoring: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [intro] literature bias caveat: "most existing high-quality networks are heavily biased by data availability, in the sense that well-studied proteins have many more interactions than understudied proteins"
