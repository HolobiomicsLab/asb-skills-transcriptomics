---
name: identifier-based-sequence-classification
description: Use when hyb-format records from miRNA/target ligation experiments (e.g.
  CLASH/qCLASH) need segment-type classification from segment reference identifiers,
  or when miRNA_seg flags ('N','5p','3p','B','U') must be set for downstream analysis.
when_to_use_negative:
- Input is not in the hyb file format defined by the Hyb software package (e.g. raw
  FASTQ or unaligned chimera reads) — records must already be parsed into hyb records
- 'The goal is subsetting rather than typing: use hyb_filter (e.g. --filter any_seg_contains
  kshv) instead of eval-based classification'
- Segment reference identifiers do not conform to conventions the identifier-based
  criteria can resolve, so typing would rely on unsupported inference
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Hybkit
  role: Python API providing HybFile iterator and HybRecord class with eval_types()
    and eval_mirna() methods for identifier-based segment-type and miRNA annotation
  repo: http://www.github.com/RenneLab/hybkit
- name: hyb_eval
  role: Command-line utility that evaluates hyb (and fold) records to identify/assign
    segment types and miRNAs using custom criteria and writes flags to column 16
  repo: http://www.github.com/RenneLab/hybkit
- name: hyb_check
  role: Used to verify the annotated output parses cleanly as a hyb file after type
    annotation
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime environment (Python3.8+) for running the hybkit API and HybRecord
    methods
  repo: https://pypi.org/project/hybkit/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:9d4d6ce3a196840dcf3dec5821ae7e66301ab8e6443332960c3be4453bc4c7e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python
  - Python3
  - hyb_check
  - hyb_eval
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# identifier-based-sequence-classification

## Summary

Assigns segment types (e.g. 'miRNA', 'mRNA', 'lncRNA') and miRNA flags to hybrid RNA records in the 'hyb' file format using identifier-based criteria, as implemented by hybkit's HybRecord.eval_types and eval_mirna methods and the hyb_eval command-line utility.

## When to use

Use when hyb-format records from miRNA/target ligation experiments (e.g. CLASH/qCLASH) need segment-type classification from segment reference identifiers, or when miRNA_seg flags ('N','5p','3p','B','U') must be set for downstream analysis.

## When NOT to use

- Input is not in the hyb file format defined by the Hyb software package (e.g. raw FASTQ or unaligned chimera reads) — records must already be parsed into hyb records
- The goal is subsetting rather than typing: use hyb_filter (e.g. --filter any_seg_contains kshv) instead of eval-based classification
- Segment reference identifiers do not conform to conventions the identifier-based criteria can resolve, so typing would rely on unsupported inference

## Inputs

- hyb-format hybrid RNA sequence file (.hyb) from CLASH/qCLASH or other miRNA/target ligation methods
- Optional associated vienna/CT fold file paired with the hyb file
- Identifier-based typing criteria / reference identifier conventions (e.g. MirBase miRNA identifiers)

## Outputs

- HybRecord objects with seg1_type and seg2_type assigned in memory (API path)
- Annotated .hyb file where each record's flags column (column 16) carries seg1_type, seg2_type, and miRNA_seg feature=value entries
- Segment-type and miRNA-type counts (e.g. CSV table and pie chart from the type/miRNA analysis)

## How to apply

Load the input .hyb file with hybkit's HybFile iterator, which yields each line as a HybRecord object. For each record, run HybRecord.eval_types() to assign segment types to seg1 and seg2 from the segment reference names using user-selectable, identifier-based criteria. Then run eval_mirna (formerly mirna_analysis) to set the miRNA_seg flag — 'N', '5p', '3p', 'B', or 'U' — indicating whether no segment, the 5p segment, the 3p segment, both, or an unknown segment is a miRNA. With the hyb_eval command-line utility, the resulting seg1_type, seg2_type, and miRNA_seg annotations are appended to the record's flags column (column 16, feature=value pairs) of the annotated output .hyb file. The identifier-based method is customizable, so type definitions must be supplied or configured to match the naming conventions of the reference sequences (e.g. MirBase identifiers such as MIMAT0000078_MirBase_miR-23a_microRNA).

## Related tools

- **Hybkit** (Python API providing HybFile iterator and HybRecord class with eval_types() and eval_mirna() methods for identifier-based segment-type and miRNA annotation) — http://www.github.com/RenneLab/hybkit
- **hyb_eval** (Command-line utility that evaluates hyb (and fold) records to identify/assign segment types and miRNAs using custom criteria and writes flags to column 16) — http://www.github.com/RenneLab/hybkit
- **hyb_check** (Used to verify the annotated output parses cleanly as a hyb file after type annotation) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime environment (Python3.8+) for running the hybkit API and HybRecord methods) — https://pypi.org/project/hybkit/

## Examples

```
for hyb_record in hybkit.HybFile.open(in_file, 'r'): hyb_record.eval_types(); hyb_record.eval_mirna()
```

## Evaluation signals

- Annotated output file parses cleanly as a hyb file when re-checked with hyb_check
- Every output record carries seg1_type, seg2_type, and miRNA_seg flags in its column-16 feature=value annotation list
- Each miRNA_seg value is one of the allowed flags: 'N', '5p', '3p', 'B', or 'U'
- A type/miRNA analysis (hyb_analyze) on the annotated file returns non-empty sequence-type counts as a CSV table matching hybrid types present in the input
- Spot-check records: segment reference names matching miRNA identifiers (e.g. MirBase MIMAT entries) are typed 'miRNA' and flagged '5p' or '3p' rather than 'N'

## Limitations

- Classification is identifier-based and user-selectable/custom, so results depend entirely on the supplied criteria matching the reference identifier naming conventions; unrecognized identifiers yield unassigned or 'U'/'N' flags
- hyb_eval operates on hyb (and fold) records only — it does not perform QC or filtering; data integrity must be checked separately with hyb_check, and artifactual ribosomal-/mitochondrial-RNA hybrids removed via hyb_filter
- The API method names were renamed (type_analysis/mirna_analysis to eval_types/eval_mirna), so older scripts or versions of hybkit may use the previous method names

## Evidence

- [readme] hyb_eval: "Evaluate hyb (and fold) records to identify / assign segment types and miRNAs using custom criteria"
- [readme] eval_types: "Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively"
- [readme] hybkit API: "Hybkit provides a Python3 module with a documented API for interacting with records in hyb files and associated vienna or CT files."
- [abstract] hyb file format: "analysis of hybrid sequence data in the "hyb" file format to enable customizable evaluation and annotation of hybrid characteristics"
