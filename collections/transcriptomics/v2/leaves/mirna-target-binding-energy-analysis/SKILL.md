---
name: mirna-target-binding-energy-analysis
description: Use when a qCLASH/CLASH hybrid-sequence dataset has paired hyb and Vienna
  (or CT) fold files and the goal is to quantify predicted miRNA-target binding energy
  and per-nucleotide intermolecular pairing patterns in miRNA-containing hybrids.
when_to_use_negative:
- The hyb file has no associated Vienna/CT fold file — fold analysis requires predicted
  secondary-structure records to merge via HybFoldIter
- The goal is only to quantify sequence types or miRNA/target counts without fold
  information — use hybkit's type/miRNA or target analyses instead
- Input is raw sequencing reads rather than already-processed hyb-format hybrid records
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: Hybkit
  role: Provides HybRecord, FoldRecord, HybFile, ViennaFile and HybFoldIter classes
    plus the hyb_check/hyb_filter/hyb_analyze command-line utilities and fold analysis
    with built-in plotting
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API used to iterate over hyb/Vienna records and orchestrate
    the fold analysis
- name: matplotlib
  role: Backend for hybkit's built-in plotting of folding patterns and energy distributions
    (>= 3.7.1)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:fb7d9f03cb44565060ce96ce90b9846f2ea36990236dbf9d0bb821c0d531d363
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python3
  - matplotlib
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mirna-target-binding-energy-analysis

## Summary

Reproduces the qCLASH fold analysis workflow of the Hybkit toolkit: pairing each hybrid ('hyb') record with its Vienna-format predicted secondary structure (FoldRecord) to compute predicted miRNA/target binding energy and per-nucleotide intermolecular folding patterns, then plotting and tabulating the results.

## When to use

Use when a qCLASH/CLASH hybrid-sequence dataset has paired hyb and Vienna (or CT) fold files and the goal is to quantify predicted miRNA-target binding energy and per-nucleotide intermolecular pairing patterns in miRNA-containing hybrids.

## When NOT to use

- The hyb file has no associated Vienna/CT fold file — fold analysis requires predicted secondary-structure records to merge via HybFoldIter
- The goal is only to quantify sequence types or miRNA/target counts without fold information — use hybkit's type/miRNA or target analyses instead
- Input is raw sequencing reads rather than already-processed hyb-format hybrid records

## Inputs

- hyb-format hybrid sequence file (.hyb) from a qCLASH/CLASH experiment (e.g. GEO GSE101978 data)
- Paired Vienna-format (or CT) predicted secondary-structure file aligned record-by-record with the hyb file

## Outputs

- PNG figures of aggregated per-nucleotide miRNA-target folding patterns
- PNG figure(s) of predicted binding-energy distributions
- CSV results table of fold-analysis metrics per hybrid/miRNA-target pair

## How to apply

First run quality control on the hyb file and its paired Vienna fold file with hyb_check, then use hyb_filter to remove artifactual ribosomal- and mitochondrial-RNA hybrids. Iterate over the hyb and Vienna files concurrently with the hybkit HybFoldIter so each HybRecord is merged with its corresponding FoldRecord built from the Vienna record. Classify segment types with HybRecord.eval_types() and identify miRNAs with eval_mirna() to retain only miRNA-containing hybrids. Then run the fold analysis (hyb_analyze in fold mode / the analysis module) to compute predicted binding energy and per-nucleotide intermolecular pairing between miRNA and target for every retained hybrid, and render aggregated folding patterns and energy distributions with hybkit's built-in matplotlib plotting, saving PNG figures plus a CSV table.

## Related tools

- **Hybkit** (Provides HybRecord, FoldRecord, HybFile, ViennaFile and HybFoldIter classes plus the hyb_check/hyb_filter/hyb_analyze command-line utilities and fold analysis with built-in plotting) — github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API used to iterate over hyb/Vienna records and orchestrate the fold analysis)
- **matplotlib** (Backend for hybkit's built-in plotting of folding patterns and energy distributions (>= 3.7.1))

## Examples

```
import hybkit
with hybkit.HybFile.open('my_hyb_file.hyb', 'r') as hf, hybkit.ViennaFile.open('my_hyb_file.vienna', 'r') as vf:
    for hyb_record, fold_record in hybkit.HybFoldIter(hf, vf):
        hyb_record.eval_types(); hyb_record.eval_mirna()
```

## Evaluation signals

- hyb_check reports no integrity errors in the hyb file and its paired Vienna file before analysis
- Every HybRecord retained after eval_types()/eval_mirna() has a merged FoldRecord — no unpaired records from the HybFoldIter
- Output directory contains the expected PNG folding-pattern/energy figures and the CSV results table for the fold analysis
- Per-nucleotide pairing summaries and energy values are present for all miRNA-containing hybrids; ribosomal and mitochondrial hybrids are absent after hyb_filter
- Count of miRNA-containing hybrids analyzed matches the counts from a preceding type/miRNA analysis on the filtered file

## Limitations

- Fold analysis depends on pre-computed Vienna/CT secondary-structure predictions; hybkit does not itself predict structures
- Only miRNA-containing hybrids passing segment-type and miRNA evaluation are analyzed; hybrids without an identified miRNA are excluded
- Artifactual ribosomal- and mitochondrial-RNA hybrids must be removed with hyb_filter before analysis or they will contaminate results

## Evidence

- [readme] hybkit provides fold-aware classes: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [methods] fold analysis description: "Fold Analysis                 Analyze and plot predicted miRNA folding patterns in miRNA-containing hybrids"
- [methods] QC and filter steps: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [methods] filter description: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [readme] integrated fold support: "Integrated analysis of predicted secondary structure (fold) information for the API and command-line utilities."
