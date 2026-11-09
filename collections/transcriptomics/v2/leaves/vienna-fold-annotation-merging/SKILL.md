---
name: vienna-fold-annotation-merging
description: Use when a qCLASH/CLASH analysis requires each hyb-format hybrid record
  to carry its predicted folding pattern (dot-bracket) and energy from a paired Vienna
  or CT file, e.g. before Fold Analysis of miRNA folding patterns.
when_to_use_negative:
- The hyb file has no paired Vienna/CT fold file, so records cannot be matched by
  identifier and no fold annotation can be merged.
- The goal is filtering rather than annotation — hyb_filter operations like 'any_seg_contains
  kshv' or duplicate/ribosomal/mitochondrial removal do not require fold merging.
- Input fold data is not in Vienna or CT format, since HybFoldIter pairing is defined
  for HybFile with ViennaFile/CTFile iterators.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0097
tools:
- name: Hybkit
  role: Provides the HybFoldIter file-iterator, HybFile, ViennaFile, FoldRecord, and
    HybRecord classes used to pair and merge fold data into hybrid records
  repo: http://www.github.com/RenneLab/hybkit
- name: Python 3.8
  role: Runtime for the hybkit API used to implement the merge workflow
- name: hyb_check
  role: hybkit command-line utility for post-merge integrity checking of the merged
    hyb/fold output
  repo: http://www.github.com/RenneLab/hybkit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:17da7fbf1c8a4ede570209f28c935946c7c659953ff6c0ef6a1494ccb82c5aaa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python
  - Python 3.8
  - hyb_check
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# vienna-fold-annotation-merging

## Summary

Merges predicted secondary-structure annotations (Vienna dot-bracket strings and Gibbs free-energy values) from fold files into hyb-format hybrid records using hybkit's HybFoldIter concurrent file iterator, with sequence-mismatch checking between paired hyb and fold records.

## When to use

Use when a qCLASH/CLASH analysis requires each hyb-format hybrid record to carry its predicted folding pattern (dot-bracket) and energy from a paired Vienna or CT file, e.g. before Fold Analysis of miRNA folding patterns.

## When NOT to use

- The hyb file has no paired Vienna/CT fold file, so records cannot be matched by identifier and no fold annotation can be merged.
- The goal is filtering rather than annotation — hyb_filter operations like 'any_seg_contains kshv' or duplicate/ribosomal/mitochondrial removal do not require fold merging.
- Input fold data is not in Vienna or CT format, since HybFoldIter pairing is defined for HybFile with ViennaFile/CTFile iterators.

## Inputs

- hyb-format hybrid record file (.hyb), e.g. GSE101978-derived qCLASH data
- Paired Vienna dot-bracket fold file (.vienna) with predicted structure and Gibbs free energy per record
- Optionally a CT format fold file (hybkit.CTFile is supported as a fold source)

## Outputs

- hyb_fold_merged.hyb — tab-delimited hyb file where each HybRecord carries fold information (dot-bracket string, Gibbs free energy) and any hyb/fold sequence-mismatch flags
- Mismatch-flag report on records whose hyb and fold sequences disagree

## How to apply

Open the hyb file (e.g. GSE101978-derived qCLASH data) with hybkit.HybFile.open and the paired Vienna dot-bracket file with hybkit.ViennaFile using Python 3.8+. Iterate concurrently with hybkit.HybFoldIter, which pairs each HybRecord with its FoldRecord by record identifier, then call the merge method to copy the FoldRecord's dot-bracket fold pattern and predicted Gibbs free-energy value into the HybRecord. Keep mismatch checking enabled (default) so that hyb-record read sequences are compared against fold-record sequences and divergent records are flagged; disable only by setting mismatches to '-1' when the sequences are known to differ legitimately. Write the fold-annotated, mismatch-flagged records to a tab-delimited hyb output such as hyb_fold_merged.hyb, then validate with hybkit's record-parsing/integrity checks (hyb_check-style) before downstream Fold Analysis.

## Related tools

- **Hybkit** (Provides the HybFoldIter file-iterator, HybFile, ViennaFile, FoldRecord, and HybRecord classes used to pair and merge fold data into hybrid records) — http://www.github.com/RenneLab/hybkit
- **Python 3.8** (Runtime for the hybkit API used to implement the merge workflow)
- **hyb_check** (hybkit command-line utility for post-merge integrity checking of the merged hyb/fold output) — http://www.github.com/RenneLab/hybkit

## Examples

```
with hybkit.HybFile.open(in_hyb, 'r') as hyb_file, hybkit.ViennaFile.open(in_vienna, 'r') as vienna_file:
    for hyb_record, fold_record in hybkit.HybFoldIter(hyb_file, vienna_file):
        hyb_record.merge_fold_record(fold_record)
```

## Evaluation signals

- hyb_fold_merged.hyb exists and parses without error under hybkit's record-parsing checks (hyb_check-style integrity checking)
- Every output HybRecord carries fold information consistent with the dot-bracket string and Gibbs free-energy value in its paired FoldRecord
- Record pairing by identifier is complete — HybFoldIter consumed the hyb and Vienna files in lockstep with no orphaned records
- Records where the HybRecord read sequence differs from the FoldRecord sequence are flagged as mismatches (unless mismatches='-1' was set)
- Merged output remains valid tab-delimited hyb format suitable for downstream Fold Analysis

## Limitations

- Merging requires one-to-one pairing by record identifier; hyb and fold files must be ordered/identifiable in matched pairs or HybFoldIter cannot associate records.
- Sequence-mismatch checking must be explicitly disabled (mismatches='-1') if hyb and fold sequences legitimately differ; otherwise divergent records are flagged, which may not be desired.
- Fold information is predicted secondary structure, so Gibbs free energies and dot-bracket patterns are model-dependent rather than experimental observations.

## Evidence

- [readme] HybFoldIter merge of Vienna fold data: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [abstract] Vienna format incorporation: "The Hybkit API includes a suite of python objects for developing custom analyses of hybrid data as well as miRNA-specific analysis methods, built-in plotting of analysis results, and incorporation of"
- [methods] hyb_check integrity checking: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] fold analysis purpose: "Fold Analysis                 Analyze and plot predicted miRNA folding patterns in
                                      miRNA-containing hybrids"
- [methods] hyb_filter with fold files: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
