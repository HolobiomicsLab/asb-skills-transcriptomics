---
name: rna-seq-sequencing-depth-quantification
description: Use when you need gene- and tissue-specific sequencing depth predictions
  (in millions of reads) for splice junction coverage of a gene panel, e.g. deciding
  which GTEx biosample and depth to use for RNA-seq mis-splicing surveillance.
when_to_use_negative:
- The query RNA-seq uses a different workflow from the GTEx control data (not Illumina
  75 bp paired-end poly-A enrichment) and was not processed through the Cummings et
  al. split-read pipeline — pre-computed MRSDs are then inaccurate; generate dataset-matched
  scores instead
- The goal is detecting allele-specific expression biases or differential gene expression
  — the MRSD model cannot be directly extended to these applications
- The disorder involves widespread transcriptomic changes (e.g. interferonopathies,
  spliceosome disruption) — MRSD predictions may not accurately reflect coverage because
  the model does not account for genomic background, eQTLs, or sQTLs
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: MRSDexplorer
  role: Python script that computes MRSD scores from merged junction counts and sequencing
    depths using --number_reads, --sj_prop, --mrsd_param, and --read_type parameters
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: process_cummings_output.py
  role: Merges and converts per-sample *.csf.splicing.txt junction count files into
    the MRSDexplorer-compatible *.junc-counts.txt format
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: transcript_bed_generator.py
  role: Generates transcript intron BED files (*.introns.bed) from a supplied GTF
    and transcript ID list for per-transcript MRSD calculation
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: STAR
  role: Split-read aligner producing *.Log.final.out sequencing depth values and the
    split-read alignments underlying junction counts
- name: MendelianRNA-seq
  role: Pipeline scripts that generate transcriptome-wide junction counts from FASTQs
    via split-read alignment (Cummings et al. 2017)
  repo: github.com/berylc/MendelianRNA-seq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:06577fb8ec31c2077faf5694727389b27a4fd12b40ebbab6688933ef9138120b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSDexplorer
  - MendelianRNA-seq
  - Python
  - STAR
  - process_cummings_output.py
  - transcript_bed_generator.py
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# rna-seq-sequencing-depth-quantification

## Summary

Quantify the Minimum Required Sequencing Depth (MRSD) per gene and tissue using the MRSDexplorer toolkit, predicting how many millions of RNA-seq reads are needed to achieve user-specified splice junction coverage in genes of interest.

## When to use

Use when you need gene- and tissue-specific sequencing depth predictions (in millions of reads) for splice junction coverage of a gene panel, e.g. deciding which GTEx biosample and depth to use for RNA-seq mis-splicing surveillance.

## When NOT to use

- The query RNA-seq uses a different workflow from the GTEx control data (not Illumina 75 bp paired-end poly-A enrichment) and was not processed through the Cummings et al. split-read pipeline — pre-computed MRSDs are then inaccurate; generate dataset-matched scores instead
- The goal is detecting allele-specific expression biases or differential gene expression — the MRSD model cannot be directly extended to these applications
- The disorder involves widespread transcriptomic changes (e.g. interferonopathies, spliceosome disruption) — MRSD predictions may not accurately reflect coverage because the model does not account for genomic background, eQTLs, or sQTLs

## Inputs

- Per-sample splice junction count files (*.csf.splicing.txt) from the Cummings et al. pipeline
- Per-sample STAR *.Log.final.out files supplying total input, uniquely mapping, and multimapping read counts
- Gene list file of GENCODE v19 gene symbols and/or Ensembl gene IDs (one gene per line, up to two columns)
- Optional transcript intron BED file (*.introns.bed) for per-transcript MRSD, generated from a GTF via transcript_bed_generator.py

## Outputs

