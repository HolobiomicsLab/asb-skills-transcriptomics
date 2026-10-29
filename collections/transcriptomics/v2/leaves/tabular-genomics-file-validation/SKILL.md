---
name: tabular-genomics-file-validation
description: Use when input consists of hyb-format hybrid-RNA files (optionally with
  paired '.vienna' or CT fold files) that must be error-checked for malformed columns,
  bad flags/header lines, or hyb-vs-Vienna sequence mismatches before running any
  type, miRNA, target, or fold analysis.
when_to_use_negative:
- Input is not hyb-format hybrid data (e.g. plain FASTQ/FASTA or an already-annotated
  feature table) — Hybkit's parsers expect the Hyb-defined 15/16-column hyb structure
- The Vienna/CT fold file does not correspond to the hyb file (different record set
  or ordering) — lockstep HybFoldIter comparison requires matched, ordered pairs
- Analysis goal is quantification or plotting (type/miRNA, target, fold analysis)
  rather than integrity checking — use hyb_eval or hyb_analyze after validation
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Hybkit
  role: Provides the hyb_check command-line utility and the HybFile, ViennaFile, CTFile,
    HybRecord, FoldRecord, and HybFoldIter API objects used to parse and cross-check
    paired hyb and fold records
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the Hybkit API (Python3.8+ required) when driving validation programmatically
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:c31fd8b42ab87a84f53ffdf218eab61e5d89a01631ff22ecf2a6699015ff04db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python3
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tabular-genomics-file-validation

## Summary

Validate tab-delimited chimeric-RNA hybrid ('hyb') files and their paired Vienna secondary-structure files for format errors and cross-file sequence mismatches before downstream analysis, using Hybkit's hyb_check/HybFoldIter mechanism.

## When to use

Use when input consists of hyb-format hybrid-RNA files (optionally with paired '.vienna' or CT fold files) that must be error-checked for malformed columns, bad flags/header lines, or hyb-vs-Vienna sequence mismatches before running any type, miRNA, target, or fold analysis.

## When NOT to use

- Input is not hyb-format hybrid data (e.g. plain FASTQ/FASTA or an already-annotated feature table) — Hybkit's parsers expect the Hyb-defined 15/16-column hyb structure
- The Vienna/CT fold file does not correspond to the hyb file (different record set or ordering) — lockstep HybFoldIter comparison requires matched, ordered pairs
- Analysis goal is quantification or plotting (type/miRNA, target, fold analysis) rather than integrity checking — use hyb_eval or hyb_analyze after validation

## Inputs

- hyb-format file of chimeric (hybrid) RNA records (.hyb), 15-or-16 tab-delimited columns per record
- Paired Vienna-format secondary-structure file (.vienna) or CT file with dot-bracket fold records matching the hyb records line-for-line

## Outputs

- Validation report (e.g. hyb_check_report.txt) listing per-record format errors and hyb/Vienna sequence mismatches with summary counts
- Zero-error pass signal for clean inputs, with all records parsed and lockstep-checked

## How to apply

Open the hyb file with hybkit.HybFile so each line is parsed as a HybRecord, checking the 15-or-16 tab-delimited column structure (id, seq, energy, seg1/seg2 mapping fields, optional flags) and that flags and header/comment lines are allowed. Open the corresponding Vienna file with hybkit.ViennaFile so each line is parsed as a FoldRecord containing the dot-bracket secondary structure. Iterate both files in lockstep with hybkit.HybFoldIter, comparing record identifiers and read sequences between paired hyb and fold entries to detect sequence mismatches. Accumulate all format errors (malformed columns, missing-data violations, disallowed header/comment lines) and hyb/fold sequence mismatches into a single validation report written to hyb_check_report.txt with per-record details and summary counts. This step mirrors Hybkit's documented quality-control convention: every analysis begins by checking data integrity (hyb_check) before downstream hyb_eval, hyb_filter, or hyb_analyze steps. A clean input must yield zero reported errors with all records passing both the parse and mismatch checks.

## Related tools

- **Hybkit** (Provides the hyb_check command-line utility and the HybFile, ViennaFile, CTFile, HybRecord, FoldRecord, and HybFoldIter API objects used to parse and cross-check paired hyb and fold records) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime for the Hybkit API (Python3.8+ required) when driving validation programmatically)

## Examples

```
hyb_check -i my_hyb_file.hyb -f my_hyb_file.vienna
```

## Evaluation signals

- Validation report file (e.g. hyb_check_report.txt) is produced, containing per-record error details and summary counts
- A clean input yields zero reported errors, with every record passing both the column/flag parse checks and the hyb/fold sequence-mismatch comparison
- Every hyb record parsed has the expected 15-or-16 tab-delimited columns (id, seq, energy, seg1/seg2 fields, optional flags) and only allowed flags
- Paired hyb and Vienna records align in identifier and read sequence — no HybFoldIter mismatch reported
- Downstream Hybkit utilities (hyb_eval, hyb_filter, hyb_analyze) consume the checked file without triggering format errors

## Limitations

- Validation is read-time and paired-file based: hyb and fold records must be provided in corresponding order, so a missing or reordered Vienna record can surface as a mismatch rather than being silently skipped
- Hybkit targets hyb-format data defined with the Hyb software package (from CLASH/qCLASH proximity-ligation experiments); other chimeric-RNA formats are out of scope
- The report captures format errors and hyb/Vienna sequence mismatches; it does not judge biological quality (e.g. artifactual ribosomal or mitochondrial hybrids, which are handled later by hyb_filter)

## Evidence

- [methods] Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`): "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] hyb_check parses hyb and fold files for errors: "hyb_check                           Parse hyb (and fold) files and check for errors"
- [readme] Hybkit API record and iterator classes: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes"
- [readme] Hybkit scope is hyb-format chimeric RNA data: "a toolkit for analysis of hyb-format chimeric (hybrid) RNA sequence data defined with the Hyb software package"
