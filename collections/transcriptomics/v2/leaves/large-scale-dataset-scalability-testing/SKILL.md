---
name: large-scale-dataset-scalability-testing
description: Use when benchmarking whether a method (e.g. a VAE-based network inferencer
  like FAVA vs. scLink, hdWGCNA, PCC) can process a very large scRNA-seq or proteomics
  matrix without failure, and where competitor methods break down.
when_to_use_negative:
- The dataset is small or has far fewer than 0.5 million conditions, where scalability
  is not the question being tested
- The goal is accuracy on small datasets rather than demonstrating the ability to
  process large datasets where baseline methods break down
- The input is not a high-dimensional condition-per-observation omics matrix (e.g.
  already a precomputed feature table or network)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: favapy (FAVA)
  role: Candidate method under scalability test; run from CLI on the large PBMC dataset
    with -n controlling number of output interactions
  repo: https://github.com/mikelkou/fava
- name: scLink
  role: Baseline method expected to fail on the large PBMC dataset, demonstrating
    FAVA's scalability
- name: hdWGCNA
  role: Baseline co-expression network method run on the same datasets for comparison
- name: AnnData
  role: Input data structure through which FAVA reads scRNA-seq matrices within the
    scverse ecosystem
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:549c9a0c0b1d716676fe66f8c3ab0ba7aa6a3ec4d4ac23f6ee60d7cbfa102def
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - PCC
  - Python
  - favapy
  - favapy (FAVA)
  - hdWGCNA
  - scLink
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# large-scale-dataset-scalability-testing

## Summary

Stress-tests whether a network-inference method scales to very large omics datasets (e.g. >0.5 million conditions) by running it on datasets like the large PBMC scRNA-seq matrix and comparing against baselines that fail, as done when evaluating FAVA against scLink.

## When to use

Use when benchmarking whether a method (e.g. a VAE-based network inferencer like FAVA vs. scLink, hdWGCNA, PCC) can process a very large scRNA-seq or proteomics matrix without failure, and where competitor methods break down.

## When NOT to use

- The dataset is small or has far fewer than 0.5 million conditions, where scalability is not the question being tested
- The goal is accuracy on small datasets rather than demonstrating the ability to process large datasets where baseline methods break down
- The input is not a high-dimensional condition-per-observation omics matrix (e.g. already a precomputed feature table or network)

## Inputs

- Large scRNA-seq expression matrix (e.g. large PBMC dataset) as tsv/csv file compatible with favapy / AnnData
- Smaller control scRNA-seq datasets (GBM, lung SCC, small PBMC, pancreatic tumor)
- Baseline method outputs (ranked interaction lists from scLink, hdWGCNA, PCC) truncated to top 10000 interactions
- Reference network databases: KEGG Release 95 and BioGRID Release 4.4.217

## Outputs

- Method x dataset x reference table of TP/FP ratios (e.g. benchmark_tpfp_ratios.csv)
- Ranked top-10000 protein-pair interaction list from the large-scale run, scored from Pearson correlation in latent space (1 = high correlation to -1 = high anti-correlation)
- Scalability record noting which methods failed on the large dataset (scLink absent on large PBMC)

## How to apply

Run the candidate method (via favapy with defaults: 50 epochs, batch size 32, auto-scaled hidden/latent dimensions) on a large scRNA-seq dataset such as the large PBMC matrix loaded as tsv/csv, alongside smaller datasets (GBM, lung SCC, small PBMC, pancreatic tumor) as controls. Run baseline methods (scLink, hdWGCNA, PCC) on the same inputs and record which ones fail — in the FAVA paper scLink was unable to process the large PBMC dataset, demonstrating FAVA's scalability advantage. Truncate every method's ranked interaction list to the top 10000 pairs (the favapy -n option; counts both proteinA-proteinB and proteinB-proteinA directions) so comparisons are on equal footing. Score each list against reference databases (KEGG Release 95, BioGRID Release 4.4.217) computing TP/FP ratios — e.g. FAVA reached 0.55 (KEGG) and 0.04 (BioGRID) on large PBMC — and write the method x dataset x reference table to a CSV. Correct application is judged by (a) the large-scale run completing where baselines fail, and (b) TP/FP ratios reproducible within the reported values.

## Related tools

- **favapy (FAVA)** (Candidate method under scalability test; run from CLI on the large PBMC dataset with -n controlling number of output interactions) — https://github.com/mikelkou/fava
- **scLink** (Baseline method expected to fail on the large PBMC dataset, demonstrating FAVA's scalability)
- **hdWGCNA** (Baseline co-expression network method run on the same datasets for comparison)
- **AnnData** (Input data structure through which FAVA reads scRNA-seq matrices within the scverse ecosystem) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy

## Examples

```
favapy /data/large_pbmc.tsv /out/large_pbmc_fava.txt -n 10000 -c
```

## Evaluation signals

- The large PBMC run completes with FAVA while scLink fails/is absent, reproducing the paper's scalability contrast
- TP/FP ratios on large PBMC fall near reported values: 0.55 (KEGG) and 0.04 (BioGRID)
- All methods' ranked lists are truncated to exactly 10000 interactions before scoring, with both interaction directions counted
- The output table covers every method x dataset x reference (KEGG Release 95, BioGRID Release 4.4.217) combination with no missing large-dataset entry for FAVA
- Latent-space Pearson correlation scores fall in [-1, 1] with 1 meaning high correlation

## Limitations

- Even FAVA's TP/FP ratios degrade on the large PBMC dataset (0.55 KEGG, 0.04 BioGRID) relative to small PBMC (3.9 KEGG), so scalability should not be conflated with accuracy
- Baseline methods may be unable to process large datasets at all (scLink on large PBMC), making full head-to-head comparison impossible at scale
- Truncating to a fixed top-10000 list counts both interaction directions (proteinA-proteinB and proteinB-proteinA), which can affect TP/FP counts if not handled consistently

## Evidence

- [abstract] FAVA can process large datasets with over 0.5 million conditions: "FAVA can process large datasets with over 0.5 million conditions"
- [methods] Run FAVA from the command line: "favapy <path-to-data-file> <path-to-save-output>"
- [other] scalability contrast with baselines on large PBMC: [evidence span withheld — anchor status: not_found]
- [other] benchmarking protocol and large-PBMC expected values: [evidence span withheld — anchor status: not_found]
