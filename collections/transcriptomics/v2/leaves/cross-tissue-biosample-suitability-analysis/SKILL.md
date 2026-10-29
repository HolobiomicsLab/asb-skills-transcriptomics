---
name: cross-tissue-biosample-suitability-analysis
description: Use when you have .results.mrsd.txt scores for multiple GTEx-derived
  biosamples and a one-per-line gene list (GENCODE v19 symbols/Ensembl IDs), and need
  to determine which genes/variants are assayable or low-MRSD (≤100 M reads) in at
  least one tissue.
when_to_use_negative:
- Input RNA-seq uses non-Illumina, non-poly-A-enrichment, long-read, single-cell,
  or spatially resolved protocols — pre-computed MRSD scores are only accurate for
  similar Illumina 75 bp paired-end poly-A workflows and no control datasets exist
  for other modalities
- The goal is detecting allele-specific expression biases or differential gene expression
  — the article explicitly states the MRSD model cannot be directly extended to these
  tasks
- The disease is associated with widespread transcriptomic change (e.g., interferonopathies,
  spliceosome disruption) or gene expression is modified by eQTLs/sQTLs — MRSD predictions
  may not reflect coverage in such genomic backgrounds
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MRSDexplorer
  role: Generates the per-gene MRSD scores (.results.mrsd.txt) for each GTEx-derived
    biosample at --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type
    unique
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: MRSD
  role: Statistical framework underlying the scores; MRSD is the gene- and tissue-specific
    minimum required sequencing depth used to flag low-MRSD genes (≤100 M reads)
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: MendelianRNA-seq
  role: Cummings et al. pipeline scripts used upstream to produce the .csf.splicing.txt
    splice junction counts from FASTQs that feed MRSDexplorer
  repo: github.com/berylc/MendelianRNA-seq
- name: Python
  role: pandas-based joining of variant genes to per-biosample MRSD scores, flagging
    of '-' entries as non-assayable, and computation of the variant fraction stratified
    by stringency
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:07617d1abe4c286f1fa2a7334a01d97cfa218a0c64906fe78a0c597dede08019
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - MendelianRNA-seq
  - Python
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-tissue-biosample-suitability-analysis

## Summary

Quantifies whether an RNA-seq biosample (whole blood, LCL, skeletal muscle, or cultured fibroblast) can assay splicing aberrations in a gene of interest, by joining per-gene MRSD scores computed with MRSDexplorer.py against a gene list (e.g., SpliceAI-prioritized ClinVar VUS-harboring genes) and flagging genes with MRSD ≤ 100 million reads.

## When to use

Use when you have .results.mrsd.txt scores for multiple GTEx-derived biosamples and a one-per-line gene list (GENCODE v19 symbols/Ensembl IDs), and need to determine which genes/variants are assayable or low-MRSD (≤100 M reads) in at least one tissue.

## When NOT to use

- Input RNA-seq uses non-Illumina, non-poly-A-enrichment, long-read, single-cell, or spatially resolved protocols — pre-computed MRSD scores are only accurate for similar Illumina 75 bp paired-end poly-A workflows and no control datasets exist for other modalities
- The goal is detecting allele-specific expression biases or differential gene expression — the article explicitly states the MRSD model cannot be directly extended to these tasks
- The disease is associated with widespread transcriptomic change (e.g., interferonopathies, spliceosome disruption) or gene expression is modified by eQTLs/sQTLs — MRSD predictions may not reflect coverage in such genomic backgrounds

## Inputs

- MRSD score tables (.results.mrsd.txt) per biosample: whole blood, LCL, skeletal muscle, cultured fibroblast (GTEx-derived), computed at --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique
- One-per-line gene list of 7,507 SpliceAI-filtered ClinVar VUS-harboring genes (GENCODE v19 gene symbols and/or Ensembl gene IDs, up to two columns)

## Outputs

- Per-variant classification table with gene-level MRSD scores in each of the four biosamples and a low-MRSD flag (written to mrsd_variant_fraction.csv)
- Fraction of SpliceAI-prioritized ClinVar VUSs in low-MRSD genes, stratified by model stringency (reported range: 25.8%-67.8%)

