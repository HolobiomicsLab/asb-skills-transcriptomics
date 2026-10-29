---
name: hyb-format-flags-annotation-parsing
description: Use when input is a hyb file (or hyb + vienna/CT pair) from CLASH/qCLASH-style
  miRNA/target ligation experiments whose records must be iterated, flagged-parsed,
  and type-annotated before filtering or analysis.
when_to_use_negative:
- Input reads are raw FASTQ sequences or another chimeric-read format that has not
  been processed into hyb format by the Hyb software package
- The goal is statistical analysis or plotting of already-parsed hyb data rather than
  parsing/annotation itself (use hyb_analyze-type analyses instead)
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0097
tools:
- name: Hybkit
  role: Provides the HybFile iterator and HybRecord class used to parse hyb records,
    read the column-16 flags string, and run eval_types/eval_mirna annotation
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API (Python3.8+ required) used to script record iteration
    and annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:f0e91d510d9e5ab1b03539af84247b2b53008ac5734e22fb3bbe9faba93b3f6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python3
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# hyb-format-flags-annotation-parsing

## Summary

Parses hyb-format chimeric (hybrid) RNA records — 15–16 tab-separated columns including the optional column-16 flags string (e.g. seg1_type, miRNA_seg) — into hybkit HybRecord objects whose segment types and miRNA annotations can be evaluated and tested.

## When to use

Use when input is a hyb file (or hyb + vienna/CT pair) from CLASH/qCLASH-style miRNA/target ligation experiments whose records must be iterated, flagged-parsed, and type-annotated before filtering or analysis.

## When NOT to use

- Input reads are raw FASTQ sequences or another chimeric-read format that has not been processed into hyb format by the Hyb software package
- The goal is statistical analysis or plotting of already-parsed hyb data rather than parsing/annotation itself (use hyb_analyze-type analyses instead)

## Inputs

- hyb-format file (.hyb) of chimeric RNA hybrid records with 15–16 tab-separated columns
- optional associated vienna or CT fold file paired to the hyb records

## Outputs

- HybRecord objects with parsed id, sequence, energy, segment mapping fields, and flags-string annotations
- evaluated segment-type and miRNA annotations on each record (via eval_types/eval_mirna)
- hyb-format output files containing the parsed/annotated records

## How to apply

Open the .hyb file with the hybkit HybFile iterator and return each tab-separated line as a HybRecord object; hyb records carry id, sequence, energy, segment mapping fields, and the optional column-16 flags string with entries such as seg1_type and miRNA_seg. Call hyb_record.eval_types() on each record to assign segment types from the parsed annotations (and eval_mirna for miRNA assignments), then interrogate records with property tests such as has_prop('any_seg_type_contains', 'lncRNA') or membership checks like seg1_type == 'miRNA'. When a paired vienna/CT fold file exists, iterate with the HybFoldIter so fold records stay synchronized with hyb records. Rationale: the hyb format is defined by the Hyb software package for CLASH/qCLASH data, and hybkit's documented API (HybRecord, FoldRecord, HybFile, ViennaFile, CTFile, HybFoldIter) is the supported way to access these fields; parsed records can then be written back in hyb format or passed to hyb_filter/hyb_analyze.

## Related tools

- **Hybkit** (Provides the HybFile iterator and HybRecord class used to parse hyb records, read the column-16 flags string, and run eval_types/eval_mirna annotation) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API (Python3.8+ required) used to script record iteration and annotation)

## Examples

```
with hybkit.HybFile.open(in_file, 'r') as hyb_file:
    for hyb_record in hyb_file:
        hyb_record.eval_types()
```

## Evaluation signals

- Number of HybRecord objects yielded by the HybFile iterator equals the number of lines (records) in the input .hyb file
- Flags string from column 16 is correctly split into named entries (e.g. seg1_type, miRNA_seg) accessible via record property tests
- After eval_types(), records lacking a miRNA or having unassigned segment types are identifiable, e.g. has_prop('any_seg_type_contains', 'lncRNA') returns consistent results
- Round-trip check: records written back out parse again as valid hyb records with identical HybRecord.id and sequence fields
- Hybkit unit tests pass (pip install pytest; pytest) and toolkit scripts pass auto_tests/auto_test.sh

## Limitations

- Requires Python3.8+ with matplotlib >= 3.7.1, BioPython >= 1.79, and typing_extensions >= 4.8.0
- The flags string (column 16) is optional; records without it provide fewer annotation properties to test
- Segment-type and miRNA assignment depends on eval_types/eval_mirna custom criteria; unannotated or unrecognized reference identifiers may not receive a type
- Duplicate hybrid filtration options key on HybRecord.id, so id collisions in the source file affect deduplication

## Evidence

- [abstract] Here, we introduce Hybkit, a Python3 API, and command-line toolkit for analysis of hybrid sequence data in the "hyb" file format: "Here, we introduce Hybkit, a Python3 API, and command-line toolkit for analysis of hybrid sequence data in the "hyb" file format"
- [readme] Primary classes: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [readme] Record analysis example: "# Analyze each record to assign segment types
                hyb_record.eval_types()"
- [methods] Toolkit quality checking: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] Fold pairing support: "Hybkit provides a Python3 module with a documented API for interacting with
    records in hyb files and associated vienna or CT files."
