---
name: chimeric-rna-segment-type-annotation
description: Use when input is a hyb-format hybrid (chimeric RNA) file from miRNA/target
  ligation methods such as CLASH/qCLASH and records need seg1_type/seg2_type and miRNA_seg
  annotations before quantification or filtering.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) or a non-hyb chimeric read format; hybkit operates
  on the hyb format defined by the Hyb software package
- The goal is filtering records rather than annotating them; use hyb_filter (e.g.
  ribosomal/mitochondrial removal or any_seg_contains) instead
- Records already carry seg1_type/seg2_type/miRNA_seg flags from a prior hyb_eval
  run and only re-annotation of the same criteria would duplicate work
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3511
tools:
- name: hybkit
  role: Python3 API providing HybFile iterator and HybRecord.eval_types/eval_mirna
    methods that assign segment types and miRNA_seg flags per record
  repo: github.com/RenneLab/hybkit
- name: hyb_eval
  role: Command-line utility that evaluates hyb (and fold) records to identify/assign
    segment types and miRNAs using custom criteria
  repo: github.com/RenneLab/hybkit
- name: hyb_check
  role: Quality-control step verifying the hyb file parses cleanly before/after annotation
  repo: github.com/RenneLab/hybkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:9f06932af0dddc457fac4b631d1eb89f689040ee133c543c7090dec9a447bd06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - hyb_check
  - hyb_eval
  - hybkit
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# chimeric-rna-segment-type-annotation

## Summary

Annotates each record in a hyb-format chimeric RNA file with segment types (e.g. 'miRNA', 'mRNA') and a miRNA_seg flag using hybkit's identifier-based evaluation, appending the results to the record's flags (column 16) so downstream miRNA/target analyses can operate.

## When to use

Use when input is a hyb-format hybrid (chimeric RNA) file from miRNA/target ligation methods such as CLASH/qCLASH and records need seg1_type/seg2_type and miRNA_seg annotations before quantification or filtering.

## When NOT to use

- Input is raw sequencing data (FASTQ) or a non-hyb chimeric read format; hybkit operates on the hyb format defined by the Hyb software package
- The goal is filtering records rather than annotating them; use hyb_filter (e.g. ribosomal/mitochondrial removal or any_seg_contains) instead
- Records already carry seg1_type/seg2_type/miRNA_seg flags from a prior hyb_eval run and only re-annotation of the same criteria would duplicate work

## Inputs

- hyb-format hybrid sequence file (e.g. my_hyb_file.hyb) from CLASH/qCLASH experiments
- optional associated vienna/CT fold file handled by the same hybkit utilities
- custom segment-type criteria (identifier-based mapping of segment reference names to types, e.g. MirBase miRNA identifiers like MIMAT0000078_MirBase_miR-23a_microRNA)

## Outputs

- annotated hyb file where every record carries seg1_type, seg2_type, and miRNA_seg flags in the column-16 flags list
- per-record miRNA_seg values restricted to 'N', '5p', '3p', 'B', or 'U'

## How to apply

Read the input .hyb file with hybkit's HybFile iterator, which yields each record as a HybRecord object. For each record call eval_types(), which uses a user-selectable, identifier-based method to assign segment types (e.g. 'miRNA' or 'mRNA') from the segment reference names. Then call eval_mirna() to set the miRNA_seg flag to 'N', '5p', '3p', 'B', or 'U' indicating whether and which segment is a miRNA. Append seg1_type, seg2_type, and miRNA_seg entries to the record's flags column (column 16, feature=value pairs) and write the annotated records with the hyb_eval command-line utility or the Python API. The same evaluation step underlies the hyb_analyze Type/miRNA analysis, so annotated flags enable subsequent type and miRNA quantification.

## Related tools

- **hybkit** (Python3 API providing HybFile iterator and HybRecord.eval_types/eval_mirna methods that assign segment types and miRNA_seg flags per record) — github.com/RenneLab/hybkit
- **hyb_eval** (Command-line utility that evaluates hyb (and fold) records to identify/assign segment types and miRNAs using custom criteria) — github.com/RenneLab/hybkit
- **hyb_check** (Quality-control step verifying the hyb file parses cleanly before/after annotation) — github.com/RenneLab/hybkit

## Examples

```
hyb_eval -i my_hyb_file.hyb -o annotated_records.hyb
```

## Evaluation signals

- hyb_check parses the annotated output file without errors
- every output record's column-16 flags list contains seg1_type, seg2_type, and miRNA_seg entries
- all miRNA_seg values fall within the allowed set {'N', '5p', '3p', 'B', 'U'}
- segment types assigned by eval_types are consistent with the identifier-based criteria (e.g. MirBase-matching reference names yield 'miRNA')
- record count and HybRecord.id values in output match the input file (annotation, not filtering, was applied)

## Limitations

- Segment typing is identifier-based and user-selectable, so accuracy depends on the supplied criteria/reference names; uncharacterized identifiers may not map to a type
- miRNA_seg is only meaningful for hybrids containing miRNA segments; other segment combinations yield non-miRNA flags
- Type/miRNA evaluation is a prerequisite for the miRNA-specific analyses (Type/miRNA, Target, Fold analysis) and cannot be skipped if those follow

## Evidence

- [readme] hyb_eval evaluates records: "hyb_eval                            Evaluate hyb (and fold) records to identify / assign
                                            segment types and miRNAs using custom criteria"
- [readme] eval_types API usage: "# Analyze each record to assign segment types
                hyb_record.eval_types()"
- [methods] renamed analysis methods: "Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively"
- [abstract] customizable evaluation finding: "analysis of hybrid sequence data in the "hyb" file format to enable customizable evaluation and annotation of hybrid characteristics"
- [methods] hyb_check QC: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
