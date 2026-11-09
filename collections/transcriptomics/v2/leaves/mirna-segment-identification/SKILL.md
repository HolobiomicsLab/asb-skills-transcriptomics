---
name: mirna-segment-identification
description: Use when working with hyb-format chimeric RNA records from CLASH/qCLASH
  (or similar miRNA/target ligation) experiments whose seg1_type, seg2_type, and miRNA_seg
  annotations are missing, and downstream steps (target analysis, fold analysis, plotting)
  require knowing which segment is a miRNA.
when_to_use_negative:
- The input is not hyb-format chimeric RNA data (e.g. raw FASTQ, mzML, or non-hybrid
  feature tables) — hybkit's records assume the Hyb-defined hyb file format
- Records already carry seg1_type, seg2_type, and miRNA_seg flags from a previous
  hyb_eval/eval_types run and re-annotation with the same criteria would be redundant
- Segment typing must come from genomic alignment/quantification rather than the record's
  reference identifiers — hybkit typing is identifier-based and cannot resolve ambiguous
  segment names beyond its custom criteria
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: hybkit
  role: Python API providing HybRecord.eval_types() and eval_mirna() for identifier-based
    segment-type and miRNA assignment on hyb records
  repo: https://github.com/RenneLab/hybkit
- name: hyb_eval
  role: Command-line utility that evaluates hyb (and fold) records to identify/assign
    segment types and miRNAs using custom criteria and appends them to record flags
  repo: https://github.com/RenneLab/hybkit
- name: hyb_check
  role: Quality-control step verifying the .hyb input parses cleanly with the 15-16
    tab-delimited columns before annotation
  repo: https://github.com/RenneLab/hybkit
- name: hyb_filter
  role: Removes artifactual ribosomal- and mitochondrial-RNA hybrids prior to type/miRNA
    evaluation
  repo: https://github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API workflow (Python3.8+ required)
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:6bbf19a21ff76a2775efb53155745af8320b7a9d75427cd66096460cb09105a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RenneLab/hybkit
  tools:
  - BioPython
  - Python
  - Python3
  - hyb_check
  - hyb_eval
  - hyb_filter
  - hybkit
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mirna-segment-identification

## Summary

Annotates each hybrid record in hyb-format qCLASH/CLASH data with segment types (e.g. miRNA, mRNA) and a miRNA_seg flag identifying which segment is the miRNA, using hybkit's identifier-based eval_types() and eval_mirna() methods or the hyb_eval command-line utility. This annotation is the prerequisite for all downstream hybkit miRNA, target, and fold analyses.

## When to use

Use when working with hyb-format chimeric RNA records from CLASH/qCLASH (or similar miRNA/target ligation) experiments whose seg1_type, seg2_type, and miRNA_seg annotations are missing, and downstream steps (target analysis, fold analysis, plotting) require knowing which segment is a miRNA.

## When NOT to use

- The input is not hyb-format chimeric RNA data (e.g. raw FASTQ, mzML, or non-hybrid feature tables) — hybkit's records assume the Hyb-defined hyb file format
- Records already carry seg1_type, seg2_type, and miRNA_seg flags from a previous hyb_eval/eval_types run and re-annotation with the same criteria would be redundant
- Segment typing must come from genomic alignment/quantification rather than the record's reference identifiers — hybkit typing is identifier-based and cannot resolve ambiguous segment names beyond its custom criteria

## Inputs

- hyb-format hybrid record file (.hyb) from CLASH/qCLASH, e.g. GEO GSE101978 replicates from Gay et al. 2018
- Optional paired Vienna or CT fold file when annotation precedes fold analysis

## Outputs

- Annotated hyb records carrying seg1_type, seg2_type, and miRNA_seg flags (e.g. miRNA_seg ∈ {N, 5p, 3p, B, U}) in the column-16 feature=value annotation list
- Annotated output file such as annotated_records.hyb, parseable by hyb_check

## How to apply

Load the input .hyb file with hybkit's HybFile iterator, yielding each line as a HybRecord object. For each record, call HybRecord.eval_types() to assign segment types from the segment reference identifiers using hybkit's type_finder criteria, then call eval_mirna() to set the miRNA_seg flag to one of 'N', '5p', '3p', 'B', or 'U' (N = no miRNA, 5p/3p = the miRNA occupies that segment, B = both segments, U = undetermined). Run the hyb_check-style integrity evaluation first and remove artifactual ribosomal- and mitochondrial-RNA hybrids with hyb_filter before annotation. Append the seg1_type, seg2_type, and miRNA_seg entries to the record's column-16 flags as feature=value pairs and write the annotated records out. Equivalent CLI path: hyb_eval evaluates hyb (and fold) records to assign segment types and miRNAs using custom criteria, so identifier-based typing is user-selectable rather than hard-coded.

## Related tools

- **hybkit** (Python API providing HybRecord.eval_types() and eval_mirna() for identifier-based segment-type and miRNA assignment on hyb records) — https://github.com/RenneLab/hybkit
- **hyb_eval** (Command-line utility that evaluates hyb (and fold) records to identify/assign segment types and miRNAs using custom criteria and appends them to record flags) — https://github.com/RenneLab/hybkit
- **hyb_check** (Quality-control step verifying the .hyb input parses cleanly with the 15-16 tab-delimited columns before annotation) — https://github.com/RenneLab/hybkit
- **hyb_filter** (Removes artifactual ribosomal- and mitochondrial-RNA hybrids prior to type/miRNA evaluation) — https://github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API workflow (Python3.8+ required))

## Examples

```
with hybkit.HybFile.open(in_file, 'r') as hyb_file:
    for hyb_record in hyb_file:
        hyb_record.eval_types()
        hyb_record.eval_mirna()
```

## Evaluation signals

- Output file parses cleanly as a hyb file via hyb_check (every record carries the 15-16 tab-delimited columns)
- Every annotated record carries seg1_type, seg2_type, and miRNA_seg flags in its column-16 feature=value annotation list
- miRNA_seg values fall within the expected alphabet {N, 5p, 3p, B, U} with no other tokens
- Tally of annotated miRNA-containing records matches the number of filtered hybrid records feeding downstream type/miRNA count tables
- Segment types assigned from identifiers (e.g. 'miRNA', 'mRNA') mirror the example_01 outputs shipped in the hybkit repository

## Limitations

- Segment typing is identifier-based; records whose segment reference names do not match hybkit's type_finder criteria will not resolve to meaningful types or a confident miRNA_seg value
- Annotation depends on prior quality control — records with artifact ribosomal/mitochondrial hybrids or duplicate HybRecord.id entries must be filtered first or counts will be inflated
- eval_types/eval_mirna require Python3.8+ and the hybkit dependency stack (matplotlib >= 3.7.1, BioPython >= 1.79, typing_extensions >= 4.8.0)

## Evidence

- [methods] eval_types and eval_mirna: "Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively"
- [methods] eval_types call: "# Analyze each record to assign segment types
hyb_record.eval_types()"
- [readme] hyb_eval description: "hyb_eval  Evaluate hyb (and fold) records to identify / assign segment types and miRNAs using custom criteria"
- [abstract] hybkit purpose: "analysis of hybrid sequence data in the "hyb" file format to enable customizable evaluation and annotation of hybrid characteristics"
