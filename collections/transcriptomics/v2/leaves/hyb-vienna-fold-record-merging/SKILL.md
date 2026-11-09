---
name: hyb-vienna-fold-record-merging
description: Use when a qCLASH/CLASH dataset consists of a .hyb file with a paired
  Vienna-format predicted-secondary-structure file, and per-record fold patterns or
  predicted binding energies of miRNA-containing hybrids must be computed and plotted.
when_to_use_negative:
- Input is a single .hyb file with no paired Vienna/CT predicted-secondary-structure
  file — FoldRecord merging and fold analysis cannot be performed
- The analysis goal is only type/miRNA quantification or target counting without any
  secondary-structure component — plain HybFile iteration without HybFoldIter suffices
- Input records are raw sequencing reads rather than processed hyb-format hybrid records
edam_operation: http://edamontology.org/operation_2464
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0092
tools:
- name: Hybkit
  role: Provides the HybRecord, FoldRecord, ViennaFile, and HybFoldIter classes for
    merging Vienna fold data into hyb records, plus hyb_check/hyb_filter/hyb_analyze
    utilities and built-in matplotlib plotting for fold analysis
  repo: https://github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API (Python3.8+) used to drive HybFoldIter iteration
    and custom analyses
  repo: https://github.com/RenneLab/hybkit
- name: matplotlib
  role: Plotting backend (>= 3.7.1) behind hybkit's built-in analysis plots of folding
    patterns and energy distributions
  repo: https://matplotlib.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:f2e23e0a4ee7d47202dd851a8d4e5648c0eb3dc645d9f9e03291de83fe42c96d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RenneLab/hybkit
  tools:
  - Hybkit
  - Python3
  - matplotlib
  edam_operation: http://edamontology.org/operation_2464
  edam_topics:
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hyb-vienna-fold-record-merging

## Summary

Merges predicted secondary-structure (fold) data from Vienna-format files into hyb-format hybrid RNA records using hybkit's FoldRecord class and the HybFoldIter concurrent iterator, enabling fold and binding-energy analysis of miRNA/target hybrids from CLASH/qCLASH experiments.

## When to use

Use when a qCLASH/CLASH dataset consists of a .hyb file with a paired Vienna-format predicted-secondary-structure file, and per-record fold patterns or predicted binding energies of miRNA-containing hybrids must be computed and plotted.

## When NOT to use

- Input is a single .hyb file with no paired Vienna/CT predicted-secondary-structure file — FoldRecord merging and fold analysis cannot be performed
- The analysis goal is only type/miRNA quantification or target counting without any secondary-structure component — plain HybFile iteration without HybFoldIter suffices
- Input records are raw sequencing reads rather than processed hyb-format hybrid records

## Inputs

- hyb-format chimeric (hybrid) RNA file (.hyb) from CLASH/qCLASH
- Paired Vienna-format predicted secondary-structure file (.vienna) aligned record-by-record with the hyb file
- miRNA reference annotations (e.g. MirBase mature miRNA identifiers such as MIMAT0000078_miR-23a) used by eval_mirna()

## Outputs

- Per-record merged HybRecord + FoldRecord pairs with fold/energy annotations
- Fold analysis results as a CSV table of predicted binding energies and per-nucleotide intermolecular pairing patterns
- PNG figures of aggregated per-nucleotide folding patterns and energy distributions produced by hybkit.plot

## How to apply

First run quality control on the hyb file and its paired Vienna file with hyb_check, then remove artifactual ribosomal- and mitochondrial-RNA hybrids with hyb_filter. Next, use the hybkit Python API to open the hyb file with hybkit.HybFile.open and iterate it concurrently with the Vienna file via hybkit.HybFoldIter, so each HybRecord is paired with a FoldRecord parsed from the corresponding Vienna record; fold records can also be supplied to command-line tools with the -f flag (e.g. hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna). Classify segments and miRNA identity with HybRecord.eval_types() and eval_mirna(), retaining miRNA-containing hybrids. Finally run hyb_analyze in fold (or energy) mode, or a custom API analysis, to compute predicted binding energy and per-nucleotide intermolecular pairing, and plot with hybkit's built-in matplotlib plotting (hybkit.plot), exporting results as CSV and PNG.

## Related tools

- **Hybkit** (Provides the HybRecord, FoldRecord, ViennaFile, and HybFoldIter classes for merging Vienna fold data into hyb records, plus hyb_check/hyb_filter/hyb_analyze utilities and built-in matplotlib plotting for fold analysis) — https://github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API (Python3.8+) used to drive HybFoldIter iteration and custom analyses) — https://github.com/RenneLab/hybkit
- **matplotlib** (Plotting backend (>= 3.7.1) behind hybkit's built-in analysis plots of folding patterns and energy distributions) — https://matplotlib.org/

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- hyb_check reports no errors for both the .hyb file and its paired Vienna file before merging
- After HybFoldIter iteration, the number of HybRecord/FoldRecord pairs matches the (filtered) line count of both source files, with record identifiers aligned between hyb and Vienna entries
- Records retained for fold analysis all have eval_types() and eval_mirna() annotations assigning a miRNA segment, and artifactual ribosomal/mitochondrial hybrids are absent
- Fold analysis output CSV contains predicted binding energy values and per-nucleotide pairing columns for every miRNA-containing hybrid
- hybkit.plot produces PNG figures of folding patterns and energy distributions consistent with the CSV table (matching sample counts)

## Limitations

- Fold data requires a Vienna (or CT) file paired record-by-record with the hyb file; missing or misaligned fold records break the merge
- Only miRNA-containing hybrids (as classified by eval_types()/eval_mirna()) are meaningful for miRNA fold-pattern analysis
- hybkit requires Python3.8+, matplotlib >= 3.7.1, BioPython >= 1.79, and typing_extensions >= 4.8.0

## Evidence

- [readme] The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes"
- [abstract] Vienna-format fold integration: "built-in plotting of analysis results, and incorporation of predicted miRNA/target interactions in Vienna format"
- [methods] QC and artifact removal before fold analysis: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] Fold analysis definition: "Analyze and plot predicted miRNA folding patterns in miRNA-containing hybrids"
