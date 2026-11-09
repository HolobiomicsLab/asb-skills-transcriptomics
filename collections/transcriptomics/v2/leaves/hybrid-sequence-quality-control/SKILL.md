---
name: hybrid-sequence-quality-control
description: Use when input is a hyb-format chimeric RNA file (e.g. from CLASH or
  qCLASH) with an associated Vienna/CT fold file, and data integrity must be checked
  (malformed columns, disallowed lines, hyb/fold sequence mismatches) before type/miRNA,
  target, or fold analyses.
when_to_use_negative:
- Input is not hyb-format chimeric RNA data (e.g. raw FASTQ reads or a non-chimeric
  small-RNA library) — hybkit parses the 'hyb' file format defined with the Hyb software
  package, not raw sequencing output
- The hyb file has no corresponding Vienna/CT fold file and fold-based mismatch checking
  is not required — plain record parsing may suffice, but concurrent HybFoldIter validation
  cannot be applied
- Records are already validated and the goal is subsetting rather than integrity checking
  — use hyb_filter (e.g. --filter any_seg_contains) instead of re-running hyb_check
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0080
tools:
- name: Hybkit
  role: Provides the hyb_check command-line utility and the Python API classes (HybFile,
    ViennaFile, HybRecord, FoldRecord, HybFoldIter) that implement paired hyb/Vienna
    parsing and error/mismatch detection
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API (Python3.8+) used to open hyb and Vienna files
    and iterate paired records for validation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:c517554b32601d0148e3fe2216e382c178bfccf8970021ecdbaf2f36d7d1aed1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python3
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hybrid-sequence-quality-control

## Summary

Quality-control skill for chimeric (hybrid) RNA sequence data in the 'hyb' file format, using hybkit's hyb_check utility to parse hyb records and concurrently validate them against paired Vienna secondary-structure (fold) files before downstream analysis.

## When to use

Use when input is a hyb-format chimeric RNA file (e.g. from CLASH or qCLASH) with an associated Vienna/CT fold file, and data integrity must be checked (malformed columns, disallowed lines, hyb/fold sequence mismatches) before type/miRNA, target, or fold analyses.

## When NOT to use

- Input is not hyb-format chimeric RNA data (e.g. raw FASTQ reads or a non-chimeric small-RNA library) — hybkit parses the 'hyb' file format defined with the Hyb software package, not raw sequencing output
- The hyb file has no corresponding Vienna/CT fold file and fold-based mismatch checking is not required — plain record parsing may suffice, but concurrent HybFoldIter validation cannot be applied
- Records are already validated and the goal is subsetting rather than integrity checking — use hyb_filter (e.g. --filter any_seg_contains) instead of re-running hyb_check

## Inputs

- hyb-format hybrid RNA sequence file (.hyb) with 15-or-16 tab-delimited columns (id, seq, energy, seg1/seg2 mapping fields, optional flags)
- Paired Vienna-format secondary-structure file (dot-bracket fold records) for the same hybrids (or CT-format fold file)

## Outputs

- Validation report file (hyb_check_report.txt) listing per-record error details and summary counts
- Parsed, validated HybRecord and FoldRecord objects suitable for downstream hybkit analyses

## How to apply

Open the input hyb file with hybkit.HybFile and parse each line as a HybRecord, validating the 15-or-16 tab-delimited column structure (id, seq, energy, seg1/seg2 mapping fields, optional flags) and allowed flags. Open the corresponding Vienna file with hybkit.ViennaFile and parse each line as a FoldRecord containing the dot-bracket secondary structure. Iterate the hyb and fold records in lockstep with hybkit.HybFoldIter, comparing record identifiers and read sequences between the paired entries so that format errors and hyb/Vienna sequence mismatches are detected at read time. Collect all format errors (malformed columns, disallowed header/comment lines, missing-data violations) and mismatches into a single validation report and write it to hyb_check_report.txt. A clean input should yield zero reported errors with all records passing both the parse and mismatch checks; every downstream hybkit analysis performs this quality-control step via hyb_check before filtering or analysis.

## Related tools

- **Hybkit** (Provides the hyb_check command-line utility and the Python API classes (HybFile, ViennaFile, HybRecord, FoldRecord, HybFoldIter) that implement paired hyb/Vienna parsing and error/mismatch detection) — github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API (Python3.8+) used to open hyb and Vienna files and iterate paired records for validation)

## Examples

```
hyb_check -i my_hyb_file.hyb -f my_hyb_file.vienna
```

## Evaluation signals

- hyb_check_report.txt is produced and a clean input yields zero reported errors with all records passing the parse and mismatch checks
- Every record in the .hyb file parses with the expected 15-or-16 tab-delimited column structure (id, seq, energy, seg1/seg2 fields, optional flags) and only allowed flags
- For each position i, hyb record i and Vienna FoldRecord i share the same record identifier and read sequence (no hyb/fold sequence mismatches reported)
- Running the hybkit unit-test suite (pytest) and command-line auto_test.sh passes, confirming API and script behavior
- Downstream hybkit analyses (hyb_filter, hyb_analyze) accept the validated files without read-time format errors

## Limitations

- Validation depends on the paired file convention: sequence mismatches are only caught when a corresponding Vienna/CT fold file is supplied and records are iterated in lockstep
- hybkit requires Python3.8+ (with matplotlib >= 3.7.1, BioPython >= 1.79, typing_extensions >= 4.8.0); malformed header/comment lines or missing-data violations are treated as errors rather than auto-repaired
- hyb_check checks format integrity and hyb/fold consistency only — it does not perform biological filtering such as removal of artifactual ribosomal- and mitochondrial-RNA hybrids (that is hyb_filter's role)

## Evidence

- [methods] Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`): "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] hyb_check parses hyb (and fold) files and checks for errors: "hyb_check                           Parse hyb (and fold) files and check for errors"
- [other] HybFoldIter merges FoldRecord data into corresponding HybRecords and checks for potential errors and mismatches: [evidence span withheld — anchor status: not_found]
- [readme] These scripts are used on the command line with hyb (and associated "vienna" or "CT") files.: "These scripts are used on the command line with hyb (and associated "vienna" or "CT") files."
