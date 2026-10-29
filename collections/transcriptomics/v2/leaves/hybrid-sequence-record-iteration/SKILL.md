---
name: hybrid-sequence-record-iteration
description: Use when qCLASH/CLASH hybrid records in hyb format must be combined record-by-record
  with predicted secondary-structure data held in a paired Vienna dot-bracket or CT
  file — e.g. before fold analysis or any record-level fold-aware filtering.
when_to_use_negative:
- No paired Vienna/CT fold file exists for the hyb file — HybFoldIter depends on one-to-one
  record pairing by identifier, so fold-merge iteration cannot be applied
- Input is raw sequencing data (e.g. FASTQ) rather than already-formed hyb-format
  chimeric records
- Only a standalone secondary-structure prediction is needed without hyb records —
  use a fold predictor directly instead of hybkit's paired iteration
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: Hybkit
  role: Provides HybFile, ViennaFile, CTFile, FoldRecord, HybRecord and the HybFoldIter
    file-iterator class used to merge fold data into hybrid records; also provides
    hyb_check for integrity validation of the merged output
  repo: http://www.github.com/RenneLab/hybkit
- name: Python 3.8
  role: Runtime for the hybkit Python3 API used to script the paired iteration and
    record merging
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:e1665a9d50f206dffa78cad5397c472974b5b76518606a91081f9bfe954cf924
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python
  - Python 3.8
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hybrid-sequence-record-iteration

## Summary

Concurrent, record-by-record iteration over paired hyb-format chimeric RNA files and Vienna dot-bracket (or CT) secondary-structure files using the hybkit Python API's HybFoldIter file-iterator class, merging FoldRecord fold predictions into the corresponding HybRecord objects.

## When to use

Use when qCLASH/CLASH hybrid records in hyb format must be combined record-by-record with predicted secondary-structure data held in a paired Vienna dot-bracket or CT file — e.g. before fold analysis or any record-level fold-aware filtering.

## When NOT to use

- No paired Vienna/CT fold file exists for the hyb file — HybFoldIter depends on one-to-one record pairing by identifier, so fold-merge iteration cannot be applied
- Input is raw sequencing data (e.g. FASTQ) rather than already-formed hyb-format chimeric records
- Only a standalone secondary-structure prediction is needed without hyb records — use a fold predictor directly instead of hybkit's paired iteration

## Inputs

- hyb-format hybrid RNA file (e.g. qCLASH-derived, GSE101978-style chimeric reads)
- Paired Vienna dot-bracket file of predicted secondary structures (one FoldRecord per HybRecord, matched by record identifier)
- Optionally, a CT-format file (hybkit also provides CTFile for this purpose)

## Outputs

- Fold-annotated HybRecord objects carrying the dot-bracket folding pattern and predicted Gibbs free energy from the paired FoldRecord
- A merged tab-delimited hyb file (e.g. hyb_fold_merged.hyb) including per-record mismatch flags

## How to apply

Open the hyb file with hybkit.HybFile.open(...,'r') and the paired Vienna file with hybkit.ViennaFile (or CTFile for CT format), then iterate with the hybkit.HybFoldIter file-iterator class, which pairs each HybRecord with its matching FoldRecord by record identifier. At each iteration step, merge the FoldRecord's predicted folding pattern (dot-bracket string) and predicted Gibbs free-energy value into the corresponding HybRecord so downstream scripts can evaluate fold-aware properties. Compare each HybRecord's read sequence against its FoldRecord's sequence — mismatch checking is enabled by default and flags hyb/fold sequence mismatches; it can be disabled by setting mismatches to '-1', but leaving it on catches pairing errors from misordered or inconsistent files. Write the fold-annotated records to a tab-delimited hyb file (e.g. hyb_fold_merged.hyb) and validate it by re-parsing with hybkit's record-parsing checks (hyb_check-style integrity checking), confirming records carry fold information and mismatch flags consistent with the paired inputs.

## Related tools

- **Hybkit** (Provides HybFile, ViennaFile, CTFile, FoldRecord, HybRecord and the HybFoldIter file-iterator class used to merge fold data into hybrid records; also provides hyb_check for integrity validation of the merged output) — http://www.github.com/RenneLab/hybkit
- **Python 3.8** (Runtime for the hybkit Python3 API used to script the paired iteration and record merging)

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- The merged hyb file (e.g. hyb_fold_merged.hyb) exists and parses without error under hybkit's record-parsing checks (hyb_check-style integrity checking)
- Record count of the merged output equals the record count of the input hyb file (one-to-one pairing, no records dropped or duplicated)
- Each output record carries fold information — dot-bracket string and predicted Gibbs free-energy value — identical to the FoldRecord matched by its record identifier
- Hyb/fold sequence mismatches are flagged; with mismatch checking left at default, any record whose read sequence differs from its FoldRecord sequence is flagged rather than silently merged
- Setting mismatches='-1' reproduces the merge without mismatch flags, confirming the flagging behavior is parameter-driven

## Limitations

- Mismatch checking must be explicitly disabled via mismatches='-1'; leaving it on will flag records where hyb and fold sequences disagree, which indicates mispaired or inconsistent input files rather than a hybkit error
- Pairing relies on matching record identifiers between the hyb and Vienna/CT files — misordered or differently identified files will produce mismatch flags or failed pairing
- Fold information is predicted secondary structure (Vienna dot-bracket / Gibbs free energy), not experimentally determined structure; hybkit handles 'vienna' or 'CT' files only, as defined with the Hyb software package

## Evidence

- [readme] The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter).: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [readme] Integrated analysis of predicted secondary structure (fold) information for the API and command-line utilities.: "Integrated analysis of predicted secondary structure (fold) information for the API and command-line utilities."
- [abstract] The Hybkit API includes a suite of python objects for developing custom analyses of hybrid data as well as miRNA-specific analysis methods, built-in plotting of analysis results, and incorporation of: "The Hybkit API includes a suite of python objects for developing custom analyses of hybrid data as well as miRNA-specific analysis methods, built-in plotting of analysis results, and incorporation of"
- [methods] Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`): "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [methods] Welcome to *hybkit*, a toolkit for analysis of hyb-format chimeric (hybrid) RNA sequence data defined with the Hyb software package: "Welcome to *hybkit*, a toolkit for analysis of hyb-format chimeric (hybrid) RNA sequence data defined with the Hyb software package"
