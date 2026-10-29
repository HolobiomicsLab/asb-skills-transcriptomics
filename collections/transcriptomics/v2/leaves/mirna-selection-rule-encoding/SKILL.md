---
name: mirna-selection-rule-encoding
description: Use when quantifying miRNAs and isomiRs from small RNA-seq reads with
  tiny-count and you must decide rule hierarchy, 5' offset, overlap range, and Partial
  flags for the selection-rules CSV.
when_to_use_negative:
- Input is already a per-feature count table — the hierarchical rule encoding step
  happens before counting, not after
- The target features are not small RNA variants needing 5'-end/length/overlap discrimination
  (e.g., bulk RNA-seq gene counts with existing tools like featureCounts)
- Annotations lack required GFF3 column-9 tags (ID/gene_id/Parent), which tiny-count
  requires for feature definitions
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: tiny-count
  role: Executes the hierarchical selection rules to classify and count miRNA/isomiR
    reads with single-nucleotide precision
  repo: github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of tiny-count; used for authoring and validating the
    selection-rules CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:e1f4aa207e49602441e219ceaa6da651b48bf0697f0b5ad61d7a99a75f59b64d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - tiny-count
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mirna-selection-rule-encoding

## Summary

Encode a hierarchical miRNA/isomiR counting rule set as a tiny-count selection-rules CSV, in which the canonical miRNA rule, shifted-5' isomiR rules, and a Partial-overlap catch-all rule are ranked so that each read is assigned by the highest-hierarchy matching rule.

## When to use

Use when quantifying miRNAs and isomiRs from small RNA-seq reads with tiny-count and you must decide rule hierarchy, 5' offset, overlap range, and Partial flags for the selection-rules CSV.

## When NOT to use

- Input is already a per-feature count table — the hierarchical rule encoding step happens before counting, not after
- The target features are not small RNA variants needing 5'-end/length/overlap discrimination (e.g., bulk RNA-seq gene counts with existing tools like featureCounts)
- Annotations lack required GFF3 column-9 tags (ID/gene_id/Parent), which tiny-count requires for feature definitions

## Inputs

- selection-rules CSV (hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, Partial flag columns)
- Reference feature annotations in GFF3 format (each feature with an ID, gene_id, or Parent tag)
- Collapsed small RNA reads in SAM format (produced from demultiplexed FASTQ data)

## Outputs

- selection-rules.csv rule table
- Per-feature read counts consistent with the hierarchical assignment order

## How to apply

Consult the tiny-count selection-rules CSV format (columns: hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, Partial flag). Write the canonical miRNA rule at hierarchy 1 with an exact 5' end match, overlap range 0-4, and read length 16-28; then add isomiR rules at hierarchy 2 with 5' offset -1 and hierarchy 3 with 5' offset +1, both retaining the 16-28 length constraint. Finally, assign the highest hierarchy value to the Partial-overlap catch-all rule so that only reads not assigned to the other miRNA rules are counted under it, and save the table as selection-rules.csv. Run tiny-count with the rules CSV, an annotated GFF3, and a collapsed SAM input to confirm the rule set parses and executes; hierarchy order enforces that canonical miRNA matches outrank shifted-5' isomiRs, which outrank partial overlaps.

## Related tools

- **tiny-count** (Executes the hierarchical selection rules to classify and count miRNA/isomiR reads with single-nucleotide precision) — github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of tiny-count; used for authoring and validating the selection-rules CSV)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- tiny-count completes the run with the rule file without parsing or execution errors
- Per-feature counts are consistent with the hierarchical assignment order (canonical miRNA rule outranks 5' offset -1/+1 isomiR rules, which outrank the Partial catch-all)
- Canonical miRNA reads with exact 5' end match, overlap 0-4, and length 16-28 are assigned to the hierarchy-1 rule, not to isomiR or Partial rules
- Partially overlapping reads are counted only under the Partial-overlap rule and never double-counted under higher-hierarchy rules
- Rule file contains one catch-all Partial row per miRNA feature class so no unassigned reads of the target class are dropped

## Limitations

- No quantitative threshold is reported in the paper for validating the rule set — acceptance is only that tiny-count runs and counts follow hierarchy order
- The Partial-overlap catch-all rule counts only reads not assigned to the other miRNA rules, so misordered hierarchies silently reassign reads
- GFF3 'Parent' tags with multiple values are not yet supported by the counting tool (README)

## Evidence

- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision.: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [readme] The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data with user-defined hierarchical feature selection rules.: "The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data with user-defined hierarchical feature selection rules."
- [readme] Each feature must have an `ID` or `gene_id`  or `Parent` tag: [evidence span withheld — anchor status: external_doc]
