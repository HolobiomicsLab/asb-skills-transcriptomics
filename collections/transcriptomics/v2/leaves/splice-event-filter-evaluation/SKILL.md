---
name: splice-event-filter-evaluation
description: Use when you have per-sample splice-junction read counts (e.g., STAR
  split-read *.csf.splicing.txt* outputs) plus known pathogenic mis-splicing events
  and need to benchmark read-count or singleton-based event filters.
when_to_use_negative:
- Your goal is MRSD score generation itself (required sequencing depth prediction)
  rather than evaluating splicing-event filters — use MRSDexplorer.py without the
  filter-evaluation framing
- Input is long-read, single-cell, or spatially resolved RNA-seq — the MRSD framework
  is validated only for bulk short-read poly-A enrichment RNA-seq with this specific
  pipeline
- You lack RNA-seq data on samples with known pathogenic mis-splicing events, so retention
  of pathogenic events cannot be measured
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: STAR
  role: Split-read aligner used to produce per-sample *.csf.splicing.txt junction
    count files and *.Log.final.out depth statistics
  repo: https://github.com/berylc/MendelianRNA-seq
- name: process_cummings_output.py
  role: Merges and converts *.csf.splicing.txt files into the output_prefix.junc-counts.txt
    format required by MRSDexplorer
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: MRSDexplorer
  role: Run with --output_junctions to compute per-million read junction coverage
    per sample, feeding the >=2-read and singleton/NRC filter comparisons
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:c552b497b66d8562c5864da7d07b599afeef62cf36b237c501eeba86f1d519b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/berylc/MendelianRNA-seq
  tools:
  - MRSDexplorer
  - Python
  - STAR
  - process_cummings_output.py
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# splice-event-filter-evaluation

## Summary

Reproduce the MRSD paper's evaluation of splice-junction read-count and singleton (NRC > 0.25) filters, quantifying how much artifactual singleton junction events are removed while retaining known pathogenic mis-splicing events detected in 21 patient RNA-seq samples.

## When to use

Use when you have per-sample splice-junction read counts (e.g., STAR split-read *.csf.splicing.txt* outputs) plus known pathogenic mis-splicing events and need to benchmark read-count or singleton-based event filters.

## When NOT to use

- Your goal is MRSD score generation itself (required sequencing depth prediction) rather than evaluating splicing-event filters — use MRSDexplorer.py without the filter-evaluation framing
- Input is long-read, single-cell, or spatially resolved RNA-seq — the MRSD framework is validated only for bulk short-read poly-A enrichment RNA-seq with this specific pipeline
- You lack RNA-seq data on samples with known pathogenic mis-splicing events, so retention of pathogenic events cannot be measured

## Inputs

- RNA-seq FASTQs from patient datasets and 21 samples with known pathogenic mis-splicing events
- Per-sample *.csf.splicing.txt junction count files from the MendelianRNA-seq/STAR split-read pipeline
- STAR *.Log.final.out files for per-sample sequencing depths (total input, uniquely mapping, multimapping)
- List of known pathogenic mis-splicing events for retention-rate evaluation

## Outputs

- output_prefix.junc-counts.txt (merged SJID chr-start-end junction count matrix, one column per sample)
- output_prefix.seq-depths.txt (four-column per-sample depth table)
- *.junc-coverage.txt (per-million read coverage of each junction per sample)
- *.results.mrsd.txt (MRSDexplorer scores)
- splice_filter_evaluation.csv (singleton reduction and pathogenic-event retention rates)

## How to apply

Align RNA-seq FASTQs from patient datasets and the 21 samples carrying known pathogenic mis-splicing events with the Cummings et al. (2017) MendelianRNA-seq pipeline using STAR split-read alignment to yield per-sample *.csf.splicing.txt* junction count files. Merge and reformat them with process_cummings_output.py into an output_prefix.junc-counts.txt file with SJID chr-start-end rows and one column per sample, and compile per-sample sequencing depths (total input, uniquely mapping, multimapping) from STAR *.Log.final.out files into output_prefix.seq-depths.txt, ensuring sample IDs exactly match across both files. Run MRSDexplorer.py with the --output_junctions flag (e.g., --tissues all, --read_type unique) to compute per-million read coverage per junction per sample (*.junc-coverage.txt) alongside *.results.mrsd.txt. Classify detected junctions under two filtering schemes — a >=2 supporting-read threshold and a singleton/NRC > 0.25 threshold — quantifying singleton junction-event reduction and retention rate of known pathogenic events, then summarize as splice_filter_evaluation.csv. Rationale: because there is little consensus on separating true positive from artifactual splicing events, the paper quantifies filter trade-offs rather than prescribing a single cutoff.

## Related tools

- **STAR** (Split-read aligner used to produce per-sample *.csf.splicing.txt junction count files and *.Log.final.out depth statistics) — https://github.com/berylc/MendelianRNA-seq
- **process_cummings_output.py** (Merges and converts *.csf.splicing.txt files into the output_prefix.junc-counts.txt format required by MRSDexplorer) — https://github.com/mcgm-mrsd/mrsd-explorer
- **MRSDexplorer** (Run with --output_junctions to compute per-million read junction coverage per sample, feeding the >=2-read and singleton/NRC filter comparisons) — https://github.com/mcgm-mrsd/mrsd-explorer

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --output_prefix example --output_junctions transcripts.introns.bed
```

## Evaluation signals

- The >=2 supporting-read filter reduces singleton junction events by up to approximately 95%, matching Table 1 and Figure 4
- Approximately 90% of the 21 known pathogenic mis-splicing events are retained under the >=2-read / singleton (NRC > 0.25) filtering scheme
- output_prefix.junc-counts.txt has SJID rows formatted chr-start-end with exactly one column per sample
- Sample IDs in output_prefix.seq-depths.txt precisely match those in output_prefix.junc-counts.txt; IDs in only one file are excluded from analysis
- *.junc-coverage.txt contains per-million read coverage values for each junction in each sample (per_M_coverage column populated)

## Limitations

- MRSD and this filter evaluation are validated only for bulk short-read poly-A enrichment RNA-seq processed with a specific bioinformatics pipeline (Cummings et al. 2017/STAR)
- The MRSD model does not explicitly account for eQTLs or sQTLs, which are known to influence gene expression and splicing profiles
- MRSD predictions may not accurately reflect sequencing coverage for patients with disorders associated with widespread transcriptomic changes (e.g., interferonopathies, spliceosome disruption)

## Evidence

- [intro] little consensus on filtering: "there is little consensus regarding the best approach to filter true positive and pathogenic events from neutral or artifactual findings"
- [methods] Cummings pipeline junction counts: "To generate transcriptome-wide junction counts, we make use of a pipeline developed by Cummings et al. (2017"
- [readme] junc-coverage output flag: "If the user supplies the `--output_junctions` option when running the `MRSDexplorer.py` script, a second output file with suffix *.junc-coverage.txt* will be generated"
- [readme] sample ID matching requirement: "the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file"
