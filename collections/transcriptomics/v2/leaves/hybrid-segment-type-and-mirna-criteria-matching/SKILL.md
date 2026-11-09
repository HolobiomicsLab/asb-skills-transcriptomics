---
name: hybrid-segment-type-and-mirna-criteria-matching
description: Use when a hyb-format file of CLASH/qCLASH hybrids must be subset to
  records whose segment types, miRNA assignments, or sequence identifiers match specified
  criteria (e.g. keep only miRNA-containing hybrids or hybrids with 'kshv' in any
  segment).
when_to_use_negative:
- The input is not hyb-format hybrid data (e.g. raw FASTQ reads or an unannotated
  alignment) — records cannot be parsed into HybRecord objects or evaluated for segment
  types.
- The goal is counting or plotting type/miRNA distributions rather than subsetting
  records — use hybkit's type/miRNA, target, or fold analyses (hyb_analyze) instead
  of filtering.
- Segment types and miRNA identifiers have not been evaluated on the records — property
  tests like seg1_type or miRNA_seg will be unreliable until eval_types()/eval_mirna()
  has run.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: Hybkit
  role: Provides hyb_filter (command-line filtering of hyb and fold records to a custom
    subset) and the HybFile/HybRecord API with eval_types, eval_mirna, and has_prop
    record-property tests used to match segment-type and miRNA criteria
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API workflow that iterates a HybFile and applies has_prop
    property tests per record
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:daf7664e623bc77942e399403846378e0f8dc77e5aff3d65b0a0d972c0eaae4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python3
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hybrid-segment-type-and-mirna-criteria-matching

## Summary

Selects a custom subset of hybrid records from hyb-format chimeric RNA data by testing each HybRecord against segment-type and miRNA criteria (e.g. seg1_type == 'miRNA', any_seg_contains substring matches on reference identifiers), via the hybkit hyb_filter utility or the HybRecord property-test API.

## When to use

Use when a hyb-format file of CLASH/qCLASH hybrids must be subset to records whose segment types, miRNA assignments, or sequence identifiers match specified criteria (e.g. keep only miRNA-containing hybrids or hybrids with 'kshv' in any segment).

## When NOT to use

- The input is not hyb-format hybrid data (e.g. raw FASTQ reads or an unannotated alignment) — records cannot be parsed into HybRecord objects or evaluated for segment types.
- The goal is counting or plotting type/miRNA distributions rather than subsetting records — use hybkit's type/miRNA, target, or fold analyses (hyb_analyze) instead of filtering.
- Segment types and miRNA identifiers have not been evaluated on the records — property tests like seg1_type or miRNA_seg will be unreliable until eval_types()/eval_mirna() has run.

## Inputs

- hyb-format file (.hyb) of chimeric RNA hybrids with 15–16 tab-separated columns per record
- optional associated Vienna (.vienna) or CT fold file aligned record-by-record with the hyb file
- user-specified filter criteria (substring, segment-type, miRNA, or flags-string properties)

## Outputs

- filtered .hyb file containing only HybRecord objects that match the criteria
- matched filtered .vienna fold file when a fold input was provided

## How to apply

Load the annotated .hyb file with the hybkit HybFile iterator, parsing each tab-separated 15–16 column record (id, sequence, energy, segment mapping fields, and optional column-16 flags such as seg1_type and miRNA_seg) into HybRecord objects. First run hyb_record.eval_types() (and eval_mirna for miRNA analysis) to assign segment types and miRNA identifiers from the record's reference identifiers, since property tests depend on these evaluated attributes. Then apply record-property tests via hyb_filter's --filter options or the API's hyb_record.has_prop() — for example any_seg_contains for substring matching on reference identifiers, membership tests such as seg1_type == 'miRNA', and miRNA_seg presence — skipping records that fail. Optionally enable hyb_filter's duplicate hybrid filtration keyed on HybRecord.id. Write passing records back out in hyb format (with the matched .vienna fold file when one is supplied alongside).

## Related tools

- **Hybkit** (Provides hyb_filter (command-line filtering of hyb and fold records to a custom subset) and the HybFile/HybRecord API with eval_types, eval_mirna, and has_prop record-property tests used to match segment-type and miRNA criteria) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API workflow that iterates a HybFile and applies has_prop property tests per record)

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- Every record in the output .hyb file passes the specified property test (e.g. each kept record has any segment whose reference identifier contains the filter substring, or seg1_type == 'miRNA').
- Records removed by the duplicate filtration option share no HybRecord.id values with other retained records.
- Output line count equals the number of input records whose evaluated types/miRNAs matched, and each line retains the 15–16 tab-separated column structure of the hyb format.
- When a .vienna fold file is supplied, the filtered fold file remains record-aligned with the filtered .hyb file (same ids, same order).
- Running hyb_check on the filtered output reports no format errors.

## Limitations

- Property tests operate on evaluated record attributes, so filter results depend on hybkit's type/miRNA evaluation criteria (eval_types/eval_mirna) having been applied, per the documented renaming of type_analysis and mirna_analysis to eval_types and eval_mirna.
- Filtering is keyed on hyb-format annotation (reference identifiers, flags-string entries such as seg1_type and miRNA_seg); records lacking the optional column-16 flags string cannot be matched on those flags.
- As stated in the article workflow, records that do not match the user-specified criteria are skipped — no rescue or fuzzy matching is performed beyond hybkit's provided substring and membership tests.

## Evidence

- [readme] hyb_filter command example: "to filter a hyb and corresponding vienna file to contain only hybrids with a sequence identifier containing the string "kshv""
- [methods] filter steps (article methods): "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [readme] property-test API usage: "hyb_record.eval_types(); # If the record contains a long noncoding RNA type, print the record identifier. if hyb_record.has_prop('any_seg_type_contains', 'lncRNA')"
- [methods] duplicate filtration option: "Add duplicate hybrid filtration (by HybRecord.id) options to hyb_filter"
- [readme] hyb_filter role in toolkit: "hyb_filter                          Filter hyb (and fold) records to a specific custom subset"
