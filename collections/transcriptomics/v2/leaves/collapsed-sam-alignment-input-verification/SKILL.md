---
name: collapsed-sam-alignment-input-verification
description: Use when preparing a small RNA-seq counting run with tiny-count and you
  have reads aligned to a genome or to small RNA/transcript sequences as a collapsed
  SAM that must be checked against a GFF3 annotation and a selection-rules CSV.
when_to_use_negative:
- Input is raw, demultiplexed FASTQ that has not yet been collapsed and aligned —
  pre-processing/alignment steps come first
- A per-feature count table already exists and only downstream steps (differential
  expression, plots) are needed
- The alignment file targets a reference whose chromosome or feature identifiers do
  not match the annotation file and cannot be reconciled — tiny-count's identifier
  matching would silently fail to assign reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: tiny-count
  role: Consumes the collapsed SAM, GFF3 annotation, and selection-rules CSV to perform
    hierarchical classification and quantification of small RNA reads; its successful
    parse-and-run is the verification signal
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end workflow that coordinates the counting step; tiny-count can be
    run alone or as part of tinyRNA
  repo: https://github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:66d6dd498794187d297c72189911d57f9310f7c9e3da8a143b049293f9e39565
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# collapsed-sam-alignment-input-verification

## Summary

Verify that the collapsed SAM alignment file destined for tiny-count is consistent with the reference annotation and selection rules before hierarchical counting, so that miRNA/isomiR assignment by 5' nucleotide, length, and alignment position is computed on a valid input.

## When to use

Use when preparing a small RNA-seq counting run with tiny-count and you have reads aligned to a genome or to small RNA/transcript sequences as a collapsed SAM that must be checked against a GFF3 annotation and a selection-rules CSV.

## When NOT to use

- Input is raw, demultiplexed FASTQ that has not yet been collapsed and aligned — pre-processing/alignment steps come first
- A per-feature count table already exists and only downstream steps (differential expression, plots) are needed
- The alignment file targets a reference whose chromosome or feature identifiers do not match the annotation file and cannot be reconciled — tiny-count's identifier matching would silently fail to assign reads

## Inputs

- Collapsed SAM alignment file of small RNA-seq reads (from collapsed FASTA input)
- Reference annotations in GFF3 / GFF2 / GTF with ID or gene_id or Parent tags in column 9
- Selection rules file (selection-rules.csv) with hierarchy, feature type, 5' nucleotide, length, overlap, 5' offset, and Partial flag columns
- Reference genome FASTA with chromosome identifiers matching the annotation (optional, for genome-aligned reads)

## Outputs

- Per-feature count table with hierarchical assignment of reads to miRNA/isomiR and other small RNA features
- Confirmation that tiny-count parsed the rules CSV and executed without errors

## How to apply

First confirm the SAM was produced from collapsed sequencing reads (tiny-count accepts quantification of reads aligned to a genome or directly to small RNA or transcript sequences). Next, check that chromosome identifiers in the SAM match the reference annotation file exactly, since the tinyRNA documentation requires identifiers like Chr1 to match the GFF3 and to be case sensitive. Assemble the counting inputs: the annotated GFF3 (whose column 9 attributes must carry an ID or gene_id or Parent tag), the selection-rules.csv encoding the miRNA/isomiR hierarchy (hierarchy 1 canonical miRNA with exact 5' end match and overlap 0-4 at length 16-28; hierarchies 2-3 isomiRs with 5' offsets -1/+1; lowest hierarchy Partial-overlap catch-all), and the collapsed SAM. Run tiny-count with these inputs and confirm it parses the rule file and completes without errors. Judgment criterion: tiny-count finishes the run and produces per-feature counts consistent with the hierarchical assignment order; the paper reports no quantitative threshold for this check.

## Related tools

- **tiny-count** (Consumes the collapsed SAM, GFF3 annotation, and selection-rules CSV to perform hierarchical classification and quantification of small RNA reads; its successful parse-and-run is the verification signal) — https://github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end workflow that coordinates the counting step; tiny-count can be run alone or as part of tinyRNA) — https://github.com/MontgomeryLab/tinyRNA

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- tiny-count completes the run with the rules CSV, annotated GFF3, and collapsed SAM without parse or execution errors
- Per-feature counts are consistent with the hierarchical assignment order (canonical miRNA rule at hierarchy 1 claims reads before isomiR offsets -1/+1, and the Partial catch-all only receives unassigned reads)
- Chromosome identifiers in the SAM match the GFF3 annotation file, respecting case sensitivity
- Every GFF3 feature carries an ID or gene_id or Parent tag in column 9 so no feature is silently dropped from counting
- Reads classified as miRNA vs isomiR are distinguishable with single-nucleotide precision as the tool advertises

## Limitations

- GFF3 Parent tags with multiple values are not yet supported by the annotation parser
- Attribute values containing commas must represent lists, which can break nonconforming annotation files
- Stable tinyRNA releases may have installation issues on Apple Silicon-based systems
- No quantitative validation threshold is reported in the paper for confirming the rule set parses and executes

## Evidence

- [abstract] tiny-count can quantify reads aligned to a genome or directly to small RNA or transcript sequences.: "tiny-count can quantify reads aligned to a genome or directly to small RNA or transcript sequences."
- [abstract] Selection rules filtering: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [readme] Chromosome identifier matching requirement: [evidence span withheld — anchor status: external_doc]
- [readme] GFF3 column 9 attribute requirement: [evidence span withheld — anchor status: external_doc]
