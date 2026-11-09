---
name: panel-gene-set-intersection-analysis
description: Use when you have per-gene MRSD scores for multiple biosamples and curated
  disease gene panel gene lists, and need to rank tissues per panel by fraction of
  low-MRSD genes.
when_to_use_negative:
- Input is raw RNA-seq data or junction counts rather than pre-computed MRSD scores
  — generate *.results.mrsd.txt via MRSDexplorer first
- The goal is panel-level assessment of differential expression or allele-specific
  expression suitability, which the MRSD model does not directly support
- The RNA-seq protocol differs substantially from GTEx Illumina 75 bp paired-end poly-A
  enrichment, for which the pre-computed MRSDs are most accurate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: MRSDexplorer
  role: Produces the per-gene, per-tissue MRSD scores (*.results.mrsd.txt) that are
    classified as low/high before panel intersection
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: Pandas-based loading of MRSD tables, low-MRSD classification (<100 M reads),
    panel intersection, and per-panel optimal-tissue tabulation
- name: GTEx
  role: Source of the RNA-seq control data underlying the MRSD scores for the four
    biosamples compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:da4aebfc2938d467ac4167c8d77a240b4c3b71b68744f37a764f4025c3fbf041
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GTEx
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# panel-gene-set-intersection-analysis

## Summary

Intersects disease gene panel gene lists (e.g. 295 PanelApp panels) with per-biosample MRSD classifications to determine, for each panel, which RNA source (whole blood, LCLs, skeletal muscle, fibroblasts) yields the highest proportion of low-MRSD (<100 M reads) genes.

## When to use

Use when you have per-gene MRSD scores for multiple biosamples and curated disease gene panel gene lists, and need to rank tissues per panel by fraction of low-MRSD genes.

## When NOT to use

- Input is raw RNA-seq data or junction counts rather than pre-computed MRSD scores — generate *.results.mrsd.txt via MRSDexplorer first
- The goal is panel-level assessment of differential expression or allele-specific expression suitability, which the MRSD model does not directly support
- The RNA-seq protocol differs substantially from GTEx Illumina 75 bp paired-end poly-A enrichment, for which the pre-computed MRSDs are most accurate

## Inputs

- Per-tissue MRSD result files (*.results.mrsd.txt) for whole blood, LCLs, skeletal muscle, and fibroblasts, with MRSD values in millions of reads
- Disease gene panel gene lists (295 PanelApp panels), keyed by GENCODE v19 gene symbol and/or Ensembl gene ID

## Outputs

- Per-panel optimal biosample table (panel_optimal_tissue.tsv)
- Per-panel proportions of low-MRSD genes per biosample
- Summary counts of panels for which each biosample is optimal (e.g. fibroblasts 186/295)

## How to apply

Load each per-tissue MRSD output (*.results.mrsd.txt) with pandas, parsing MRSD values reported in millions of reads, and classify a gene as low-MRSD (<100 million reads) per tissue, treating '-' entries (median splice junction coverage of 0 reads) as not low-MRSD. Load the 295 PanelApp disease gene panel gene lists and intersect each panel with the per-tissue low-MRSD classifications. For each panel, compute the proportion of panel genes low-MRSD in each of the four biosamples (whole blood, LCLs, skeletal muscle, fibroblasts) and select the biosample with the highest proportion (allowing joint-optimal ties) as the optimal RNA source. Count panels for which each biosample is optimal and compare against published benchmarks (fibroblasts optimal for 186/295 = 63.1% of panels) before writing the per-panel table.

## Related tools

- **MRSDexplorer** (Produces the per-gene, per-tissue MRSD scores (*.results.mrsd.txt) that are classified as low/high before panel intersection) — github.com/mcgm-mrsd/mrsd-explorer
- **Python** (Pandas-based loading of MRSD tables, low-MRSD classification (<100 M reads), panel intersection, and per-panel optimal-tissue tabulation)
- **GTEx** (Source of the RNA-seq control data underlying the MRSD scores for the four biosamples compared)

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 8 --sj_prop 0.75 --mrsd_param 0.95 genes.txt  # then classify MRSD<100 M per tissue in pandas and intersect with 295 panel gene lists
```

## Evaluation signals

- Fraction of genes low-MRSD in at least one biosample matches the reported 64.2% (2,133/3,322 multi-exon disease genes)
- Per-panel optimal biosample counts reproduce fibroblasts 186/295 (63.1%), LCLs 126/295, skeletal muscle 70/295, whole blood 21/295
- Genes with '-' MRSD (median junction coverage 0 reads) are excluded from the low-MRSD set in every tissue
- Every panel gene in the output maps to a row in the MRSD result tables (or is explicitly reported as unmapped)
- Output table panel_optimal_tissue.tsv has one row per panel (295 rows) with the optimal biosample per panel

## Limitations

- Genes whose median splice junction coverage is 0 reads ('-' entries) cannot be assessed and are treated as not low-MRSD
- MRSD predictions may not accurately reflect coverage in patients with disorders associated with widespread transcriptomic changes (e.g. interferonopathies, spliceosome disruption), and the model does not account for eQTLs or sQTLs
- Pre-computed MRSD scores are most accurate for GTEx-like Illumina 75 bp paired-end poly-A enrichment workflows; other methodologies require regenerating scores

## Evidence

- [abstract] Fibroblasts are the optimum RNA source among the four biosamples for 63.1% of disease gene panels: "Applying MRSD scoring to established disease gene panels shows that fibroblasts, of these four biosamples, are the optimum source of RNA for 63.1% of gene panels"
- [abstract] MRSD scores are gene- and tissue-specific sequencing depth estimates: "We have derived a metric, minimum required sequencing depth (MRSD), that estimates the depth of sequencing required from RNA sequencing to achieve user-specified sequencing coverage of a gene,"
- [readme] Output format of MRSDexplorer results file with '-' for zero-coverage genes: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [readme] MRSDexplorer outputs a text file with suffix *.results.mrsd.txt listing MRSD scores in millions of reads: "The primary output file of `MRSDexplorer.py` is a text file with the suffix *.results.mrsd.txt*."
- [discussion] MRSD-based selection of biosamples for bulk RNA-seq: "The MRSD-based approach that we describe here allows informed selection of biosample(s) for bulk RNA-seq, based on the required number of sequencing reads"
