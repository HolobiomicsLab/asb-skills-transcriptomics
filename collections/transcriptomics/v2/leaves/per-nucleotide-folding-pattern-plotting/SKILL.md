---
name: per-nucleotide-folding-pattern-plotting
description: Use when analyzing CLASH/qCLASH hybrid RNA data in 'hyb' format that
  has a paired Vienna (or CT) predicted-structure file, and you need predicted miRNA/target
  folding patterns, per-nucleotide pairing, or binding-energy distributions.
when_to_use_negative:
- When the hybrid data has no associated Vienna or CT predicted-structure file — FoldRecord
  merging and fold analysis require predicted secondary-structure input
- When the goal is only hybrid counting or target enrichment without structure/energy
  — the simpler Type/miRNA or Target analyses of hybkit are the appropriate choice
- When structures are experimentally determined rather than predicted — this skill
  analyzes Vienna-format predicted folding patterns, not measured structures
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: hybkit
  role: Provides HybFoldIter, FoldRecord, ViennaFile, HybRecord (eval_types/eval_mirna),
    hyb_check, hyb_filter, hyb_analyze fold mode, and built-in plotting used to merge
    fold data and compute/plot per-nucleotide patterns and energies
  repo: http://www.github.com/RenneLab/hybkit
- name: matplotlib
  role: Rendering backend for hybkit's built-in plotting of folding-pattern and energy
    figures (>= 3.7.1 dependency)
  repo: https://matplotlib.org/
- name: BioPython
  role: Dependency of the hybkit Python API that hosts the fold-analysis record classes
    (>= 1.79)
  repo: https://biopython.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:62b754d26021b1952ae12ba415ebc827d3bf39c5baa8dcf10514c711ac37cc1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Python3
  - hybkit
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Per-nucleotide folding pattern plotting for miRNA-target hybrids

## Summary

A hybkit-based fold-analysis skill that merges Vienna-format secondary-structure data with miRNA-containing hybrids from qCLASH/CLASH 'hyb' files, computes predicted binding energy and per-nucleotide intermolecular pairing, and plots aggregated folding patterns and energy distributions with hybkit's built-in matplotlib plotting.

## When to use

Use when analyzing CLASH/qCLASH hybrid RNA data in 'hyb' format that has a paired Vienna (or CT) predicted-structure file, and you need predicted miRNA/target folding patterns, per-nucleotide pairing, or binding-energy distributions.

## When NOT to use

- When the hybrid data has no associated Vienna or CT predicted-structure file — FoldRecord merging and fold analysis require predicted secondary-structure input
- When the goal is only hybrid counting or target enrichment without structure/energy — the simpler Type/miRNA or Target analyses of hybkit are the appropriate choice
- When structures are experimentally determined rather than predicted — this skill analyzes Vienna-format predicted folding patterns, not measured structures

## Inputs

- qCLASH/CLASH hybrid data in the 'hyb' file format (e.g. GEO GSE101978-derived hybrids)
- Paired predicted secondary-structure file in Vienna format (or CT format) with one record per hybrid
- miRNA reference identifiers (e.g. MirBase, e.g. MIMAT0000078_MirBase_miR-23a) for miRNA classification via eval_mirna()
- Filter keyword(s) for segment-identifier-based filtering (e.g. 'kshv')

## Outputs

- Per-nucleotide intermolecular folding-pattern plots (PNG) for miRNA-containing hybrids
- Predicted binding-energy distribution plots (PNG)
- Tabular fold-analysis results as a CSV table
- Filtered hyb and Vienna file subsets (from hyb_filter) for reproducible record alignment

## How to apply

First run hyb_check on the hyb file and its paired Vienna fold file to validate integrity, then use hyb_filter to remove artifactual ribosomal- and mitochondrial-RNA hybrids (optionally filtering by identifier strings, e.g. 'kshv', with --filter any_seg_contains). Iterate the hyb and Vienna files in lockstep using hybkit's HybFoldIter so each HybRecord is paired with a FoldRecord built from the corresponding Vienna record; call HybRecord.eval_types() and HybRecord.eval_mirna() to classify segment types and miRNA identity, retaining only miRNA-containing hybrids. Run the hybkit fold analysis (hyb_analyze in fold mode, or the analysis module on FoldRecord data) to compute predicted binding energy and per-nucleotide intermolecular pairing between the miRNA and its target for every retained hybrid. Finally, render aggregated per-nucleotide folding patterns and energy distributions via hybkit's built-in matplotlib plotting (hybkit.plot), saving figures as PNG and the tabular results as CSV. Correctness hinges on record alignment between hyb and Vienna files: every plotted record must carry a merged FoldRecord, and the outputs must include both a CSV table and graphical (PNG) representations as hybkit analyses return 'results in both tabular and graph form'.

## Related tools

- **hybkit** (Provides HybFoldIter, FoldRecord, ViennaFile, HybRecord (eval_types/eval_mirna), hyb_check, hyb_filter, hyb_analyze fold mode, and built-in plotting used to merge fold data and compute/plot per-nucleotide patterns and energies) — http://www.github.com/RenneLab/hybkit
- **matplotlib** (Rendering backend for hybkit's built-in plotting of folding-pattern and energy figures (>= 3.7.1 dependency)) — https://matplotlib.org/
- **BioPython** (Dependency of the hybkit Python API that hosts the fold-analysis record classes (>= 1.79)) — https://biopython.org/

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv  # then: hyb_analyze fold -i filtered.hyb -f filtered.vienna
```

## Evaluation signals

- hyb_check reports no integrity errors on both the hyb file and its paired Vienna fold file before analysis
- Each analyzed HybRecord has a corresponding FoldRecord merged via HybFoldIter — record counts in hyb and Vienna inputs match after filtering
- Only miRNA-containing hybrids (eval_types()/eval_mirna() positive, ribosomal/mitochondrial hybrids removed) contribute to the plots
- Outputs include both a CSV table and PNG figures, consistent with hybkit analyses providing 'results in both tabular and graph form'
- Per-nucleotide pairing plots show positions consistent with the Vienna dot-bracket pairing of the merged FoldRecords

## Limitations

- Fold analysis depends on predicted structures in Vienna/CT format; accuracy is bounded by the predictor that generated the fold file
- Artifactual ribosomal- and mitochondrial-RNA hybrids must be removed beforehand or they will contaminate the fold-analysis results
- hyb_filter operations on hyb files require supplying the paired Vienna file (-f) to keep the two files in sync; filtering one without the other breaks record alignment

## Evidence

- [readme] Fold Analysis — Analyze and plot predicted miRNA folding patterns in miRNA-containing hybrids: "Fold Analysis                 Analyze and plot predicted miRNA folding patterns in
                                      miRNA-containing hybrids"
- [readme] Hybkit API fold classes: "a class for fold records (FoldRecord), and file-iterator classes
    (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [methods] Quality control and artifact filtering precede analysis: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] Integrated fold support in hybkit: "Integrated analysis of predicted secondary structure (fold) information for
       the API and command-line utilities."
- [readme] Analyses return tabular and graphical outputs: "These analyses provide analysis results in both tabular and graph form."
