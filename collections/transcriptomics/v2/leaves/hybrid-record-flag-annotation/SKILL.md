---
name: hybrid-record-flag-annotation
description: Use when a .hyb file from CLASH/qCLASH miRNA-target ligation experiments
  contains unannotated records and you need seg1_type, seg2_type, and miRNA_seg flags
  added to each record's flags column before type/miRNA, target, or fold analysis.
when_to_use_negative:
- The input is not in the hyb file format defined with the Hyb software package (e.g.,
  raw FASTQ or BAM) — convert or use a different skill
- Records already carry seg1_type/seg2_type/miRNA_seg flags and the goal is only subsetting
  — use hyb_filter instead of re-annotation
- The goal is plotting or statistics over already-annotated records (type/miRNA, target,
  grouped target, or fold analysis) — use hyb_analyze rather than re-running annotation
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3523
tools:
- name: hybkit
  role: Python3 API supplying HybFile/HybRecord and the eval_types/eval_mirna methods
    used to assign segment types and miRNA flags per record
  repo: github.com/RenneLab/hybkit
- name: hyb_eval
  role: Command-line utility that performs the same record evaluation, identifying/assigning
    segment types and miRNAs using custom criteria and writing annotated hyb records
  repo: github.com/RenneLab/hybkit
- name: hyb_check
  role: Quality-control step run before/after annotation to verify the hyb (and fold)
    file parses without errors
  repo: github.com/RenneLab/hybkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:1ab14defd557c17ba15fb464fe5355061c3d3342fd0f58dcaabfd1e279268020
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
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# hybrid-record-flag-annotation

## Summary

Annotates each record in a hyb-format chimeric RNA file with segment-type and miRNA flags using hybkit's identifier-based evaluation (HybRecord.eval_types and eval_mirna, exposed via the hyb_eval CLI), enabling downstream miRNA/target analyses.

## When to use

Use when a .hyb file from CLASH/qCLASH miRNA-target ligation experiments contains unannotated records and you need seg1_type, seg2_type, and miRNA_seg flags added to each record's flags column before type/miRNA, target, or fold analysis.

## When NOT to use

- The input is not in the hyb file format defined with the Hyb software package (e.g., raw FASTQ or BAM) — convert or use a different skill
- Records already carry seg1_type/seg2_type/miRNA_seg flags and the goal is only subsetting — use hyb_filter instead of re-annotation
- The goal is plotting or statistics over already-annotated records (type/miRNA, target, grouped target, or fold analysis) — use hyb_analyze rather than re-running annotation

## Inputs

- hyb-format file (.hyb) of chimeric/hybrid RNA records from miRNA/target ligation methods (CLASH or qCLASH)
- optionally an associated vienna or CT fold file evaluated alongside the hyb records

## Outputs

- Annotated .hyb file in which every record's column-16 flags list carries seg1_type, seg2_type, and miRNA_seg feature=value entries (miRNA_seg in {'N','5p','3p','B','U'})
- HybRecord objects with attributes set for downstream analyses (hyb_analyze type/miRNA, target, grouped target, or fold analyses)

## How to apply

Load the .hyb input with hybkit's HybFile iterator so each line is yielded as a HybRecord object. For each record call eval_types(), which assigns segment types (e.g., 'miRNA' or 'mRNA') from the segment reference identifiers using user-selectable criteria, then call eval_mirna() to set the miRNA_seg flag ('N', '5p', '3p', 'B', or 'U') indicating which segment, if any, is a mature miRNA (e.g., a MirBase entry such as MIMAT0000078_MirBase_miR-23a). The resulting seg1_type, seg2_type, and miRNA_seg entries are appended to the record's flags string (column 16, feature=value pairs), matching what the hyb_eval command-line utility does. Before annotation, run hyb_check on the file to confirm data integrity. The same logic is reachable either through the Python API or by running the hyb_eval script, which evaluates hyb (and associated fold/vienna) records 'using custom criteria'.

## Related tools

- **hybkit** (Python3 API supplying HybFile/HybRecord and the eval_types/eval_mirna methods used to assign segment types and miRNA flags per record) — github.com/RenneLab/hybkit
- **hyb_eval** (Command-line utility that performs the same record evaluation, identifying/assigning segment types and miRNAs using custom criteria and writing annotated hyb records) — github.com/RenneLab/hybkit
- **hyb_check** (Quality-control step run before/after annotation to verify the hyb (and fold) file parses without errors) — github.com/RenneLab/hybkit

## Examples

```
import hybkit; hyb_record.eval_types(); hyb_record.eval_mirna()
```

## Evaluation signals

- The annotated output file parses cleanly as a hyb file when re-run through hyb_check
- Every record in the output carries seg1_type, seg2_type, and miRNA_seg flags in its column-16 feature=value annotation list
- All miRNA_seg values fall within the allowed set {'N', '5p', '3p', 'B', 'U'}
- Segment types are consistent with the reference identifiers (e.g., MirBase-style identifiers such as MIMAT0000078_MirBase_miR-23a yield 'miRNA'; other references yield 'mRNA' or other types)
- Record count in the annotated output equals the record count in the validated input file (annotation must not drop or duplicate records)

## Limitations

- Type assignment is identifier-based: records whose segment reference names do not match known type criteria (e.g., non-miRNA, non-standard identifiers) may receive unexpected or unassigned types, so criteria must be user-selected to match the dataset
- The annotation depends on hybkit's built-in criteria; miRNA_seg reflects which segment is a miRNA per the current identifier mapping (e.g., MirBase), and evolving identifier conventions may require custom criteria
- Annotated records alone do not validate miRNA/target biology; flag values are derived purely from reference names, not from expression or folding evidence

## Evidence

- [readme] Evaluate hyb (and fold) records to identify / assign segment types and miRNAs using custom criteria: "Evaluate hyb (and fold) records to identify / assign segment types and miRNAs using custom criteria"
- [methods] Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively: "Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively"
- [methods] Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`): "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] Return each line in a hyb file as a HybRecord object: "Return each line in a hyb file as a HybRecord object"
- [readme] Record attributes can be analyzed, set, and evaluated using included class methods.: "Record attributes can be analyzed, set, and evaluated using included class methods."
