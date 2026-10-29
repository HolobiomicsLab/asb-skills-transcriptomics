---
name: star-split-read-alignment
description: Use when RNA-seq FASTQs must be converted into per-sample splice junction
  counts (chr-start-end SJIDs) for MRSD calculation, junction coverage estimation,
  or evaluation of read-count/singleton filters on mis-splicing events.
when_to_use_negative:
- 'Long-read, single-cell, or spatially resolved RNA-seq: the article notes appropriate
  control datasets for these modalities have not yet been generated, so the Cummings/STAR
  pipeline outputs are not calibrated for them'
- 'Analyses requiring allele-specific expression or differential gene expression quantification:
  the junction-count output is designed for splice junction coverage, not gene/allele-level
  expression'
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: STAR
  role: Split-read aligner used within the MendelianRNA-seq pipeline to produce transcriptome-wide
    splice junction counts; its *.Log.final.out file supplies the sequencing depth
    values needed for MRSD
  repo: https://github.com/berylc/MendelianRNA-seq
- name: process_cummings_output.py
  role: Merges and converts the *.csf.splicing.txt files into the *.junc-counts.txt
    format required by MRSDexplorer
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:ef01c453285e32441507dccd5a428d840cf9ef58adf7d237eca928124306629f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/berylc/MendelianRNA-seq
  tools:
  - MRSDexplorer
  - Python
  - STAR
  - process_cummings_output.py
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# STAR split-read alignment for splice junction read counting

## Summary

Generates transcriptome-wide splice junction read counts from RNA-seq FASTQs by running the Cummings et al. (2017) MendelianRNA-seq pipeline with STAR split-read alignment, producing per-sample *.csf.splicing.txt junction count files that feed into MRSDexplorer and splicing-event filtering analyses.

## When to use

Use when RNA-seq FASTQs must be converted into per-sample splice junction counts (chr-start-end SJIDs) for MRSD calculation, junction coverage estimation, or evaluation of read-count/singleton filters on mis-splicing events.

## When NOT to use

- Long-read, single-cell, or spatially resolved RNA-seq: the article notes appropriate control datasets for these modalities have not yet been generated, so the Cummings/STAR pipeline outputs are not calibrated for them
- Analyses requiring allele-specific expression or differential gene expression quantification: the junction-count output is designed for splice junction coverage, not gene/allele-level expression

## Inputs

- RNA-seq FASTQ files (bulk short-read poly-A enriched, e.g. Illumina 75 bp paired-end)
- STAR aligner with split-read alignment settings from the MendelianRNA-seq pipeline
- Reference genome/annotation (pipeline-compatible, e.g. GENCODE human v19 for downstream transcript models)

## Outputs

- Per-sample *.csf.splicing.txt splice junction count files
- STAR *.Log.final.out alignment log files with total input, uniquely mapping, and multimapping read counts

## How to apply

Align the RNA-seq FASTQs from patient samples and control datasets (e.g. GTEx-derived or the 21 samples with known pathogenic mis-splicing events) using the Cummings et al. (2017) MendelianRNA-seq pipeline with STAR split-read alignment, which yields per-sample *.csf.splicing.txt junction count files. Record the sequencing depth statistics (total input, uniquely mapping, multimapping reads) from the STAR *.Log.final.out file for each run, since these are required downstream to compute MRSD. The resulting count files are then merged and reformatted with process_cummings_output.py into a *.junc-counts.txt file whose rows are SJIDs of the form chr-start-end with one column per sample. Sample IDs from the *.Log.final.out-derived depths must exactly match those in the junc-counts file or samples are excluded from analysis.

## Related tools

- **STAR** (Split-read aligner used within the MendelianRNA-seq pipeline to produce transcriptome-wide splice junction counts; its *.Log.final.out file supplies the sequencing depth values needed for MRSD) — https://github.com/berylc/MendelianRNA-seq
- **process_cummings_output.py** (Merges and converts the *.csf.splicing.txt files into the *.junc-counts.txt format required by MRSDexplorer) — https://github.com/mcgm-mrsd/mrsd-explorer

## Examples

```
python process_cummings_output.py output_list output_prefix  # merges STAR *.csf.splicing.txt files into output_prefix.junc-counts.txt
```

## Evaluation signals

- Each sample produces a *.csf.splicing.txt junction count file with one row per junction (chr-start-end SJID format)
- Each STAR run has a *.Log.final.out file containing total input, uniquely mapping, and multimapping read counts that can be compiled into a *.seq-depths.txt file
- Sample IDs from the *.Log.final.out files must precisely match those in the merged *.junc-counts.txt file; mismatched IDs are silently excluded from analysis
- After merging with process_cummings_output.py, the *.junc-counts.txt file should have an SJID column followed by one integer count column per sample, matching the documented example format
- When applied to the 21 patient samples with known pathogenic mis-splicing events, downstream filtering should retain ~90% of known pathogenic events at >=2 supporting reads / NRC > 0.25 and remove up to ~95% of singleton junction events (Table 1, Figure 4)

## Limitations

- MRSD scores (and thus the junction counts feeding them) are calibrated for bulk short-read poly-A enrichment RNA-seq processed with this specific bioinformatics pipeline; other experimental or bioinformatic approaches are not yet characterized
- Junction counts from alternative alignment approaches may be used only if converted to the identical *.junc-counts.txt format
- Splice junctions with median coverage of 0 reads return '-' as the required depth in MRSD output, so ultra-low-coverage genes yield no usable MRSD

## Evidence

- [methods] Cummings et al. pipeline split-read alignment: "To generate transcriptome-wide junction counts, we make use of a pipeline developed by Cummings et al. (2017"
- [methods] STAR Log.final.out depth values: "when using the STAR split-read aligner, these values can be found in the *.Log.final.out* file for the run"
- [readme] MendelianRNA-seq repo with computational steps: "The computational steps required to produce a count file from FASTQs, including split-read alignment, are described in detail"
- [readme] Sample ID matching requirement: "the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file. Sample IDs identified in only one of the two files will be excluded"
