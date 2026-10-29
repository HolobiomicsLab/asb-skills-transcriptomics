---
name: hyb-fold-sequence-mismatch-checking
description: Use when a qCLASH/CLASH hyb file has a paired Vienna (or CT) secondary-structure
  file and you need to validate record identity, merge fold predictions into hybrids,
  or detect hyb/fold sequence mismatches before analysis.
when_to_use_negative:
- The hyb file has no corresponding Vienna or CT fold file — HybFoldIter pairing requires
  a matching FoldRecord per HybRecord.
- Records are not ordered/identifiable across the two files — pairing relies on matching
  record identifiers and lockstep iteration.
- The goal is only duplicate removal or segment-type filtering, which hyb_filter (--filter
  any_seg_contains, duplicate filtration by HybRecord.id) handles without fold merging.
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0099
tools:
- name: Hybkit
  role: Provides HybFile, ViennaFile, HybFoldIter, HybRecord and FoldRecord classes
    for lockstep iteration, fold-attribute merging, and default mismatch checking
    between hyb and Vienna files
  repo: http://www.github.com/RenneLab/hybkit
- name: hyb_check
  role: Command-line utility that wraps this concurrent hyb/Vienna iteration to parse
    and check records for errors before analysis
  repo: http://www.github.com/RenneLab/hybkit
- name: Python 3.8
  role: Runtime for the hybkit API (Python3.8+ required)
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:d22b38e75d0500b63100b5b0f57385fe1d631d5809571926dbcf24d83f01a769
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python
  - Python 3.8
  - Python3
  - hyb_check
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# hyb-fold-sequence-mismatch-checking

## Summary

Concurrent iteration over paired hyb and Vienna dot-bracket files with hybkit's HybFoldIter class to merge FoldRecord data (predicted fold pattern and Gibbs free energy) into HybRecords while detecting sequence mismatches and format errors between the two files.

## When to use

Use when a qCLASH/CLASH hyb file has a paired Vienna (or CT) secondary-structure file and you need to validate record identity, merge fold predictions into hybrids, or detect hyb/fold sequence mismatches before analysis.

## When NOT to use

- The hyb file has no corresponding Vienna or CT fold file — HybFoldIter pairing requires a matching FoldRecord per HybRecord.
- Records are not ordered/identifiable across the two files — pairing relies on matching record identifiers and lockstep iteration.
- The goal is only duplicate removal or segment-type filtering, which hyb_filter (--filter any_seg_contains, duplicate filtration by HybRecord.id) handles without fold merging.

## Inputs

- hyb-format hybrid RNA file (15-or-16 tab-delimited columns: id, seq, energy, seg1/seg2 mapping fields, optional flags), e.g. from GEO GSE101978 qCLASH data
- Paired Vienna dot-bracket secondary-structure file (one FoldRecord per hyb record, sharing record identifiers)

## Outputs

- hyb_fold_merged.hyb — tab-delimited hyb file with fold pattern and Gibbs free-energy merged into each record plus mismatch flags
- Validation/check report listing per-record hyb/fold sequence mismatches and format errors (e.g. hyb_check_report.txt)

## How to apply

Open the hyb file with hybkit.HybFile and the paired Vienna dot-bracket file with hybkit.ViennaFile, then iterate both in lockstep with hybkit.HybFoldIter, which pairs each HybRecord with its matching FoldRecord by record identifier. For each iteration step, merge the FoldRecord's dot-bracket folding pattern and predicted Gibbs free-energy value into the corresponding HybRecord. Mismatch checking is enabled by default: the read sequence in each HybRecord is compared against the sequence in its FoldRecord, and records are flagged where the hyb/fold sequences mismatch; this can be disabled by setting mismatches to "-1". HybFile parsing validates the 15-or-16 tab-delimited column structure (id, seq, energy, seg1/seg2 mapping fields, optional flags), collecting format errors and mismatches into a validation report. Write the fold-annotated records with mismatch flags to a tab-delimited hyb file (e.g. hyb_fold_merged.hyb) for downstream hyb_analyze-style analyses such as Fold Analysis.

## Related tools

- **Hybkit** (Provides HybFile, ViennaFile, HybFoldIter, HybRecord and FoldRecord classes for lockstep iteration, fold-attribute merging, and default mismatch checking between hyb and Vienna files) — http://www.github.com/RenneLab/hybkit
- **hyb_check** (Command-line utility that wraps this concurrent hyb/Vienna iteration to parse and check records for errors before analysis) — http://www.github.com/RenneLab/hybkit
- **Python 3.8** (Runtime for the hybkit API (Python3.8+ required))

## Examples

```
with hybkit.HybFile.open(in_file, 'r') as hyb_file:  # iterate with hybkit.HybFoldIter against the paired Vienna file to merge fold data and flag hyb/fold sequence mismatches
```

## Evaluation signals

- hyb_fold_merged.hyb exists and parses without error under hybkit's record-parsing checks (hyb_check-style integrity checking)
- Each output HybRecord carries fold information (dot-bracket pattern and Gibbs free energy) consistent with its paired FoldRecord
- Mismatch flags in the output match records where the hyb read sequence differs from the FoldRecord sequence
- A clean input yields zero reported errors with all records passing parse and mismatch checks
- Record identifiers pair one-to-one between the hyb and Vienna files across the iteration (no skipped or unmatched records)

## Limitations

- Mismatch checking must be explicitly disabled (set mismatches to "-1") if sequences are expected to differ; by default any hyb/fold sequence difference is flagged.
- Format errors (malformed columns, disallowed header/comment lines, missing-data violations) abort or surface at read time, so pre-cleaned input may be required.
- Requires hybkit (Python3.8+) with its dependencies; the hyb format is that defined by the Hyb software package, so other hybrid-file dialects are not covered.

## Evidence

- [methods] Hybkit provides a HybFoldIter class for concurrent iteration over hyb and Vienna files: [evidence span withheld — anchor status: not_found]
- [methods] mismatch checking enabled by default; it can be disabled by setting mismatches to "-1": [evidence span withheld — anchor status: not_found]
- [readme] hyb_check Parse hyb (and fold) files and check for errors: "hyb_check                           Parse hyb (and fold) files and check for errors"
- [readme] file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter): "a class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [methods] Each analysis performs quality control steps on the data by checking data integrity: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
