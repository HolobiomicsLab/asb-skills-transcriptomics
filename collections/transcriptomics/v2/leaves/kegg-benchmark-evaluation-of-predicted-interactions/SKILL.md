---
name: kegg-benchmark-evaluation-of-predicted-interactions
description: Use when you have a ranked list of predicted protein interactions from
  omics data (scRNA-seq or bulk proteomics) and need to quantify its accuracy against
  a KEGG gold-standard functional association set, or to compare predictors like FAVA
  vs plain PCC.
when_to_use_negative:
- The input is not a ranked predicted interaction list but raw expression data — run
  the predictor (favapy or PCC) first before benchmarking
- You need a physical protein-protein interaction (PPI) gold standard rather than
  functional association networks — KEGG functional associations are the benchmark
  used here, not binary PPIs
- The organism/pathway scope of your proteins is not covered by KEGG, making the benchmark
  reference inappropriate
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: favapy
  role: Produces the FAVA ranked interaction pairs (top 100000, both directions) that
    are benchmarked against KEGG
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Used to compute the PCC baseline, sweep the Pearson cut-off, and plot precision
    curves against KEGG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:b51bfa6bd6f6e265e1ee2e82f561d3a954d9b60b077aeffb670cc6a36260c086
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# kegg-benchmark-evaluation-of-predicted-interactions

## Summary

Benchmark ranked predicted protein-protein interactions (e.g. from FAVA or a PCC baseline) against KEGG functional associations by computing precision curves over the ranking, verifying that the scRNA-seq-derived FAVA network yields >5000 true positives with <500 false positives (>90% precision) and outperforms PCC.

## When to use

Use when you have a ranked list of predicted protein interactions from omics data (scRNA-seq or bulk proteomics) and need to quantify its accuracy against a KEGG gold-standard functional association set, or to compare predictors like FAVA vs plain PCC.

## When NOT to use

- The input is not a ranked predicted interaction list but raw expression data — run the predictor (favapy or PCC) first before benchmarking
- You need a physical protein-protein interaction (PPI) gold standard rather than functional association networks — KEGG functional associations are the benchmark used here, not binary PPIs
- The organism/pathway scope of your proteins is not covered by KEGG, making the benchmark reference inappropriate

## Inputs

- Ranked interaction list from FAVA (top 100000 pairs, both directions) derived from HPA scRNA-seq matrix (566109 cells, tsv/csv)
- Ranked interaction list from a plain PCC baseline computed on the same scRNA-seq matrix (cut-off swept across the full range 1 to -1)
- Ranked interaction lists from FAVA and PCC on PRIDE bulk proteomics data (633 experiments, 18846 proteins)
- KEGG functional associations used as the benchmark gold standard

## Outputs

- Precision curves (precision vs ranked predictions) for FAVA and PCC on both datasets
- True positive and false positive counts against KEGG at given ranking depths (expect >5000 TP and <500 FP on scRNA-seq)
- Runtime comparison of FAVA vs PCC per dataset (expected ≥9x speedup, ~1 day vs ~9 days)

## How to apply

Run the interaction predictor (e.g. FAVA via favapy, and a plain PCC baseline sweeping the Pearson cut-off from +1 to -1) on the same input matrix, keeping the top 100000 interaction pairs with both directions (proteinA-proteinB and proteinB-proteinA). Rank the predicted pairs and walk down the ranking, classifying each pair as a true positive or false positive against KEGG functional associations. Plot precision curves (precision vs number of predictions) for both the scRNA-seq Human Protein Atlas data (566109 cells) and the PRIDE bulk proteomics data (633 experiments, 18846 proteins). Accept the method if the scRNA-seq curve exceeds 5000 true positives with fewer than 500 false positives (precision > 90%) and FAVA clearly outperforms PCC; additionally record wall-clock runtime to confirm FAVA is at least 9x faster (~1 day vs ~9 days for PCC).

## Related tools

- **favapy** (Produces the FAVA ranked interaction pairs (top 100000, both directions) that are benchmarked against KEGG) — https://github.com/mikelkou/fava
- **Python** (Used to compute the PCC baseline, sweep the Pearson cut-off, and plot precision curves against KEGG)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- The scRNA-seq precision curve shows more than 5000 true positives with fewer than 500 false positives (>90% precision) against KEGG
- FAVA precision curve clearly dominates the plain PCC curve on both HPA scRNA-seq and PRIDE proteomics data
- Wall-clock runtime of FAVA is at least 9x faster than PCC (~1 day vs ~9 days)
- Predicted pairs are output with both directions (proteinA-proteinB and proteinB-proteinA) and capped at the top 100000 interactions
- FAVA covers understudied proteins (e.g. 4210 interactions between 1039 understudied proteins), consistent with networks free of literature bias

## Limitations

- Benchmark accuracy depends on KEGG functional associations as the reference; performance reflects overlap with KEGG, not true physical interactions
- Simple co-expression analysis (PCC) performs poorly on such data due to high sparseness and redundancy, so the PCC baseline is a weak comparator by design
- Precision can be biased by data availability since well-studied proteins have many more interactions in existing networks; the KEGG gold standard itself may carry literature bias

## Evidence

- [results] FAVA benchmarked against KEGG with >90% precision on scRNA-seq: [evidence span withheld — anchor status: not_found]
- [intro] FAVA outperforms PCC and handles sparseness/redundancy: "this approach does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy"
- [intro] Latent-space correlations improve networks over the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [abstract] Understudied protein coverage as a quality signal: "has predicted 4210 interactions between 1039 understudied proteins"
- [methods] favapy CLI invocation for generating predictions to benchmark: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
