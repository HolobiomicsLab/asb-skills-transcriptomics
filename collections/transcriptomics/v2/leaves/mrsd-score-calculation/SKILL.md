---
name: mrsd-score-calculation
description: Use when you need gene- or transcript-specific predictions of RNA-seq
  depth required to survey splice junctions (e.g. low-MRSD <100 M reads classification)
  from bulk short-read poly-A RNA-seq reference datasets.
when_to_use_negative:
- Your sequencing workflow is long-read, single-cell, or spatially resolved RNA-seq,
  or total-RNA/other library prep — appropriate control datasets to develop corresponding
  MRSD models are not yet available
- You need suitability predictions for allele-specific expression, differential gene
  expression, or isoform imbalance — the MRSD model cannot directly be extended to
  these
- Patient disorders involve widespread transcriptomic changes (e.g. interferonopathies,
  chromatin structure disorders, spliceosome disruption) — MRSD predictions may not
  accurately reflect coverage in such genomic backgrounds
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3170
tools:
- name: MRSDexplorer
  role: Main script that computes MRSD scores from junction counts, sequencing depths,
    and a gene list or transcript BED
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: process_cummings_output.py
  role: Merges and reformats per-sample *.csf.splicing.txt files into a *.junc-counts.txt
    compatible with MRSDexplorer
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: transcript_bed_generator.py
  role: Generates transcript intron BED files (*.introns.bed) from a supplied GTF
    and transcript ID list
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: STAR
  role: Split-read aligner used in the Cummings et al. pipeline; *.Log.final.out supplies
    sample sequencing depths
  repo: https://github.com/berylc/MendelianRNA-seq
- name: MRSD
  role: Statistical framework predicting RNA-seq depth required for user-specified
    splice junction coverage
  repo: github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:a0c134412946310ea90b99ecf832997e40f8e94cfd5cf667b2f8161a788a6471
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  - STAR
  - process_cummings_output.py
  - transcript_bed_generator.py
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# mrsd-score-calculation

## Summary

Calculate Minimum Required Sequencing Depth (MRSD) scores that quantify, in millions of reads, the RNA-seq depth needed to cover a user-specified proportion of splice junctions for genes or transcripts of interest, stratified by control biosample (whole blood, LCLs, skeletal muscle, fibroblasts from GTEx).

## When to use

Use when you need gene- or transcript-specific predictions of RNA-seq depth required to survey splice junctions (e.g. low-MRSD <100 M reads classification) from bulk short-read poly-A RNA-seq reference datasets.

## When NOT to use

- Your sequencing workflow is long-read, single-cell, or spatially resolved RNA-seq, or total-RNA/other library prep — appropriate control datasets to develop corresponding MRSD models are not yet available
- You need suitability predictions for allele-specific expression, differential gene expression, or isoform imbalance — the MRSD model cannot directly be extended to these
- Patient disorders involve widespread transcriptomic changes (e.g. interferonopathies, chromatin structure disorders, spliceosome disruption) — MRSD predictions may not accurately reflect coverage in such genomic backgrounds

## Inputs

- Per-sample STAR *.csf.splicing.txt splice junction count files
- Per-sample sequencing depths from STAR *.Log.final.out files
- Gene list file (one per line, GENCODE v19 gene symbol and/or Ensembl gene ID) or transcript intron BED file
- GENCODE v19 GTF annotation (for transcript_bed_generator.py)

## Outputs

- *.junc-counts.txt merged junction count matrix (SJID rows, sample columns)
- *.seq-depths.txt sequencing depth table (sample_id, total_input, uniquely_mapping, multimapping)
- *.results.mrsd.txt MRSD scores in millions of reads per gene/transcript, stratified by control dataset
- *.junc-coverage.txt per-junction per-million-read coverage (with --output_junctions)

## How to apply

