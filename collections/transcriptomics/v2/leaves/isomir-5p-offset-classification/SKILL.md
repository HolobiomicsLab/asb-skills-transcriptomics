---
name: isomir-5p-offset-classification
description: Use when counting small RNA-seq reads where miRNAs must be distinguished
  from isomiRs by single-nucleotide 5' end shifts — e.g. you have collapsed reads
  (SAM/FASTA), a GFF3 annotation, and need hierarchical 5'-offset-based assignment.
when_to_use_negative:
- Reads have already been assigned to features — no hierarchical rule-based counting
  is needed
- You only need bulk quantification without distinguishing miRNAs from isomiRs by
  5' end position
- Your annotations/reads are not small RNA-seq data (tiny-count targets small RNA
  reads such as miRNA, isomiR, piRNA, siRNA, tRNA, rRNA fragments)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: tiny-count
  role: Executes the hierarchical selection-rules CSV against the annotated GFF3 and
    collapsed SAM input to produce per-feature counts with single-nucleotide precision
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Language in which tiny-count is implemented; usable to write/validate the
    selection-rules.csv rows
  repo: https://github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:78ff78d75992501e5b0ec95bcc50a8f6cfd4e07934a87a924ed73cb67e025e72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - Python
  - tiny-count
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# isomir-5p-offset-classification

## Summary

Reconstructs tiny-count selection-rules.csv rows that classify miRNAs vs isomiRs with single-nucleotide precision: a hierarchy-1 exact 5' end rule (overlap 0-4, length 16-28), hierarchy-2/-3 isomiR rules with 5' offsets of -1 and +1, and a lowest-hierarchy Partial-overlap catch-all so only unassigned reads fall through.

## When to use

Use when counting small RNA-seq reads where miRNAs must be distinguished from isomiRs by single-nucleotide 5' end shifts — e.g. you have collapsed reads (SAM/FASTA), a GFF3 annotation, and need hierarchical 5'-offset-based assignment.

## When NOT to use

- Reads have already been assigned to features — no hierarchical rule-based counting is needed
- You only need bulk quantification without distinguishing miRNAs from isomiRs by 5' end position
- Your annotations/reads are not small RNA-seq data (tiny-count targets small RNA reads such as miRNA, isomiR, piRNA, siRNA, tRNA, rRNA fragments)

## Inputs

- selection-rules.csv (tiny-count rule table: hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, Partial flag)
- Annotated reference features in GFF3 (GFF2/GTF accepted by tiny-count)
- Collapsed small RNA sequencing reads (collapsed FASTA / SAM alignment input)

## Outputs

- Per-feature count table reflecting hierarchical assignment order (canonical miRNA vs 5'-offset isomiRs vs Partial catch-all)
- Validated selection-rules.csv rule configuration

## How to apply

Consult the Section 2.3 miRNA rule specification and the tiny-count selection-rules CSV format, whose columns are hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, and Partial flag. Write the canonical miRNA rule row as hierarchy 1 with an exact 5' end match, overlap range 0-4, and read length 16-28 nt. Write two isomiR rows: hierarchy 2 with 5' offset -1 and hierarchy 3 with 5' offset +1, both retaining the length 16-28 constraint, so single-nucleotide 5' shifts are captured separately from the canonical miRNA. Write the lowest-hierarchy catch-all row with the Partial flag enabled to capture partially overlapping reads; assigning it a higher hierarchy value ensures only reads not assigned to the other miRNA rules are counted under it. Save the table as selection-rules.csv, then run tiny-count with the rules CSV, the annotated GFF3, and the collapsed SAM input to confirm the rule set parses and executes without errors.

## Related tools

- **tiny-count** (Executes the hierarchical selection-rules CSV against the annotated GFF3 and collapsed SAM input to produce per-feature counts with single-nucleotide precision) — https://github.com/MontgomeryLab/tinyRNA
- **Python** (Language in which tiny-count is implemented; usable to write/validate the selection-rules.csv rows) — https://github.com/MontgomeryLab/tinyRNA

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- tiny-count completes the run with the selection-rules.csv and the annotated GFF3 + collapsed SAM input without errors
- Per-feature counts are consistent with the hierarchical assignment order (hierarchy 1 canonical miRNA > hierarchy 2 offset -1 > hierarchy 3 offset +1 > Partial catch-all)
- Reads with exact 5' end match and overlap 0-4 nt, length 16-28, are counted under the canonical miRNA rule, not the isomiR or Partial rules
- Only reads not assigned to the other miRNA rules appear under the Partial-overlap catch-all rule
- Rule table parses with all seven columns present: hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, Partial flag

## Limitations

- The paper reports no quantitative threshold for validating the rule-set run — correctness is judged only by successful execution and hierarchy-consistent counts
- Partial-overlap catch-all must have lower precedence; a higher hierarchy value would incorrectly absorb reads that belong to the canonical or offset rules
- Reads falling outside the length 16-28 constraint are not captured by the miRNA/isomiR rule rows

## Evidence

- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision.: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [readme] The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data with user-defined hierarchical feature selection rules.: "The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data with user-defined hierarchical feature selection rules."
