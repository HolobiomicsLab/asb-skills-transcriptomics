---
name: duplicate-hybrid-record-removal
description: Use when a .hyb file (optionally with an associated vienna/CT fold file)
  contains repeated hybrid records and you need a deduplicated subset before type/miRNA,
  target, or fold analysis.
when_to_use_negative:
- The hyb file has already been deduplicated, or each hybrid identifier is guaranteed
  unique — the duplicate filtration step would be a no-op
- The task is quantifying abundance, where duplicate counts carry information (e.g.
  type/miRNA Analysis or Target Analysis counting hybrids) rather than producing a
  unique-record subset
- The input is not hyb-format chimeric RNA data (e.g. FASTQ or BAM), since HybRecord
  parsing and id-based deduplication depend on the hyb record structure
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3511
tools:
- name: hybkit
  role: Provides hyb_filter, the command-line utility whose duplicate hybrid filtration
    (by HybRecord.id) option removes repeated hybrid records, and the HybFile/HybRecord
    API used to parse and write hyb records
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime environment (Python3.8+) for the hybkit API and command-line toolkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:e1d6bd9b6acce80a232a4a258d11d0ea53dfb6e54c55a11a5a7f98f0a2157f7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python3
  - hybkit
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# duplicate-hybrid-record-removal

## Summary

Removes duplicate hybrids from hyb-format chimeric RNA data using hyb_filter's duplicate hybrid filtration option, which keys duplicates on the HybRecord.id sequence identifier so each hybrid is retained only once.

## When to use

Use when a .hyb file (optionally with an associated vienna/CT fold file) contains repeated hybrid records and you need a deduplicated subset before type/miRNA, target, or fold analysis.

## When NOT to use

- The hyb file has already been deduplicated, or each hybrid identifier is guaranteed unique — the duplicate filtration step would be a no-op
- The task is quantifying abundance, where duplicate counts carry information (e.g. type/miRNA Analysis or Target Analysis counting hybrids) rather than producing a unique-record subset
- The input is not hyb-format chimeric RNA data (e.g. FASTQ or BAM), since HybRecord parsing and id-based deduplication depend on the hyb record structure

## Inputs

- hyb-format file (.hyb) of chimeric RNA hybrid records with 15–16 tab-separated columns, including the id field and optional column-16 flags string
- associated vienna or CT fold file (optional, passed with -f)

## Outputs

- filtered .hyb file containing only records passing the filter criteria with duplicates removed (one record per unique HybRecord.id)
- corresponding filtered vienna/CT fold records when a fold file was supplied

## How to apply

Load the annotated hyb file with the hybkit HybFile iterator, parsing each tab-separated 15–16 column hyb record (id, sequence, energy, segment mapping fields, and the optional column-16 flags string) into HybRecord objects. Apply any user-specified filter criteria first — e.g. --filter any_seg_contains kshv substring matching on reference identifiers, or segment-type properties such as seg1_type == 'miRNA' from the flags string — skipping non-matching records. Then enable hyb_filter's duplicate hybrid filtration option, which keys duplicates on HybRecord.id so records sharing an identifier are collapsed. Write all surviving HybRecord objects back out in hyb format (and the matching vienna records when a fold file is supplied with -f) to the filtered output file. Deduplication is deliberately placed after property filtering so that duplicates are removed only within the desired subset.

## Related tools

- **hybkit** (Provides hyb_filter, the command-line utility whose duplicate hybrid filtration (by HybRecord.id) option removes repeated hybrid records, and the HybFile/HybRecord API used to parse and write hyb records) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime environment (Python3.8+) for the hybkit API and command-line toolkit)

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- Number of records in the output .hyb file equals the number of unique HybRecord.id values in the input subset — no identifier appears more than once
- Every output record's id matches at least one input record id, and all output records satisfy the applied filter criteria (e.g. any_seg_contains kshv)
- When a fold file is supplied with -f, the output vienna records correspond one-to-one (by identifier) with the output hyb records
- Running hyb_check on the filtered output parses without errors, confirming record integrity (15–16 columns, valid segment fields)
- Downstream hyb_analyze type/miRNA analysis on the deduplicated file completes and yields counts consistent with unique records only

## Limitations

- Deduplication is keyed solely on HybRecord.id; hybrids differing in sequence or annotation but sharing an identifier are treated as duplicates, and distinct identifiers of identical sequences are not collapsed
- hyb_filter's duplicate filtration operates on records that already passed the other filter criteria, so the order of filtering and deduplication affects which copy of a duplicate is retained
- Removing duplicates is inappropriate when hybrid counts are meaningful downstream, such as in the quantification performed by Type/miRNA or Target Analyses

## Evidence

- [methods] Duplicate hybrid filtration (by HybRecord.id) options added to hyb_filter: "Add duplicate hybrid filtration (by HybRecord.id) options to hyb_filter"
- [readme] hyb_filter filters hyb and fold records to a custom subset: "hyb_filter                          Filter hyb (and fold) records to a specific
                                            custom subset"
- [readme] Example filtering a hyb and vienna file to hybrids with a sequence identifier containing kshv: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [methods] hyb_filter is used to remove artifactual ribosomal- and mitochondrial-RNA hybrids: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