## How to apply

First generate MRSD scores per biosample with MRSDexplorer.py using the parameters of the reference analysis (--number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique) over the Cummings-pipeline splice junction counts and .seq-depths.txt files. Then, in Python/pandas, join each variant's gene (via GENCODE v19 symbol and/or Ensembl gene ID) to its MRSD score in each of the four biosample score tables (.results.mrsd.txt), treating '-' entries as genes whose median junction coverage was 0 reads (not assayable). Flag a gene as low-MRSD if its score is ≤ 100 million reads in at least one biosample, and compute the fraction of variants lying in low-MRSD genes, stratifying across model stringency settings to obtain the range reported in the paper. The MRSD percentile (--mrsd_param 0.95) and junction coverage criteria (--number_reads, --sj_prop) define model stringency and thus control the resulting fraction.

## Related tools

- **MRSDexplorer** (Generates the per-gene MRSD scores (.results.mrsd.txt) for each GTEx-derived biosample at --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique) — github.com/mcgm-mrsd/mrsd-explorer
- **MRSD** (Statistical framework underlying the scores; MRSD is the gene- and tissue-specific minimum required sequencing depth used to flag low-MRSD genes (≤100 M reads)) — github.com/mcgm-mrsd/mrsd-explorer
- **MendelianRNA-seq** (Cummings et al. pipeline scripts used upstream to produce the .csf.splicing.txt splice junction counts from FASTQs that feed MRSDexplorer) — github.com/berylc/MendelianRNA-seq
- **Python** (pandas-based joining of variant genes to per-biosample MRSD scores, flagging of '-' entries as non-assayable, and computation of the variant fraction stratified by stringency)

## Examples

```
python MRSDexplorer.py --tissues whole_blood,LCL,skeletal_muscle,fibroblast --number_reads 8 --sj_prop 0.75 --mrsd_param 0.95 --read_type unique vus_genes.txt
```

## Evaluation signals

- The computed fraction of variants in low-MRSD genes must reproduce the reported 25.8%-67.8% range, with the upper bound 67.8% (5,086/7,507 VUSs) corresponding to the least stringent model setting in at least one biosample
- Genes with '-' in a .results.mrsd.txt entry must be classified as non-assayable (median junction coverage of 0 reads), not imputed
- Every gene in the input list must match the pre-computed MRSD table by GENCODE v19 symbol or Ensembl gene ID; unmatched identifiers should be accounted for (the README reports 'Number of genes successfully cross-referenced')

## Limitations

- MRSD was validated only for bulk short-read poly-A-enrichment RNA-seq processed with the specific Cummings bioinformatics pipeline; other experimental or bioinformatics approaches are not characterized
- The model does not explicitly account for eQTLs, sQTLs, or genomic backgrounds with widespread transcriptomic change, so predictions may not reflect patient coverage for disorders like interferonopathies or spliceosome disruption
- The 25.8%-67.8% range is dependent on model stringency settings (--number_reads, --sj_prop, --mrsd_param); the fraction is not a single fixed number

## Evidence

- [full_text] At a specified read coverage of 8 reads, between 25.8% and 67.8% of SpliceAI-prioritized ClinVar VUSs may lie in genes that are low-MRSD in at least one of the four tissues, dependent on model stringency.: [evidence span withheld — anchor status: not_found]
- [abstract] MRSD definition from abstract: "We have derived a metric, minimum required sequencing depth (MRSD), that estimates the depth of sequencing required from RNA sequencing to achieve user-specified sequencing coverage of a gene,"
- [readme] README on '-' entries: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [readme] README output format: "The primary output file of `MRSDexplorer.py` is a text file with the suffix *.results.mrsd.txt*."
- [discussion] Article limitation on protocol scope: "the analysis of a selection of tissues by bulk short-read poly-A enrichment RNA-seq processed using a specific bioinformatics analysis pipeline"
- [readme] README GTEx workflow caveat: "GTEx samples are sequenced using an Illumina 75 bp paired-end read poly-A enrichment workflow. As such, the pre-computed MRSDs are most accurate for assessing transcript coverage with similar RNA-seq"
