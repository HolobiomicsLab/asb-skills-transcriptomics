---
name: qclash-replicate-data-pooling-and-filtering
description: Use when multiple qCLASH or CLASH replicate hyb files (with associated
  vienna/fold files) must be QC-checked, filtered, and pooled for grouped analysis
  of miRNA targets across replicates.
when_to_use_negative:
- Data is a single experimental replicate — use Hybkit's single-replicate Target Analysis
  instead of Grouped Target Analysis.
- Input is not hyb-format hybrid sequence data (e.g., raw FASTQ reads or pre-computed
  count tables) — hybkit operates on 'hyb' files and associated vienna/CT fold files.
- Records have not been segment-type annotated (eval_types/eval_mirna not run) and
  no segment-based filtering or miRNA identification is required.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: Hybkit
  role: Core toolkit providing command-line utilities (hyb_check, hyb_filter, hyb_analyze)
    and the Python3 API (HybFile, FoldRecord, HybFoldIter, hybkit.analysis, hybkit.plot)
    used for QC, filtering, and pooled replicate analysis
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime environment (Python3.8+) for the Hybkit API-based pooling and analysis
    scripts
- name: matplotlib
  role: Dependency used by hybkit.plot to render grouped target-analysis plots
- name: BioPython
  role: Hybkit dependency (>= 1.79) supporting sequence record handling in the API
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:38d6332150eb45a830d27bab4686f460dea9e5e6a522930707e2c8b8539cf605
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Hybkit
  - Python3
  - matplotlib
  - typing_extensions
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# qCLASH replicate data pooling and filtering

## Summary

A Hybkit-based workflow for pooling qCLASH hybrid ('hyb' + fold) data across multiple experimental replicates (e.g., 6 KSHV miRNA qCLASH replicates from GEO GSE101978) after quality checking and removal of artifactual ribosomal- and mitochondrial-RNA hybrids, to enable grouped target analysis of a set of miRNAs.

## When to use

Use when multiple qCLASH or CLASH replicate hyb files (with associated vienna/fold files) must be QC-checked, filtered, and pooled for grouped analysis of miRNA targets across replicates.

## When NOT to use

- Data is a single experimental replicate — use Hybkit's single-replicate Target Analysis instead of Grouped Target Analysis.
- Input is not hyb-format hybrid sequence data (e.g., raw FASTQ reads or pre-computed count tables) — hybkit operates on 'hyb' files and associated vienna/CT fold files.
- Records have not been segment-type annotated (eval_types/eval_mirna not run) and no segment-based filtering or miRNA identification is required.

## Inputs

- hyb-format hybrid RNA sequence files from multiple qCLASH replicates (e.g., 6 replicates from GEO GSE101978)
- Associated vienna (or CT) fold files for each replicate
- hybkit toolkit (command-line utilities or Python3 API)
- Optional segment-string filter criteria (e.g., 'kshv') and ribosomal/mitochondrial removal settings

## Outputs

- QC-passed, filtered hyb records pooled across replicates
- Grouped target-analysis summary tables in CSV form (hybrid-target counts, miRNA target distributions)
- Plots of grouped target analysis results (e.g., PNG charts via hybkit.plot)

## How to apply

Load each of the replicate hyb files and associated fold files from the GEO accession (e.g., GSE101978) using hybkit's HybFile/FoldFile iterators or the HybFoldIter class. First run quality control with hyb_check to verify data integrity of the hyb and fold files. Then use hyb_filter to remove artifactual ribosomal- and mitochondrial-RNA hybrids (optionally also filtering duplicates by HybRecord.id, or subsetting with filters such as 'any_seg_contains kshv'). Evaluate each remaining record with HybRecord.eval_types() and eval_mirna() to assign segment-type and miRNA annotations. Finally, pool the filtered, evaluated records from all replicates and run hyb_analyze / hybkit.analysis.Analysis grouped target analysis to quantify targets of the miRNA set across pooled replicates, producing summary CSV tables and plots via hybkit.plot.

## Related tools

- **Hybkit** (Core toolkit providing command-line utilities (hyb_check, hyb_filter, hyb_analyze) and the Python3 API (HybFile, FoldRecord, HybFoldIter, hybkit.analysis, hybkit.plot) used for QC, filtering, and pooled replicate analysis) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime environment (Python3.8+) for the Hybkit API-based pooling and analysis scripts)
- **matplotlib** (Dependency used by hybkit.plot to render grouped target-analysis plots)
- **BioPython** (Hybkit dependency (>= 1.79) supporting sequence record handling in the API)

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- hyb_check completes over all replicate hyb and fold files without reporting data-integrity errors.
- Filtered output no longer contains ribosomal- or mitochondrial-RNA hybrid records (segment-type filter applied).
- Grouped target-analysis CSV tables are produced and readable, and summary counts match values reported in the Hybkit documentation example run on the GSE101978 qCLASH dataset.
- PNG plots from hybkit.plot are generated for the pooled replicates alongside the CSV outputs.
- Each record evaluated has assigned segment types and miRNA annotations (eval_types() and eval_mirna() were run before pooling).

## Limitations

- Requires Python3.8+ and dependency versions (matplotlib >= 3.7.1, BioPython >= 1.79, typing_extensions >= 4.8.0); older environments may fail.
- Grouped Target Analysis pools replicates and does not itself perform per-replicate statistics; single-replicate analysis should use the separate Target Analysis mode.
- Filtering by strings like 'any_seg_contains kshv' requires segment identifiers containing that string; hybrids lacking the marker are removed and could be false negatives.
- Duplicate hybrid filtration (by HybRecord.id) is an optional filter step and must be explicitly enabled via hyb_filter options.

## Evidence

- [methods] Grouped Target Analysis definition: [evidence span withheld — anchor status: not_found]
- [methods] Filtering artifactual hybrids: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [methods] QC step: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [methods] Dataset source: "data acquired from the NCBI Gene Expression Omnibus (GEO) accession `GSE101978"
- [readme] Example analyses in both CLI and API: "Example analyses for publicly available qCLASH hybrid sequence data implemented in each of the command-line scripts and hybkit Python API."
- [readme] Filter example command: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [readme] API record evaluation: "# Analyze each record to assign segment types
hyb_record.eval_types()"
