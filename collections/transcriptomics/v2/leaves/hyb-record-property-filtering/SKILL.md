---
name: hyb-record-property-filtering
description: Use when a hyb-format file from a CLASH/qCLASH experiment must be subset
  by record properties — e.
when_to_use_negative:
- The data are not in hyb format or lack per-record segment annotation needed for
  the property tests (segment types must be assigned via hyb_eval/eval_types first).
- The goal is quantification or plotting of type/miRNA/target/fold distributions rather
  than subsetting records — use hyb_analyze instead.
- The input is FASTQ reads or raw chimeric alignments; hyb records must already exist
  from an upstream pipeline such as Hyb.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Hybkit
  role: Provides the hyb_filter command-line utility and the HybRecord/HybFile API
    used to load, property-test, de-duplicate, and rewrite hyb (and paired vienna)
    records
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API and toolkit (Python3.8+ required); enables custom
    record-property filtering via HybRecord methods such as eval_types() and has_prop()
  repo: github.com/RenneLab/hybkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:0ac3a14ad8add0e26e73292775abd3ce4d869eb1adee8d3cb10621c0cc55c7e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python3
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# hyb-record-property-filtering

## Summary

Selecting subsets of hybrid records from hyb-format chimeric RNA data by record properties (segment-type membership, reference-identifier substring, miRNA flags) and duplicate removal, using the hybkit hyb_filter command-line tool or the HybRecord API. It underpins quality control of microRNA/target ligation (CLASH/qCLASH) datasets.

## When to use

Use when a hyb-format file from a CLASH/qCLASH experiment must be subset by record properties — e.g. keeping only hybrids whose segments contain a reference-identifier string like 'kshv', are of a given seg1_type such as 'miRNA', or when artifactual ribosomal-/mitochondrial-RNA hybrids or duplicate HybRecord.id entries must be removed.

## When NOT to use

- The data are not in hyb format or lack per-record segment annotation needed for the property tests (segment types must be assigned via hyb_eval/eval_types first).
- The goal is quantification or plotting of type/miRNA/target/fold distributions rather than subsetting records — use hyb_analyze instead.
- The input is FASTQ reads or raw chimeric alignments; hyb records must already exist from an upstream pipeline such as Hyb.

## Inputs

- hyb-format hybrid file (.hyb) with 15–16 tab-separated columns, optionally annotated with a column-16 flags string (seg1_type, miRNA_seg, etc.)
- Paired predicted-secondary-structure file in vienna or CT format (optional, supplied with -f)

## Outputs

- Filtered .hyb file containing only HybRecord objects passing the criteria, in hyb format
- Paired filtered vienna/CT fold file when a fold file was supplied

## How to apply

Parse the annotated hyb file with hybkit's HybFile iterator, reading each tab-separated 15–16 column record (id, sequence, energy, segment fields, and the optional column-16 flags string carrying entries such as seg1_type and miRNA_seg) into HybRecord objects. Apply the chosen record-property test to each record — e.g. --filter any_seg_contains kshv for substring matching on reference identifiers, or segment-type membership such as seg1_type == 'miRNA' / miRNA_seg presence evaluated after calling hyb_record.eval_types() in the API — skipping non-matching records. Enable hyb_filter's duplicate hybrid filtration, keyed on HybRecord.id, to remove duplicate hybrids when configured. Write all passing HybRecord objects back out in hyb format, and if a paired vienna/CT fold file is supplied via -f, keep it in lockstep with the filtered records. In the standard toolkit workflow this step sits after hyb_check quality checking and is used for removing artifactual ribosomal- and mitochondrial-RNA hybrids before type/miRNA, target, or fold analyses.

## Related tools

- **Hybkit** (Provides the hyb_filter command-line utility and the HybRecord/HybFile API used to load, property-test, de-duplicate, and rewrite hyb (and paired vienna) records) — github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API and toolkit (Python3.8+ required); enables custom record-property filtering via HybRecord methods such as eval_types() and has_prop()) — github.com/RenneLab/hybkit

## Examples

```
$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- Every record in the filtered .hyb satisfies the filter predicate (e.g. at least one segment identifier contains the requested substring, or seg1_type matches the requested type).
- The filtered output remains valid hyb format with 15–16 tab-separated columns per record and no records lost other than non-matching/duplicate ones.
- When duplicate filtration is enabled, no HybRecord.id appears more than once in the output.
- When a paired vienna/CT file is given with -f, the fold file and hyb file contain the same number of records in the same order.
- When used for QC, ribosomal- and mitochondrial-RNA hybrids are absent from the filtered output.

## Limitations

- Segment-type-based tests require prior type assignment; records must be evaluated (eval_types/eval_mirna or hyb_eval) before type membership can be filtered.
- Substring filters such as any_seg_contains are reference-identifier based (e.g. 'kshv'); identifiers not embedding the organism/feature string will not match.
- hybkit requires Python3.8+ and depends on matplotlib, BioPython, and typing_extensions; the toolkit operates on hyb files and their associated vienna/CT fold files only.

## Evidence

- [readme] hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [methods] removing artifactual ribosomal- and mitochondrial-RNA hybrids (hyb_filter): "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [methods] Add duplicate hybrid filtration (by HybRecord.id) options to hyb_filter: "Add duplicate hybrid filtration (by HybRecord.id) options to hyb_filter"
- [readme] hyb_filter Filter hyb (and fold) records to a specific custom subset: "hyb_filter                          Filter hyb (and fold) records to a specific
                                            custom subset"
