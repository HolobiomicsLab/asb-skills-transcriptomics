---
name: hyb-format-record-parsing
description: Use when input is hyb-format chimeric RNA data (e.g., CLASH/qCLASH output)
  that must be read into records, integrity-checked, or merged lockstep with a paired
  Vienna fold file before type/miRNA/target analysis.
when_to_use_negative:
- Input is raw sequencing reads (e.g., FASTQ) that have not yet been processed by
  the Hyb software package into the hyb format defined by Travis et al. (Methods 2014)
- Data are already a count table or CSV/plot output from a downstream Type/miRNA or
  Target Analysis, not line-level hybrid records
- The goal is de novo RNA secondary-structure prediction rather than reading existing
  predicted fold files into FoldRecords
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3168
tools:
- name: Hybkit
  role: Provides HybFile/ViennaFile/CTFile iterators, HybRecord and FoldRecord classes,
    HybFoldIter lockstep merging, and the hyb_check/hyb_eval/hyb_filter command-line
    utilities used to parse and validate records
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API (Python3.8+), used to script HybFile iteration
    and per-record evaluation
  repo: http://www.github.com/RenneLab/hybkit
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:8282883925e1113c7fc0a0aef21244fb3c25099592997f1baaa4194db6d89e89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python
  - Python3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hyb-format-record-parsing

## Summary

Parses hyb-format chimeric (hybrid) RNA files — and optionally their paired Vienna/CT secondary-structure files — into HybRecord and FoldRecord objects, validating the 15-or-16 tab-delimited column structure and hyb/Vienna sequence consistency before analysis.

## When to use

Use when input is hyb-format chimeric RNA data (e.g., CLASH/qCLASH output) that must be read into records, integrity-checked, or merged lockstep with a paired Vienna fold file before type/miRNA/target analysis.

## When NOT to use

- Input is raw sequencing reads (e.g., FASTQ) that have not yet been processed by the Hyb software package into the hyb format defined by Travis et al. (Methods 2014)
- Data are already a count table or CSV/plot output from a downstream Type/miRNA or Target Analysis, not line-level hybrid records
- The goal is de novo RNA secondary-structure prediction rather than reading existing predicted fold files into FoldRecords

## Inputs

- hyb-format chimeric RNA file (.hyb) from microRNA/target ligation experiments (e.g., qCLASH data such as GEO GSE101978)
- Paired Vienna-format secondary-structure file (dot-bracket), or a CT file, associated with the hyb file

## Outputs

- HybRecord objects (one per hyb-file line) with validated column structure and flags
- FoldRecord objects carrying dot-bracket secondary structure from Vienna/CT records
- hyb_check_report.txt validation report with per-record error details and summary counts
- Annotated hyb file (e.g., annotated_records.hyb) with seg1_type/seg2_type/miRNA_seg flags in column 16

## How to apply

Open the input .hyb file with hybkit's HybFile iterator, which returns each line as a HybRecord, and validate that it has the 15-or-16 tab-delimited column structure (id, seq, energy, seg1/seg2 mapping fields, optional flags) with only allowed header/comment lines and flags. If a companion secondary-structure file exists, parse it with ViennaFile (or CTFile) so each line becomes a FoldRecord holding the dot-bracket structure. Iterate the hyb and fold records in lockstep with HybFoldIter, comparing record identifiers and read sequences between paired entries to detect hyb/Vienna sequence mismatches at read time. Run the checks via hyb_check (or the API equivalent), collecting format errors and mismatches into a single report written to hyb_check_report.txt with per-record details and summary counts. After validation, annotate records with HybRecord.eval_types() and eval_mirna() so seg1_type, seg2_type, and miRNA_seg entries are appended to the column-16 flags string as feature=value pairs.

## Related tools

- **Hybkit** (Provides HybFile/ViennaFile/CTFile iterators, HybRecord and FoldRecord classes, HybFoldIter lockstep merging, and the hyb_check/hyb_eval/hyb_filter command-line utilities used to parse and validate records) — github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API (Python3.8+), used to script HybFile iteration and per-record evaluation) — http://www.github.com/RenneLab/hybkit

## Examples

```
with hybkit.HybFile.open(in_file, 'r') as hyb_file: for hyb_record in hyb_file: hyb_record.eval_types()
```

## Evaluation signals

- A clean input yields a hyb_check_report.txt with zero reported errors and all records passing the parse and hyb/fold mismatch checks
- The output file parses cleanly as a hyb file when re-run through hyb_check
- Every record written after eval_types/eval_mirna carries seg1_type, seg2_type, and miRNA_seg flags in its column-16 annotation list (miRNA_seg restricted to N/5p/3p/B/U)
- Lockstep HybFoldIter comparison flags any identifier or read-sequence mismatch between paired hyb and Vienna entries
- Downstream count tables (e.g., CSV from Type/miRNA analysis) are internally consistent with the number of validated records parsed

## Limitations

- Fold-integrated checking requires a matching Vienna (or CT) file; hyb/Vienna sequence mismatches are only detected when records are iterated in lockstep via HybFoldIter
- The hyb format allows both 15- and 16-column variants, and only a constrained set of header/comment lines and flags is permitted, so non-conforming files fail validation
- Flag-based segment typing is identifier-dependent (e.g., seg names must match type_finder criteria such as MirBase identifiers like MIMAT0000078), so unrecognized reference names will not be classified

## Evidence

- [abstract] Here, we introduce Hybkit, a Python3 API, and command-line toolkit for analysis of hybrid sequence data in the "hyb" file format: "Here, we introduce Hybkit, a Python3 API, and command-line toolkit for analysis of hybrid sequence data in the "hyb" file format"
- [methods] Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`): "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes"
