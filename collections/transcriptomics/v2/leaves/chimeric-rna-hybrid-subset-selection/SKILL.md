---
name: chimeric-rna-hybrid-subset-selection
description: Use when a hyb-format file (with optional associated vienna/CT fold files)
  from CLASH/qCLASH microRNA/target ligation experiments must be subset to records
  matching identifier substrings, segment-type criteria, or de-duplicated by record
  id.
when_to_use_negative:
- When the goal is quantification or plotting of type/miRNA/target/fold summaries
  rather than record selection — hyb_analyze is the appropriate utility.
- When records need segment types assigned or miRNAs identified under custom criteria
  rather than filtered on existing annotations — hyb_eval is designed for that step.
- When input data is not in hyb format or lacks the annotated flags-string/column-16
  properties the filter criteria rely on.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3512
tools:
- name: hybkit
  role: Provides hyb_filter CLI utility and Python API (HybFile, HybRecord) for parsing,
    property-testing, and writing filtered hyb records
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime environment (Python 3.8+) for the hybkit API used to iterate and filter
    HybRecord objects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:b8e2eb2120a03f7ae772963c75a6ae382c999ab3d011bee9c29905c01b1e93ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python3
  - hybkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# chimeric-rna-hybrid-subset-selection

## Summary

Selection of hybrids with specific properties from hyb-format chimeric RNA data using hybkit's hyb_filter utility and its Python API, enabling record-level filtration by sequence identifiers, segment types, miRNA flags, and duplicate removal within command-line workflows.

## When to use

Use when a hyb-format file (with optional associated vienna/CT fold files) from CLASH/qCLASH microRNA/target ligation experiments must be subset to records matching identifier substrings, segment-type criteria, or de-duplicated by record id.

## When NOT to use

- When the goal is quantification or plotting of type/miRNA/target/fold summaries rather than record selection — hyb_analyze is the appropriate utility.
- When records need segment types assigned or miRNAs identified under custom criteria rather than filtered on existing annotations — hyb_eval is designed for that step.
- When input data is not in hyb format or lacks the annotated flags-string/column-16 properties the filter criteria rely on.

## Inputs

- hyb-format file (.hyb) of chimeric/hybrid RNA records from CLASH or qCLASH
- associated vienna (.vienna) or CT fold file (optional)
- filter criteria (e.g. --filter any_seg_contains <string>, segment-type or miRNA_seg conditions)

## Outputs

- filtered .hyb file containing only records passing the criteria
- filtered .vienna fold file corresponding to the retained hybrid records (when a fold input is given)

## How to apply

Load the annotated hyb file with the hybkit HybFile iterator, which parses each tab-separated 15–16 column hyb record (id, sequence, energy, segment mapping fields, and the optional column-16 flags string with entries like seg1_type and miRNA_seg) into HybRecord objects. Call hyb_record.eval_types() to assign segment types before filtering when type- or miRNA-based criteria are required. Apply the user-specified criteria via record-property tests — e.g. the --filter any_seg_contains kshv substring match on reference identifiers, segment-type membership such as seg1_type == 'miRNA', or has_prop('any_seg_type_contains', 'lncRNA') in the API — skipping non-matching records. Optionally remove duplicate hybrids using hyb_filter's duplicate-hybrid filtration option keyed on HybRecord.id. Write all passing HybRecord objects back out in hyb format (and the fold records to vienna format when a fold file is supplied) to produce the filtered outputs.

## Related tools

- **hybkit** (Provides hyb_filter CLI utility and Python API (HybFile, HybRecord) for parsing, property-testing, and writing filtered hyb records) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime environment (Python 3.8+) for the hybkit API used to iterate and filter HybRecord objects)

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- Every record in the output .hyb file satisfies the applied filter (e.g. each id or segment reference contains the requested substring such as 'kshv').
- Record count in the output equals the number of passing records observed during iteration; no records were dropped or duplicated.
- Output records remain valid 15–16 column tab-separated hyb records with intact id, sequence, energy, and segment mapping fields.
- When a vienna file is co-filtered, the fold-file record count and ordering match the filtered hyb file.
- Duplicate-removal run yields a record set where all HybRecord.id values are unique.

## Limitations

- Filtration is limited to properties encoded in the hyb record (identifiers, segment fields, flags string such as seg1_type/miRNA_seg); segment types must be assigned (e.g. via eval_types) before type-based filtering is meaningful.
- Duplicate hybrid filtration is keyed only on HybRecord.id, so records with differing ids but otherwise identical content are not collapsed.
- Requires hyb-format input (15–16 column records) as defined by the Hyb software package; other chimeric-RNA formats are not directly supported.

## Evidence

- [readme] hyb_filter Filter hyb (and fold) records to a specific custom subset: "hyb_filter                          Filter hyb (and fold) records to a specific
                                            custom subset"
- [readme] Example filter invocation on hyb and vienna files: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [readme] API property test example on HybRecord: "if hyb_record.has_prop('any_seg_type_contains', 'lncRNA')
                    print(hyb_record.id)"
- [methods] Workflow position of hyb_filter for removing artifactual hybrids: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [methods] Duplicate filtration option keyed on HybRecord.id: "Add duplicate hybrid filtration (by HybRecord.id) options to hyb_filter"
