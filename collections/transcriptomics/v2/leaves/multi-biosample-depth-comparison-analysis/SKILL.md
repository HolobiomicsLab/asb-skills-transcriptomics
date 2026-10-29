---
name: multi-biosample-depth-comparison-analysis
description: Use when you need per-gene, per-tissue predicted RNA-seq depths (MRSD
  scores) for splice-junction coverage of a disease gene panel across several control
  biosample datasets.
when_to_use_negative:
- When assessing suitability for detecting allele-specific expression biases or differential
  gene expression — the article states the MRSD model cannot directly be extended
  to those tasks
- When the planned data are long-read, single-cell, or spatially resolved RNA-seq
  — no appropriate control datasets yet exist to develop corresponding MRSD models
- When sequencing workflows differ substantially from Illumina 75 bp paired-end poly-A
  enrichment, unless you regenerate your own MRSD scores for that methodology
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: MRSDexplorer
  role: Generates MRSD scores for genes/transcripts across biosample control datasets
    given .junc-counts.txt and .seq-depths.txt inputs
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: process_cummings_output.py
  role: Merges and converts per-sample *.csf.splicing.txt junction count files into
    the MRSDexplorer-compatible .junc-counts.txt format
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: STAR
  role: Split-read aligner producing the *.Log.final.out files from which sample sequencing
    depths are taken
- name: MRSD
  role: Statistical framework predicting RNA-seq depth required to achieve user-specified
    splice junction coverage, applied per biosample
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:a178222bfe732961f76cbfd6eca2d2e35455fc6317ffe9b8b6f35ab4e13282e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  - STAR
  - process_cummings_output.py
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# multi-biosample-depth-comparison-analysis

## Summary

Generates MRSD (Minimum Required Sequencing Depth) scores in millions of reads for a list of genes across multiple GTEx control biosamples (whole blood, LCLs, skeletal muscle, cultured fibroblasts) using the MRSDexplorer toolkit, so the depth needed to cover splice junctions can be compared per tissue.

## When to use

Use when you need per-gene, per-tissue predicted RNA-seq depths (MRSD scores) for splice-junction coverage of a disease gene panel across several control biosample datasets.

## When NOT to use

- When assessing suitability for detecting allele-specific expression biases or differential gene expression — the article states the MRSD model cannot directly be extended to those tasks
- When the planned data are long-read, single-cell, or spatially resolved RNA-seq — no appropriate control datasets yet exist to develop corresponding MRSD models
- When sequencing workflows differ substantially from Illumina 75 bp paired-end poly-A enrichment, unless you regenerate your own MRSD scores for that methodology

## Inputs

- Per-sample splice junction count files with suffix *.csf.splicing.txt from each GTEx biosample (from the Cummings et al. pipeline)
- STAR *.Log.final.out files per sample for sequencing depths (total input, uniquely mapping, multimapping reads)
- Gene list file with GENCODE v19 gene symbols and/or Ensembl gene IDs, one per line (e.g. 3,322 multi-exon PanelApp genes)

## Outputs

- output_prefix.junc-counts.txt merged junction count matrix (SJID rows chr-start-end, sample ID columns)
- output_prefix.seq-depths.txt sample depth file (#sample_id, total_input, uniquely_mapping, multimapping)
- *.results.mrsd.txt files reporting MRSD scores in millions of reads per gene, stratified by control biosample dataset
- Optionally *.junc-coverage.txt per-junction per-million coverage when --output_junctions is supplied

## How to apply

Merge per-sample *.csf.splicing.txt junction count files from each GTEx biosample with process_cummings_output.py into one output_prefix.junc-counts.txt (splice junction IDs as chr-start-end rows, sample IDs as columns), and compile STAR *.Log.final.out depths (total input, uniquely mapping, multimapping) into an output_prefix.seq-depths.txt whose sample_id column exactly matches the junction counts file. Build a gene-of-interest file listing the multi-exon PanelApp genes as GENCODE v19 gene symbols and/or Ensembl gene IDs, one per line (up to two columns). Move the .junc-counts.txt and .seq-depths.txt pairs into the files/ subdirectory of MRSDexplorer and run MRSDexplorer.py with the standardized parameters --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique. Collect the resulting *.results.mrsd.txt files, which report MRSD scores in millions of reads per gene stratified by control biosample dataset; genes with MRSD < 100 M reads are considered low-MRSD and amenable to RNA-seq.

## Related tools

- **MRSDexplorer** (Generates MRSD scores for genes/transcripts across biosample control datasets given .junc-counts.txt and .seq-depths.txt inputs) — github.com/mcgm-mrsd/mrsd-explorer
- **process_cummings_output.py** (Merges and converts per-sample *.csf.splicing.txt junction count files into the MRSDexplorer-compatible .junc-counts.txt format) — github.com/mcgm-mrsd/mrsd-explorer
- **STAR** (Split-read aligner producing the *.Log.final.out files from which sample sequencing depths are taken)
- **MRSD** (Statistical framework predicting RNA-seq depth required to achieve user-specified splice junction coverage, applied per biosample)

## Examples

```
python MRSDexplorer.py --tissues whole_blood,LCLs,skeletal_muscle,fibroblasts --number_reads 8 --sj_prop 0.75 --mrsd_param 0.95 --read_type unique panelapp_genes.txt
```

## Evaluation signals

- Output *.results.mrsd.txt begins with an INPUT PARAMETERS block echoing --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique, and the selected datasets
- Number of genes successfully cross-referenced in the results file matches the expected panel size (e.g. 3,322 multi-exon PanelApp genes)
- sample_id column of .seq-depths.txt exactly matches sample IDs in .junc-counts.txt; IDs present in only one file are excluded
- MRSD values are reported in millions of reads; a '-' means median splice junction coverage was 0 reads for that gene
- Per-tissue low-MRSD (<100 M reads) fractions should be consistent with published benchmarks: 28.2% whole blood, 49.4% LCLs, 43.6% skeletal muscle, 53.7% fibroblasts, 64.2% in at least one tissue

## Limitations

- MRSD is validated only for bulk short-read poly-A enrichment RNA-seq (GTEx Illumina 75 bp paired-end workflow) processed with a specific bioinformatics pipeline; other approaches are not characterized
- MRSD predictions may not accurately reflect coverage for disorders with widespread transcriptomic changes (e.g. interferonopathies, chromatin structure disorders, spliceosome disruption)
- The model does not explicitly account for eQTLs or sQTLs, and '-' values arise when median junction coverage among control samples is 0 reads

## Evidence

- [results] At the individual tissue level, 28.2% of PanelApp genes in whole blood, 49.4% in LCLs, 43.6% in skeletal muscle, and 53.7% in cultured fibroblasts were predicted to be low-MRSD: [evidence span withheld — anchor status: not_found]
- [readme] GTEx poly-A workflow caveat: "GTEx samples are sequenced using an Illumina 75 bp paired-end read poly-A enrichment workflow. As such, the pre-computed MRSDs are most accurate for assessing transcript coverage with similar RNA-seq"
- [readme] Sample ID matching requirement: "Importantly, the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file. Sample IDs identified in only one of the two files will be excluded from"
- [readme] MRSDexplorer parameter defaults: "Desired number of reads to cover proportion of splice junctions specified in `--sj_prop` (default = 8)"
- [methods] MRSD definition: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
