---
name: proportion-summary-statistics
description: Use when you have per-gene binary classifications across biosamples (e.g.
  low-MRSD <100M reads) and gene panels, and need to report the fraction of genes
  classified and the count/proportion of panels where each option is optimal.
when_to_use_negative:
- MRSD scores were generated for a different sequencing workflow or parameterization
  than GTEx's Illumina 75 bp paired-end poly-A enriched data, so the <100M-read low-MRSD
  cutoff and pre-computed scores are not directly applicable
- The goal is differential expression, allele-specific expression, or isoform-imbalance
  assessment, which the MRSD model does not quantify
- Input data are junction counts or sequencing depths rather than computed MRSD result
  tables; generate MRSD scores with MRSDexplorer.py first
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0085
tools:
- name: MRSDexplorer
  role: Generates the *.results.mrsd.txt score tables per biosample that the proportion
    summaries are computed from
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: Used with pandas to parse MRSD tables, classify genes as low-MRSD (<100M reads),
    intersect with panel gene lists, and tabulate per-panel optimal biosamples
- name: MRSD
  role: Statistical framework producing the gene- and tissue-specific minimum required
    sequencing depth (in millions of reads) that the low-MRSD threshold classifies
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:f970de38df4fe7eebf790d7cf9bb395dd64823c8c24c3368fa671e9f3484f357
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# proportion-summary-statistics

## Summary

Computes and validates proportion-based summary statistics from per-gene/per-tissue classification tables, e.g. the fraction of multi-exon disease genes with low-MRSD (<100 million reads) and the proportion of panels for which each biosample is optimal, as in the MRSD PanelApp tissue-comparison analysis.

## When to use

Use when you have per-gene binary classifications across biosamples (e.g. low-MRSD <100M reads) and gene panels, and need to report the fraction of genes classified and the count/proportion of panels where each option is optimal.

## When NOT to use

- MRSD scores were generated for a different sequencing workflow or parameterization than GTEx's Illumina 75 bp paired-end poly-A enriched data, so the <100M-read low-MRSD cutoff and pre-computed scores are not directly applicable
- The goal is differential expression, allele-specific expression, or isoform-imbalance assessment, which the MRSD model does not quantify
- Input data are junction counts or sequencing depths rather than computed MRSD result tables; generate MRSD scores with MRSDexplorer.py first

## Inputs

- Per-biosample MRSD score tables (*.results.mrsd.txt) for whole blood, LCLs, skeletal muscle, and fibroblasts, with MRSD values in millions of reads
- 295 PanelApp disease gene panel gene lists
- List of 3,322 multi-exon disease genes

## Outputs

- Fraction of genes low-MRSD in at least one biosample (expected 64.2%, 2,133/3,322)
- Per-panel optimal biosample table (panel_optimal_tissue.tsv)
- Count of panels for which each biosample is optimal (fibroblasts 186/295 = 63.1%; LCLs 126/295; skeletal muscle 70/295; whole blood 21/295)

## How to apply

Load the per-tissue MRSD result tables (*.results.mrsd.txt) with pandas, parsing MRSD values reported in millions of reads, and classify each gene as low-MRSD if its score is below the 100-million-read threshold, treating '-' entries (median splice junction coverage of 0 reads) as not low-MRSD. Aggregate per gene across biosamples to compute the fraction low-MRSD in at least one biosample and validate against the reported 64.2% (2,133/3,322). For each of the 295 PanelApp disease gene panels, intersect the panel gene list with the per-tissue classifications, compute the proportion of panel genes low-MRSD in each of the four biosamples (whole blood, LCLs, skeletal muscle, fibroblasts), and select the highest (or joint-highest) as the optimal RNA source. Summarize counts of panels per optimal biosample and write the per-panel table (e.g. panel_optimal_tissue.tsv), checking totals sum to 295 and that fibroblasts match 186/295 (63.1%).

## Related tools

- **MRSDexplorer** (Generates the *.results.mrsd.txt score tables per biosample that the proportion summaries are computed from) — https://github.com/mcgm-mrsd/mrsd-explorer
- **Python** (Used with pandas to parse MRSD tables, classify genes as low-MRSD (<100M reads), intersect with panel gene lists, and tabulate per-panel optimal biosamples)
- **MRSD** (Statistical framework producing the gene- and tissue-specific minimum required sequencing depth (in millions of reads) that the low-MRSD threshold classifies) — https://github.com/mcgm-mrsd/mrsd-explorer

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --output_prefix example transcripts.introns.bed
```

## Evaluation signals

- Fraction of genes low-MRSD in at least one biosample equals 64.2% (2,133 of 3,322 multi-exon disease genes)
- Panel counts sum to 295 and fibroblasts are optimal (or joint-optimal) for exactly 186 panels (63.1%), with LCLs 126, skeletal muscle 70, whole blood 21
- Genes with '-' MRSD entries (median splice junction coverage of 0 reads) are excluded from the low-MRSD class in every tissue
- Each panel's optimal biosample corresponds to the maximum proportion of panel genes low-MRSD across the four biosamples, with ties allowed as joint-optimal
- Per-panel table written with a stable schema (panel ID, per-tissue proportions, optimal biosample) to panel_optimal_tissue.tsv

## Limitations

- Pre-computed MRSD scores are most accurate for GTEx-style Illumina 75 bp paired-end poly-A enrichment RNA-seq processed with the Cummings et al. pipeline; other workflows require regenerated scores
- The MRSD model does not account for eQTLs/sQTLs, genomic background transcriptome-wide effects (e.g. interferonopathies, spliceosome disruption), or multiple testing issues
- The MRSD model cannot be directly extended to predict suitability for allele-specific expression or differential gene expression

## Evidence

- [abstract] Low-MRSD threshold applied to 3,322 genes across four biosamples: "We applied MRSD across four human biosamples: whole blood, lymphoblastoid cell lines (LCLs), skeletal muscle, and cultured fibroblasts"
- [abstract] Fibroblasts optimal for 63.1% of panels: "Applying MRSD scoring to established disease gene panels shows that fibroblasts, of these four biosamples, are the optimum source of RNA for 63.1% of gene panels"
- [readme] MRSD scores reported in millions of reads in results file: "followed by a list of MRSD scores (in millions of reads) for each of the successfully mapped gene/transcript identifiers, stratified by control dataset"
- [readme] '-' entries mean zero junction coverage: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [intro] MRSD scores guide biosample selection and sequencing depth: "MRSD scores (see web resources for access) can be utilized to select the most appropriate biosample to detect specific splicing aberrations and to guide required depth of sequencing."
