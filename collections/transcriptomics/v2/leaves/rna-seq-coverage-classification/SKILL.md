---
name: rna-seq-coverage-classification
description: Use when you need to assess how accurately predicted splice junction
  coverage (e.g., MRSD scores) matches achieved coverage in independent RNA-seq samples,
  using PPV/NPV as validation metrics.
when_to_use_negative:
- Goal is detecting allele-specific expression biases or differential gene expression
  — the article states the MRSD model cannot directly be extended to these
- Data are long-read, single-cell, or spatially resolved RNA-seq — no appropriate
  control datasets exist yet to develop corresponding MRSD models
- Junction counts come from a non-standard pipeline not converted to the *.junc-counts.txt
  format expected by MRSDexplorer
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: MRSDexplorer
  role: Generates MRSD scores and per-junction coverage outputs over the parameter
    grid for validation classification
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: MRSD
  role: Statistical framework predicting minimum required sequencing depth that is
    being validated against achieved coverage
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: process_cummings_output.py
  role: Merges and reformats Cummings pipeline *.csf.splicing.txt junction count files
    into *.junc-counts.txt inputs
  repo: github.com/berylc/MendelianRNA-seq
- name: Python
  role: Runs the MRSDexplorer scripts and computes the PPV/NPV classification table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:cc67e44f3bd54e561f3c129dbffba2723c026239cf0d0bb6a6f078983e806086
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  - process_cummings_output.py
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rna-seq-coverage-classification

## Summary

Validates MRSD-predicted RNA-seq depths by classifying each gene/sample as correctly covered or not covered at the predicted depth and computing positive and negative predictive values across parameter combinations.

## When to use

Use when you need to assess how accurately predicted splice junction coverage (e.g., MRSD scores) matches achieved coverage in independent RNA-seq samples, using PPV/NPV as validation metrics.

## When NOT to use

- Goal is detecting allele-specific expression biases or differential gene expression — the article states the MRSD model cannot directly be extended to these
- Data are long-read, single-cell, or spatially resolved RNA-seq — no appropriate control datasets exist yet to develop corresponding MRSD models
- Junction counts come from a non-standard pipeline not converted to the *.junc-counts.txt format expected by MRSDexplorer

## Inputs

- *.junc-counts.txt splice junction count files for independent validation samples (e.g., 12 blood, 4 LCL, 52 muscle)
- *.seq-depths.txt sequencing depth files with sample_id, total_input, uniquely_mapping, multimapping columns
- MRSDexplorer parameter grid (--number_reads, --sj_prop, --mrsd_param, --read_type unique)

## Outputs

- *.results.mrsd.txt MRSD score files per dataset and parameter combination
- *.junc-coverage.txt per-million-read junction coverage files per sample
- per-parameter-combination PPV/NPV table (e.g., mrsd_validation_ppv_npv.csv)

## How to apply

Load paired validation-cohort files (*.junc-counts.txt and *.seq-depths.txt, e.g., 68 samples: 12 blood, 4 LCL, 52 muscle) into the MRSDexplorer files/ directory, confirming sample IDs match exactly across the paired files since unmatched IDs are excluded. Run MRSDexplorer.py over the parameter grid (--number_reads, --sj_prop, --mrsd_param percentile, --read_type unique) to produce *.results.mrsd.txt score files per dataset and parameter combination, and use --output_junctions to generate *.junc-coverage.txt files listing per-million-read junction coverage for each sample. For each parameter combination, classify each gene/sample as correctly covered or not covered at the MRSD-predicted depth, then compute positive predictive value (PPV) and negative predictive value (NPV). In the published validation, PPVs ranged 90.1%-98.2% and NPVs 56.4%-94.7% (mean PPV 96%, NPV 79%), indicating the model is conservative; computed values should fall within these ranges.

## Related tools

- **MRSDexplorer** (Generates MRSD scores and per-junction coverage outputs over the parameter grid for validation classification) — github.com/mcgm-mrsd/mrsd-explorer
- **MRSD** (Statistical framework predicting minimum required sequencing depth that is being validated against achieved coverage) — github.com/mcgm-mrsd/mrsd-explorer
- **process_cummings_output.py** (Merges and reformats Cummings pipeline *.csf.splicing.txt junction count files into *.junc-counts.txt inputs) — github.com/berylc/MendelianRNA-seq
- **Python** (Runs the MRSDexplorer scripts and computes the PPV/NPV classification table)

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --read_type unique --output_junctions --output_prefix example transcripts.introns.bed
```

## Evaluation signals

- Computed PPVs across all parameter combinations fall within the reported 90.1%-98.2% range
- Computed NPVs fall within the reported 56.4%-94.7% range with mean PPV 96% and NPV 79% on the 68-sample validation cohort
- Sample IDs in *.junc-counts.txt and *.seq-depths.txt match exactly; mismatched IDs are excluded from analysis
- *.junc-coverage.txt files report per_M_coverage values for each junction/sample, and genes with median junction coverage of 0 reads return '-' as MRSD
- The per-parameter-combination PPV/NPV table (e.g., mrsd_validation_ppv_npv.csv) covers every tested (number_reads, sj_prop, mrsd_param, read_type) combination

## Limitations

- MRSD is validated only for bulk short-read poly-A enrichment RNA-seq processed with the Cummings et al. bioinformatics pipeline; other experimental/bioinformatics approaches are not characterized
- MRSD predictions may not accurately reflect coverage for transcripts in patients with disorders associated with widespread transcriptome changes (e.g., interferonopathies, chromatin structure disorders, spliceosome disruption)
- The model does not explicitly account for eQTLs or sQTLs influencing gene expression; pre-computed MRSD scores are most accurate for Illumina 75 bp paired-end poly-A workflows like GTEx

## Evidence

- [abstract] MRSD has high precision (90.1%-98.2%): "MRSD has high precision (90.1%-98.2%) and overcomes transcript region-specific sequencing biases"
- [readme] Sample IDs must match across input files: "the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file. Sample IDs identified in only one of the two files will be excluded from analysis."
- [readme] Optional junction coverage output: "If the user supplies the `--output_junctions` option when running the `MRSDexplorer.py` script, a second output file with suffix *.junc-coverage.txt* will be generated"
- [methods] MRSD is a statistical framework for sequencing depth prediction: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
