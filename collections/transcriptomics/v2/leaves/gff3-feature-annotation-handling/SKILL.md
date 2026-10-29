---
name: gff3-feature-annotation-handling
description: Use when supplying reference annotations (GFF3/GFF2/GTF) to tiny-count
  or tinyRNA for hierarchical small RNA-seq quantification, and feature IDs, Parent
  relationships, or genome/annotation identifier matching must be validated first.
when_to_use_negative:
- Input is already a per-feature count table rather than reference annotations
- Features lack any `ID`/`gene_id`/`Parent` column-9 tags and cannot be reformatted
  to satisfy tiny-count's GFF3 requirements
- '`Parent` tags carry multiple values and the annotation cannot be restructured,
  since multi-value Parent tags are not yet supported'
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0622
tools:
- name: tiny-count
  role: Parses the validated GFF3 annotation together with the selection-rules.csv
    and collapsed SAM input to hierarchically classify and quantify small RNA reads
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of tiny-count and the tinyRNA tools that consume the
    annotation files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:872d8e2db56d58298a8329d8ea6096273c528a1cfa814391d3986aa820ef6fbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - Python
  - tiny-count
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gff3-feature-annotation-handling

## Summary

Prepares and validates GFF3 (or GFF2/GTF) reference annotation files so tiny-count can parse features and assign small RNA-seq reads to them under hierarchical selection rules. Correct column-9 attributes and chromosome identifier consistency are the key failure points.

## When to use

Use when supplying reference annotations (GFF3/GFF2/GTF) to tiny-count or tinyRNA for hierarchical small RNA-seq quantification, and feature IDs, Parent relationships, or genome/annotation identifier matching must be validated first.

## When NOT to use

- Input is already a per-feature count table rather than reference annotations
- Features lack any `ID`/`gene_id`/`Parent` column-9 tags and cannot be reformatted to satisfy tiny-count's GFF3 requirements
- `Parent` tags carry multiple values and the annotation cannot be restructured, since multi-value Parent tags are not yet supported

## Inputs

- Reference annotation file (GFF3, GFF2, or GTF) with column-9 tag=value attributes
- Reference genome FASTA with matching, case-sensitive chromosome identifiers
- tiny-count selection-rules.csv (hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, Partial flag)
- Collapsed SAM input of small RNA-seq alignments

## Outputs

- Validated/parsable GFF3 feature set consumed by tiny-count
- Per-feature small RNA read counts consistent with hierarchical rule assignment

## How to apply

Check column 9 of every feature in the annotation file: each feature must carry an `ID` or `gene_id` or `Parent` tag (treated as the `ID` for counting). Discontinuous features must be linked either by a `Parent` tag whose value is the logical parent's `ID` or by sharing the same `ID`, so tiny-count resolves them as one logical feature. Attribute values containing commas must represent lists, and `Parent` tags with multiple values are not supported, so such records must be rewritten before counting. Cross-check chromosome identifiers in the annotation against the reference genome FASTA — they must match exactly and are case sensitive (e.g. Chr1). The validated annotation file then feeds tiny-count together with a selection-rules.csv (hierarchy, 5' nucleotide, length, overlap, 5' offset, Partial flag columns) and collapsed SAM input to produce per-feature counts.

## Related tools

- **tiny-count** (Parses the validated GFF3 annotation together with the selection-rules.csv and collapsed SAM input to hierarchically classify and quantify small RNA reads) — https://github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of tiny-count and the tinyRNA tools that consume the annotation files)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- tiny-count parses the annotation file and completes the run with the rules CSV without errors
- Every feature in the GFF3 resolves to an `ID` (directly or via `Parent`), and no feature errors are emitted during parsing
- Discontinuous features collapse to their logical parent rather than being counted as separate features
- Chromosome identifiers in annotation and genome FASTA match exactly (case sensitive), so alignments are attributed to the intended features
- Produced per-feature counts are consistent with the hierarchical assignment order of the selection rules

## Limitations

- `Parent` tags with multiple values are not yet supported by tiny-count's annotation parser
- Attribute values containing commas must represent lists — comma-containing scalars will not parse as intended
- Chromosome identifiers are case sensitive and must exactly match the reference genome FASTA (e.g. Chr1), so silently mismatched identifiers cause features to go uncounted
- The paper reports no quantitative threshold for validating the rule/annotation run; acceptance is a clean, error-free tiny-count execution

## Evidence

- [readme] Each feature must have an `ID` or `gene_id`  or `Parent` tag (referred to as `ID` henceforth).: [evidence span withheld — anchor status: external_doc]
- [readme] Discontinuous features must be defined with the Parent tag whose value is the logical parent's ID, or by sharing the same ID.: [evidence span withheld — anchor status: external_doc]
- [readme] Chromosome identifiers (e.g. Chr1): Should match your reference annotation file chromosome identifiers Are case sensitive: [evidence span withheld — anchor status: external_doc]
- [abstract] It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision.: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