Prepare three inputs per reference dataset: (1) a merged splice-junction count file (*.junc-counts.txt) with SJIDs (chr-start-end) as rows and sample IDs as columns, produced by aligning FASTQs with the STAR split-read aligner via the Cummings et al. (2017) pipeline and merging per-sample *.csf.splicing.txt files with process_cummings_output.py; (2) a matching *.seq-depths.txt with four columns (sample_id, total_input, uniquely_mapping, multimapping) extracted from STAR *.Log.final.out files, where sample IDs must precisely match the junction count file; and (3) a one-per-line gene list (GENCODE v19 symbol and/or Ensembl ID) or a transcript intron BED generated by transcript_bed_generator.py from a GTF. Move the file pairs into the files/ subdirectory and run MRSDexplorer.py with parameters such as --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique. MRSD scores leverage variation in sample read depth across controls to return the 95th-percentile depth required for 8-read coverage of 75% of splice junctions, output in *.results.mrsd.txt in millions of reads; '-' entries indicate genes whose median junction coverage was 0 reads.

## Related tools

- **MRSDexplorer** (Main script that computes MRSD scores from junction counts, sequencing depths, and a gene list or transcript BED) — github.com/mcgm-mrsd/mrsd-explorer
- **process_cummings_output.py** (Merges and reformats per-sample *.csf.splicing.txt files into a *.junc-counts.txt compatible with MRSDexplorer) — github.com/mcgm-mrsd/mrsd-explorer
- **transcript_bed_generator.py** (Generates transcript intron BED files (*.introns.bed) from a supplied GTF and transcript ID list) — github.com/mcgm-mrsd/mrsd-explorer
- **STAR** (Split-read aligner used in the Cummings et al. pipeline; *.Log.final.out supplies sample sequencing depths) — https://github.com/berylc/MendelianRNA-seq
- **MRSD** (Statistical framework predicting RNA-seq depth required for user-specified splice junction coverage) — github.com/mcgm-mrsd/mrsd-explorer

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --output_prefix example transcripts.introns.bed
```

## Evaluation signals

- Output *.results.mrsd.txt reports scores in millions of reads with input parameters echoed (e.g. read type unique, coverage 8, sj_prop 0.75, confidence 0.95) and includes summary statistics such as 'Number of genes successfully cross-referenced: 3322'
- Sample IDs in *.seq-depths.txt exactly match those in *.junc-counts.txt; IDs present in only one file are excluded from analysis
- Genes with median splice junction coverage of 0 reads appear as '-' in the MRSD output rather than a numeric score
- Applied to the 3,322 multi-exon PanelApp genes, per-tissue low-MRSD (<100 M reads) fractions should match the reported values: 28.2% (whole blood), 49.4% (LCLs), 43.6% (skeletal muscle), 53.7% (fibroblasts), 64.2% in at least one tissue
- Paired 150 bp vs 75 bp LCL reference comparison should reproduce ~86.5% (1,559/1,802) of surveyable genes with lower MRSD under 150 bp reads

## Limitations

- Pre-computed MRSDs are most accurate for Illumina 75 bp paired-end poly-A enrichment workflows like GTEx; alternative methodologies require generating custom scores
- MRSD does not explicitly account for eQTLs, sQTLs, or genomic background effects that alter the transcriptome
- MRSD was validated only for bulk short-read poly-A RNA-seq processed with the Cummings et al. bioinformatics pipeline

## Evidence

- [readme] MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage of a transcript(s) of interest.: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage of a transcript(s) of interest."
- [readme] GTEx 75bp PE workflow caveat: "GTEx samples are sequenced using an Illumina 75 bp paired-end read poly-A enrichment workflow. As such, the pre-computed MRSDs are most accurate for assessing transcript coverage with similar RNA-seq"
- [readme] Output file and '-' semantics: "The primary output file of `MRSDexplorer.py` is a text file with the suffix *.results.mrsd.txt*."
- [readme] Sequencing depth file format and ID matching: "Importantly, the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file."
- [methods] STAR log as depth source: "when using the STAR split-read aligner, these values can be found in the *.Log.final.out* file for the run"
- [other] Default parameter usage in task workflow: [evidence span withheld — anchor status: not_found]
- [abstract] MRSD precision and scope: "MRSD has high precision (90.1%-98.2%) and overcomes transcript region-specific sequencing biases"
