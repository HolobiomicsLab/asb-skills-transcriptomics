---
name: cross-tissue-gene-coverage-comparison
description: Use when per-tissue MRSD score tables (*.results.mrsd.txt) are available
  for two or more biosamples and you must rank tissues by the fraction of low-MRSD
  genes within a gene panel.
when_to_use_negative:
- The goal is suitability for differential expression or isoform-imbalance analysis
  — the article states suitability for these applications is not quantified or proven
- The RNA-seq data are long-read, single-cell, or spatially resolved — the MRSD model
  is validated only for bulk short-read poly-A enrichment workflows
- Sample IDs mismatch between the *.junc-counts.txt and *.seq-depths.txt files — such
  samples are excluded from MRSD calculation, so downstream tissue comparison would
  be unreliable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MRSD
  role: Statistical framework whose per-gene, per-tissue scores provide the quantitative
    input for cross-tissue coverage comparison
- name: MRSDexplorer
  role: Generates the *.results.mrsd.txt score tables consumed by the comparison workflow
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: pandas-based parsing of MRSD tables, low-MRSD classification, panel intersection,
    and optimal-tissue aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:54d6eded900050d0391a55cf9f1133a2040f56187da0b9ab6f63b794139af218
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-tissue-gene-coverage-comparison

## Summary

Compares per-tissue MRSD scores (in millions of reads) across multiple RNA-seq biosamples to classify genes as low-MRSD (<100 M reads) and select the optimal RNA source per disease gene panel. Grounds tissue-selection decisions in quantitative splice-junction coverage predictions from GTEx-derived control data.

## When to use

Use when per-tissue MRSD score tables (*.results.mrsd.txt) are available for two or more biosamples and you must rank tissues by the fraction of low-MRSD genes within a gene panel.

## When NOT to use

- The goal is suitability for differential expression or isoform-imbalance analysis — the article states suitability for these applications is not quantified or proven
- The RNA-seq data are long-read, single-cell, or spatially resolved — the MRSD model is validated only for bulk short-read poly-A enrichment workflows
- Sample IDs mismatch between the *.junc-counts.txt and *.seq-depths.txt files — such samples are excluded from MRSD calculation, so downstream tissue comparison would be unreliable

## Inputs

- Per-tissue MRSD score tables (*.results.mrsd.txt) with gene symbol, Ensembl ID, and MRSD in millions of reads, one per biosample (e.g., whole blood, LCLs, skeletal muscle, fibroblasts)
- Disease gene panel gene lists (e.g., 295 PanelApp panels, one gene list per panel)

## Outputs

- Per-gene low-MRSD (<100 M reads) classification matrix across biosamples
- Per-panel optimal biosample table (panel_optimal_tissue.tsv) with panel-level low-MRSD proportions per tissue
- Summary counts of panels for which each biosample is (joint-)optimal

## How to apply

Load the per-tissue MRSD tables (*.results.mrsd.txt) with pandas, treating MRSD values in millions of reads and marking '-' entries (median splice junction coverage of 0 reads) as NOT low-MRSD. Classify each gene as low-MRSD (<100 million reads) per tissue, then aggregate across tissues and verify the fraction of genes low-MRSD in at least one biosample matches expectations (64.2%, 2,133/3,322 in the source study). Intersect each disease gene panel's gene list with the per-tissue low-MRSD classifications, compute per-panel proportions per biosample, and select the tissue with the highest proportion (ties counted as joint-optimal) as the optimal RNA source. Write the per-panel optimal biosample assignments to a summary table (e.g., panel_optimal_tissue.tsv).

## Related tools

- **MRSD** (Statistical framework whose per-gene, per-tissue scores provide the quantitative input for cross-tissue coverage comparison)
- **MRSDexplorer** (Generates the *.results.mrsd.txt score tables consumed by the comparison workflow) — github.com/mcgm-mrsd/mrsd-explorer
- **Python** (pandas-based parsing of MRSD tables, low-MRSD classification, panel intersection, and optimal-tissue aggregation)

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --output_prefix example transcripts.introns.bed
```

## Evaluation signals

- Aggregation check: fraction of the 3,322 multi-exon genes low-MRSD in at least one biosample equals 64.2% (2,133/3,322)
- Optimal-tissue counts reproduce the reported ranking: fibroblasts 186/295 panels (63.1%), LCLs 126/295, skeletal muscle 70/295, whole blood 21/295
- '-' MRSD entries (median splice junction coverage of 0 reads) are excluded from the low-MRSD class rather than imputed as low
- Every panel's optimal biosample corresponds to the maximum per-panel low-MRSD proportion across the four tissues, with ties handled as joint-optimal
- Per-panel proportions are computed over the intersection of panel genes with successfully scored genes only (genes missing from *.results.mrsd.txt are dropped)

## Limitations

- Pre-computed MRSDs derive from GTEx samples sequenced with an Illumina 75 bp paired-end poly-A workflow, so scores are most accurate for similar RNA-seq workflows
- The MRSD model does not explicitly account for eQTLs, sQTLs, or genomic backgrounds causing widespread transcriptomic changes (e.g., interferonopathies, spliceosome disruption)
- '-' entries indicate median splice junction coverage of 0 reads, so unexpressed genes cannot be ranked across tissues

## Evidence

- [abstract] Fibroblasts optimum for 63.1% of panels: "Applying MRSD scoring to established disease gene panels shows that fibroblasts, of these four biosamples, are the optimum source of RNA for 63.1% of gene panels"
- [methods] MRSD definition: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
- [readme] Output format of MRSDexplorer: "The primary output file of `MRSDexplorer.py` is a text file with the suffix *.results.mrsd.txt*."
- [readme] Interpretation of '-' entries: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
