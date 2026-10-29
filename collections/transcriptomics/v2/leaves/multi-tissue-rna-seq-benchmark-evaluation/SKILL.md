---
name: multi-tissue-rna-seq-benchmark-evaluation
description: 'Use when MRSD-predicted sequencing depths must be validated on independent
  RNA-seq cohorts: you have paired junction-count and sequencing-depth files for control
  biosamples and need PPV/NPV per parameter combination.'
when_to_use_negative:
- 'The RNA-seq data are long-read, single-cell, or spatially resolved: no appropriate
  control datasets exist yet to develop corresponding MRSD models'
- 'The goal is predicting suitability for detecting allele-specific expression biases
  or differential gene expression: the MRSD model cannot be directly extended to these'
- 'Sequencing methodology differs from Illumina 75 bp paired-end poly-A enrichment
  (e.g. total RNA, ribo-depletion): pre-computed GTEx-derived MRSDs are most accurate
  only for similar workflows, so scores must be regenerated rather than benchmarked
  as-is'
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MRSDexplorer
  role: Runs the MRSD calculation over the parameter grid and produces *.results.mrsd.txt
    and *.junc-coverage.txt outputs used for PPV/NPV benchmarking
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: MRSD
  role: Statistical framework being evaluated; predicts sequencing depth required
    for user-specified splice junction coverage
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: Runs MRSDexplorer.py and computes the per-combination PPV/NPV classification
    and validation table
  repo: github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:972e81e87e93ecaee87290f2ca66730c1f77bd23531b889a6e41b37fabce4dba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multi-tissue-rna-seq-benchmark-evaluation

## Summary

Benchmark evaluation of MRSD splice-junction coverage predictions against independent RNA-seq validation samples by sweeping the MRSDexplorer parameter grid (--number_reads, --sj_prop, --mrsd_param, --read_type) and computing per-combination PPV/NPV over 68 biosamples (12 blood, 4 LCL, 52 muscle).

## When to use

Use when MRSD-predicted sequencing depths must be validated on independent RNA-seq cohorts: you have paired junction-count and sequencing-depth files for control biosamples and need PPV/NPV per parameter combination.

## When NOT to use

- The RNA-seq data are long-read, single-cell, or spatially resolved: no appropriate control datasets exist yet to develop corresponding MRSD models
- The goal is predicting suitability for detecting allele-specific expression biases or differential gene expression: the MRSD model cannot be directly extended to these
- Sequencing methodology differs from Illumina 75 bp paired-end poly-A enrichment (e.g. total RNA, ribo-depletion): pre-computed GTEx-derived MRSDs are most accurate only for similar workflows, so scores must be regenerated rather than benchmarked as-is

## Inputs

- *.junc-counts.txt merged splice junction count files (SJID rows, per-sample columns; e.g. chr1-11995-899764)
- *.seq-depths.txt sequencing depth files with columns #sample_id, total_input, uniquely_mapping, multimapping
- Gene list (.genes.txt with GENCODE v19 symbol and/or Ensembl ID) or transcript intron BED (.introns.bed) defining coverage targets
- Parameter grid over --number_reads, --sj_prop, --mrsd_param, --read_type

## Outputs

- *.results.mrsd.txt MRSD score files (scores in millions of reads) per dataset and parameter combination
- *.junc-coverage.txt per-million-read junction coverage listings per sample
- mrsd_validation_ppv_npv.csv per-parameter-combination PPV/NPV table

## How to apply

Load validation-cohort *.junc-counts.txt and *.seq-depths.txt files (68 samples: 12 blood, 4 LCL, 52 muscle) into MRSDexplorer's files/ directory, confirming sample IDs match across paired files since IDs found in only one file are excluded. Run MRSDexplorer.py across the parameter grid (--number_reads, --sj_prop, --mrsd_param percentile, --read_type unique) to emit *.results.mrsd.txt score files per dataset/parameter combination. Supply --output_junctions to generate *.junc-coverage.txt per-million-read junction coverage files. For each parameter combination, classify each gene/sample as correctly covered or not covered at the MRSD-predicted depth, compute PPV and NPV, and confirm PPVs fall in the reported 90.1%-98.2% range and NPVs in 56.4%-94.7%, writing the table to mrsd_validation_ppv_npv.csv.

## Related tools

- **MRSDexplorer** (Runs the MRSD calculation over the parameter grid and produces *.results.mrsd.txt and *.junc-coverage.txt outputs used for PPV/NPV benchmarking) — github.com/mcgm-mrsd/mrsd-explorer
- **MRSD** (Statistical framework being evaluated; predicts sequencing depth required for user-specified splice junction coverage) — github.com/mcgm-mrsd/mrsd-explorer
- **Python** (Runs MRSDexplorer.py and computes the per-combination PPV/NPV classification and validation table) — github.com/mcgm-mrsd/mrsd-explorer

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --read_type unique --output_prefix example transcripts.introns.bed
```

## Evaluation signals

- Computed PPVs across all parameter combinations fall within the reported 90.1%-98.2% precision range
- Computed NPVs fall within the reported 56.4%-94.7% range, reflecting the model's conservative behaviour (mean NPV 79%)
- Average PPV across the 68 validation samples approximates the reported mean of 96%
- Every sample_id in *.seq-depths.txt precisely matches an ID in *.junc-counts.txt; mismatched IDs are silently excluded from analysis
- Both *.results.mrsd.txt and *.junc-coverage.txt files are generated for each parameter combination and genes with '-' MRSD indicate median junction coverage of 0 reads

## Limitations

- MRSD predictions may not accurately reflect sequencing coverage for transcripts in patients with disorders involving widespread transcriptomic changes (e.g. interferonopathies, chromatin structure disorders, spliceosome disruption)
- The current MRSD model does not explicitly account for eQTLs or sQTLs influencing gene expression and splicing profiles
- Validation is limited to bulk short-read poly-A enrichment RNA-seq processed with a specific bioinformatics pipeline; long-read, single-cell, and spatially resolved approaches are not yet characterized

## Evidence

- [abstract] MRSD has high precision (90.1%-98.2%) and overcomes transcript region-specific sequencing biases: "MRSD has high precision (90.1%-98.2%) and overcomes transcript region-specific sequencing biases"
- [readme] Sample ID matching requirement: "Importantly, the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file. Sample IDs identified in only one of the two files will be excluded from"
- [readme] Junction coverage output option: "If the user supplies the `--output_junctions` option when running the `MRSDexplorer.py` script, a second output file with suffix *.junc-coverage.txt* will be generated"
- [discussion] Model does not account for eQTLs/sQTLs: "the current MRSD model does not explicitly account for the presence of expression quantitative trait loci (eQTLs) or splicing quantitative trait loci (sQTLs)"
- [readme] PPV/NPV parameter defaults from README: "`--number_reads` Desired number of reads to cover proportion of splice junctions specified in `--sj_prop` (default = 8) `--sj_prop` Desired proportion of transcript splice junctions"