- output_prefix.junc-counts.txt — reformatted junction count matrix (junction IDs chr-start-end as rows, sample IDs as columns)
- output_prefix.seq-depths.txt — four-column sample depth table (#sample_id, total_input, uniquely_mapping, multimapping)
- *.results.mrsd.txt — MRSD scores in millions of reads per gene/transcript, stratified by control biosample dataset
- Optional *.junc-coverage.txt — per-million read coverage of each junction per sample (with --output_junctions)

## How to apply

Generate transcriptome-wide junction counts from FASTQs via the Cummings et al. split-read alignment pipeline (STAR), then merge per-sample *.csf.splicing.txt files with process_cummings_output.py into a output_prefix.junc-counts.txt file with splice junction IDs (chr-start-end) as rows and sample IDs as columns. Compile per-sample depths (total input, uniquely mapping, multimapping reads from STAR *.Log.final.out files) into a matching output_prefix.seq-depths.txt file whose sample_id column exactly matches the junction count file. Provide a gene list file of GENCODE v19 gene symbols and/or Ensembl gene IDs (one per line, up to two columns), or a transcript intron BED generated with transcript_bed_generator.py. Move the .junc-counts.txt/.seq-depths.txt pair into the files/ subdirectory and run MRSDexplorer.py, choosing --number_reads (default 8), --sj_prop (default 0.75), --mrsd_param (default 0.95, i.e. the 95th percentile among control samples) and --read_type (default unique). Collect the *.results.mrsd.txt output reporting MRSD scores in millions of reads per gene, stratified by control biosample dataset; genes whose median junction coverage was 0 reads receive '-'.

## Related tools

- **MRSDexplorer** (Python script that computes MRSD scores from merged junction counts and sequencing depths using --number_reads, --sj_prop, --mrsd_param, and --read_type parameters) — github.com/mcgm-mrsd/mrsd-explorer
- **process_cummings_output.py** (Merges and converts per-sample *.csf.splicing.txt junction count files into the MRSDexplorer-compatible *.junc-counts.txt format) — github.com/mcgm-mrsd/mrsd-explorer
- **transcript_bed_generator.py** (Generates transcript intron BED files (*.introns.bed) from a supplied GTF and transcript ID list for per-transcript MRSD calculation) — github.com/mcgm-mrsd/mrsd-explorer
- **STAR** (Split-read aligner producing *.Log.final.out sequencing depth values and the split-read alignments underlying junction counts)
- **MendelianRNA-seq** (Pipeline scripts that generate transcriptome-wide junction counts from FASTQs via split-read alignment (Cummings et al. 2017)) — github.com/berylc/MendelianRNA-seq

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 8 --sj_prop 0.75 --mrsd_param 0.95 --read_type unique genes_of_interest.txt
```

## Evaluation signals

- The *.results.mrsd.txt header reports the input parameters echoed back (read type, desired coverage 8 reads, sj proportion 0.75, confidence level 0.95) and the number of genes successfully cross-referenced (e.g. 3322 for the PanelApp panel)
- Every sample_id in the .seq-depths.txt file exactly matches a sample column in the .junc-counts.txt file; IDs present in only one file are silently excluded from analysis
- MRSD scores are reported in millions of reads, and '-' appears only where the median junction coverage for the gene was 0 reads
- Reproducing the published benchmark yields low-MRSD (<100 M reads) fractions of 28.2% in whole blood, 49.4% in LCLs, 43.6% in skeletal muscle, 53.7% in fibroblasts, with 64.2% low-MRSD in at least one tissue
- A non-numerical placeholder (e.g. '-') fills unused depth columns; the file must still pass input validation with all four columns populated

## Limitations

- MRSD predictions may not accurately reflect coverage for disorders associated with widespread transcriptomic changes (e.g. interferonopathies, chromatin structure disorders, spliceosome disruption)
- The current MRSD model does not explicitly account for eQTLs or sQTLs, which influence gene expression profiles; dynamic score generation accounting for these and multiple testing is future work
- Pre-computed scores are calibrated to GTEx Illumina 75 bp paired-end poly-A enrichment bulk short-read RNA-seq processed with a specific bioinformatics pipeline; long-read, single-cell, and spatially resolved RNA-seq lack appropriate control datasets
- The MRSD model cannot be directly extended to predict suitability for detecting allele-specific expression biases or differential gene expression
- Sample IDs must precisely match between .junc-counts.txt and .seq-depths.txt; mismatched IDs are excluded from analysis

## Evidence

- [readme] MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
- [methods] depths from STAR logs: "To calculate MRSD, a text file must be provided detailing the sequencing depths of the samples listed in the *.junc-counts.txt* file"
- [readme] default parameters: [evidence span withheld — anchor status: not_found]
- [readme] output format: "followed by a list of MRSD scores (in millions of reads) for each of the successfully mapped gene/transcript identifiers, stratified by control dataset."
- [readme] GTEx sequencing workflow caveat: "GTEx samples are sequenced using an Illumina 75 bp paired-end read poly-A enrichment workflow. As such, the pre-computed MRSDs are most accurate for assessing transcript coverage with similar RNA-seq"
- [abstract] validation of MRSD precision: "MRSD has high precision (90.1%-98.2%) and overcomes transcript region-specific sequencing biases"
