---
name: small-rna-hierarchical-classification-rules
description: Use when small RNA-seq reads must be assigned to features by ordered
  precedence — e.g. separating canonical miRNAs from 5'-shifted isomiRs or partial-overlap
  fragments — rather than counted by simple overlap.
when_to_use_negative:
- Input is already a feature count table — no rule-based assignment of reads is needed
- Genome- or transcript-level alignment quantification without variant/class discrimination
  is sufficient (tiny-count does this too, but the rule CSV hierarchy is unnecessary)
- Annotations are discontinuous features with multi-valued Parent tags, which the
  README notes are not yet supported
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: tiny-count
  role: Executes the hierarchical selection-rules CSV against GFF3 annotations and
    collapsed SAM alignments to produce per-feature counts with single-nucleotide
    precision
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end workflow (CWL-coordinated) in which tiny-count's rule-based counting
    can be embedded
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language used to write/validate the selection-rules.csv rule
    table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:7d011a02a62ce3ef1ba302128465c1ec367d645620ccec6c47428457c231fa26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# small-rna-hierarchical-classification-rules

## Summary

Configure tiny-count selection rules to hierarchically classify small RNA-seq reads into distinct small RNA classes and variants (e.g. canonical miRNAs vs isomiRs) with single-nucleotide precision, using ordered rule rows in a selection-rules CSV.

## When to use

Use when small RNA-seq reads must be assigned to features by ordered precedence — e.g. separating canonical miRNAs from 5'-shifted isomiRs or partial-overlap fragments — rather than counted by simple overlap.

## When NOT to use

- Input is already a feature count table — no rule-based assignment of reads is needed
- Genome- or transcript-level alignment quantification without variant/class discrimination is sufficient (tiny-count does this too, but the rule CSV hierarchy is unnecessary)
- Annotations are discontinuous features with multi-valued Parent tags, which the README notes are not yet supported

## Inputs

- Collapsed SAM alignment file from small RNA-seq reads
- GFF3 (or GFF2/GTF) reference annotations with ID/gene_id/Parent attributes in column 9
- selection-rules.csv with columns: hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, Partial flag

## Outputs

- Per-feature counts assigned according to the hierarchical selection-rule order
- Pipeline statistics summarizing rule-based classification

## How to apply

Write a selection-rules.csv whose columns are hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, and the Partial flag. For the paper's miRNA/isomiR example: rule row hierarchy 1 captures canonical miRNAs via exact 5' end match with overlap range 0–4 and read length 16–28; hierarchy 2 and 3 rows capture isomiRs with 5' offsets of -1 and +1 respectively, retaining the length 16–28 constraint; a final catch-all row with the Partial flag enabled absorbs partially overlapping reads. Rules are evaluated in hierarchical order, so earlier (lower-hierarchy-number) rows take precedence and the Partial-overlap rule only counts reads not assigned to the other miRNA rules. Then run tiny-count with the rules CSV, the annotated GFF3 reference, and the collapsed SAM input; tiny-count filters reads by 5' nucleotide, length, alignment position relative to reference features, and mismatch number.

## Related tools

- **tiny-count** (Executes the hierarchical selection-rules CSV against GFF3 annotations and collapsed SAM alignments to produce per-feature counts with single-nucleotide precision) — https://github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end workflow (CWL-coordinated) in which tiny-count's rule-based counting can be embedded) — https://github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language used to write/validate the selection-rules.csv rule table)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- tiny-count completes the run with the selection-rules.csv, GFF3, and collapsed SAM without parsing errors
- Per-feature counts are consistent with the hierarchical assignment order: reads matching the canonical rule (exact 5' end, overlap 0–4, length 16–28) are not counted under the isomiR offset rules or the Partial catch-all
- Rule rows with 5' offset -1 and +1 only capture reads shifted by exactly one nucleotide at the 5' end relative to the miRNA
- The Partial-flag catch-all row yields counts only for reads that failed all higher-precedence miRNA/isomiR rules
- No quantitative threshold is specified in the paper for this check; acceptance is successful parsing and hierarchical-consistent counting

## Limitations

- Multi-valued Parent tags in annotation files are not yet supported by tiny-count's GFF3 requirements
- Chromosome identifiers in the reference FASTA are case sensitive and must match the annotation file's chromosome identifiers
- No quantitative acceptance threshold is reported in the paper for verifying the rule set — validation is limited to successful execution and consistency with hierarchical assignment order

## Evidence

- [abstract] Selection rules filter reads by 5' nucleotide, length, position, and mismatches.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] tiny-count distinguishes miRNAs and isomiRs with single-nucleotide precision.: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [methods] The Partial-overlap catch-all rule receives a hierarchy value ensuring it only counts unassigned reads.: [evidence span withheld — anchor status: not_found]
- [readme] tinyRNA provides a workflow with user-defined hierarchical feature selection rules.: "tinyRNA is a set of tools to simplify the analysis of next-generation sequencing data. The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data"
