---
name: ppv-npv-validation-metrics-computation
description: Use when validating MRSD-predicted required sequencing depths against
  observed junction coverage in held-out samples (e.g. 12 blood, 4 LCL, 52 muscle)
  across a grid of --number_reads, --sj_prop, --mrsd_param, --read_type parameter
  combinations.
when_to_use_negative:
- Long-read, single-cell, or spatially resolved RNA-seq data, for which no appropriate
  control datasets exist yet to develop or validate MRSD models
- Datasets not generated with bulk short-read poly-A enrichment RNA-seq processed
  via the Cummings et al. split-read pipeline, unless MRSD scores are regenerated
  for the alternative workflow
- Predicting suitability for allele-specific expression or differential gene expression
  analysis, which the MRSD model cannot be directly extended to
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: MRSDexplorer
  role: Runs the MRSD score calculation over the parameter grid and generates the
    *.results.mrsd.txt and *.junc-coverage.txt files from which PPV/NPV are computed
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: MRSD
  role: Statistical framework whose predicted minimum required sequencing depths are
    being validated via PPV/NPV
- name: Python
  role: Used to run MRSDexplorer.py and to compute the per-parameter-combination PPV/NPV
    table (mrsd_validation_ppv_npv.csv)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:ecd3ae8b4533416391430acc5df8bb6e717a1a1db68250b2dbf7b5da06ed9d2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - MRSD
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ppv-npv-validation-metrics-computation

## Summary

Computes positive predictive value (PPV) and negative predictive value (NPV) of MRSD-predicted sequencing depths against observed splice junction coverage in independent RNA-seq validation samples, to quantify how conservatively the MRSD model predicts achieved junction coverage.

## When to use

Use when validating MRSD-predicted required sequencing depths against observed junction coverage in held-out samples (e.g. 12 blood, 4 LCL, 52 muscle) across a grid of --number_reads, --sj_prop, --mrsd_param, --read_type parameter combinations.

## When NOT to use

- Long-read, single-cell, or spatially resolved RNA-seq data, for which no appropriate control datasets exist yet to develop or validate MRSD models
- Datasets not generated with bulk short-read poly-A enrichment RNA-seq processed via the Cummings et al. split-read pipeline, unless MRSD scores are regenerated for the alternative workflow
- Predicting suitability for allele-specific expression or differential gene expression analysis, which the MRSD model cannot be directly extended to

## Inputs

- *.junc-counts.txt splice junction count files for 68 independent validation samples (12 blood, 4 LCL, 52 muscle)
- *.seq-depths.txt sequencing depth files (total_input, uniquely_mapping, multimapping columns) with sample IDs matching the junc-counts files
- MRSDexplorer.py parameter grid: --number_reads, --sj_prop, --mrsd_param, --read_type (unique)
- Gene list or transcript intron BED file (*.introns.bed) defining genes/transcripts of interest

## Outputs

- *.results.mrsd.txt MRSD score files per dataset and parameter combination
- *.junc-coverage.txt per-million-read junction coverage files (via --output_junctions)
- mrsd_validation_ppv_npv.csv: per-parameter-combination PPV/NPV table

## How to apply

Load the validation cohort's junction count files (*.junc-counts.txt) and sequencing depth files (*.seq-depths.txt) for the 68 samples into the MRSDexplorer files/ directory, confirming that sample IDs precisely match across the paired files since IDs present in only one file are excluded from analysis. Run MRSDexplorer.py over the parameter grid (--number_reads, --sj_prop, --mrsd_param percentile, --read_type unique) to produce *.results.mrsd.txt score files, and use the --output_junctions option to also emit *.junc-coverage.txt files listing per-million-read junction coverage for each sample. For each parameter combination, classify each gene/sample as correctly covered or not covered at the MRSD-predicted depth and compute PPV and NPV from these classifications. Finally, check that computed PPVs fall within the reported 90.1%-98.2% range and NPVs within 56.4%-94.7%, and write the per-parameter-combination table to mrsd_validation_ppv_npv.csv.

## Related tools

- **MRSDexplorer** (Runs the MRSD score calculation over the parameter grid and generates the *.results.mrsd.txt and *.junc-coverage.txt files from which PPV/NPV are computed) — https://github.com/mcgm-mrsd/mrsd-explorer
- **MRSD** (Statistical framework whose predicted minimum required sequencing depths are being validated via PPV/NPV)
- **Python** (Used to run MRSDexplorer.py and to compute the per-parameter-combination PPV/NPV table (mrsd_validation_ppv_npv.csv))

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --read_type unique --output_prefix example --output_junctions transcripts.introns.bed
```

## Evaluation signals

- Computed PPVs across all parameter combinations fall within the reported 90.1%-98.2% range
- Computed NPVs across all parameter combinations fall within the reported 56.4%-94.7% range
- Average over the 68 validation samples reproduces mean PPV ~96% and mean NPV ~79%
- Sample IDs in every *.seq-depths.txt file precisely match those in the paired *.junc-counts.txt file (mismatched IDs are silently excluded from analysis)
- mrsd_validation_ppv_npv.csv contains one PPV/NPV row per parameter combination and is populated for all grid points

## Limitations

- MRSD was validated only for bulk short-read poly-A enrichment RNA-seq processed with a specific bioinformatics pipeline (STAR split-read alignment, Illumina 75 bp paired-end GTEx-style workflow)
- MRSD predictions may not accurately reflect coverage for transcripts in patients with disorders associated with widespread transcriptome changes, and the model does not explicitly account for eQTLs or sQTLs
- Where '-' is given as a required read depth in the output, median coverage of splice junctions for the given gene was 0 reads

## Evidence

- [other] MRSD achieved 96% PPV and 79% NPV on average; across all parameter combinations PPVs ranged from 90.1% to 98.2% and NPVs from 56.4% to 94.7%: [evidence span withheld — anchor status: not_found]
- [abstract] MRSD precision claim: "MRSD has high precision (90.1%-98.2%) and overcomes transcript region-specific sequencing biases"
- [readme] Sample ID matching requirement: "Importantly, the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file."
- [readme] Junction coverage output: "If the user supplies the `--output_junctions` option when running the `MRSDexplorer.py` script, a second output file with suffix *.junc-coverage.txt* will be generated"
- [methods] STAR sequencing depth source: "when using the STAR split-read aligner, these values can be found in the *.Log.final.out* file for the run"
