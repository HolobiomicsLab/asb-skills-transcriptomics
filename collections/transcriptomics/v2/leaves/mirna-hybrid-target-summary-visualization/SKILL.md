---
name: mirna-hybrid-target-summary-visualization
description: Use when you have multiple hyb (and associated fold/vienna) files from
  replicate miRNA/target ligation experiments (e.g. qCLASH) and need to analyze and
  plot targets of a defined set of miRNAs across pooled replicates.
when_to_use_negative:
- You need to analyze targets from a single experimental replicate rather than pooled
  replicates — hybkit provides a separate single-replicate 'Target Analysis' example
- You need to quantify sequence types or miRNA types rather than miRNA targets — use
  hybkit's Type/miRNA Analysis
- You need to analyze predicted miRNA folding patterns in miRNA-containing hybrids
  — use hybkit's Fold Analysis
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: Hybkit
  role: Provides the hyb_check, hyb_filter, hyb_analyze command-line utilities and
    the hybkit Python API (HybFile, FoldFile, HybRecord, analysis, plot modules) used
    to perform the grouped target analysis and plotting
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API implementation of the grouped target analysis (Python3.8+
    required)
- name: matplotlib
  role: Plotting dependency (>= 3.7.1) backing hybkit's built-in plotting of analysis
    results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:7d9beeb172ec7fd8c8edfadc956499513a456972926ee68f248e762a4f7f044f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Hybkit
  - Python3
  - matplotlib
  - typing_extensions
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# miRNA-hybrid-target-summary-visualization

## Summary

Running hybkit's grouped target analysis to quantify and plot targets of a set of miRNAs (e.g. all KSHV miRNAs) across pooled experimental replicates of qCLASH hybrid data in 'hyb' format, via both the Python API and command-line scripts.

## When to use

Use when you have multiple hyb (and associated fold/vienna) files from replicate miRNA/target ligation experiments (e.g. qCLASH) and need to analyze and plot targets of a defined set of miRNAs across pooled replicates.

## When NOT to use

- You need to analyze targets from a single experimental replicate rather than pooled replicates — hybkit provides a separate single-replicate 'Target Analysis' example
- You need to quantify sequence types or miRNA types rather than miRNA targets — use hybkit's Type/miRNA Analysis
- You need to analyze predicted miRNA folding patterns in miRNA-containing hybrids — use hybkit's Fold Analysis

## Inputs

- Six qCLASH replicate .hyb files (e.g. KSHV miRNA hybrid data from GEO GSE101978)
- Associated fold files in vienna or CT format paired with each hyb file
- A defined set of miRNAs to analyze (e.g. all KSHV miRNAs)

## Outputs

- Grouped target-analysis CSV summary tables (hybrid-target counts and miRNA target distributions)
- PNG plots generated with the hybkit.plot module (tabular and graph form results)

## How to apply

Load each replicate hyb file (and associated fold files, e.g. from GEO accession GSE101978) with hybkit's HybFile/FoldFile iterators and run quality control via hyb_check to verify data integrity. Remove artifactual ribosomal- and mitochondrial-RNA hybrids with hyb_filter's segment-type filter, then evaluate each remaining record with HybRecord.eval_types() and eval_mirna() to assign segment types and miRNA identities. Pool the filtered, evaluated records from all replicates and run the grouped target analysis (hyb_analyze / hybkit.analysis.Analysis) to quantify targets of the set of miRNAs across the pooled replicates. Generate plots with the hybkit.plot module and write summary tables (hybrid-target counts and miRNA target distributions) to CSV; each of hybkit's example analyses is implemented as both a command-line shell script and a Python API script.

## Related tools

- **Hybkit** (Provides the hyb_check, hyb_filter, hyb_analyze command-line utilities and the hybkit Python API (HybFile, FoldFile, HybRecord, analysis, plot modules) used to perform the grouped target analysis and plotting) — github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API implementation of the grouped target analysis (Python3.8+ required))
- **matplotlib** (Plotting dependency (>= 3.7.1) backing hybkit's built-in plotting of analysis results)

## Examples

```
import hybkit; with hybkit.HybFile.open(in_file, 'r') as hyb_file:
    for hyb_record in hyb_file:
        hyb_record.eval_types()
        hyb_record.eval_mirna()
```

## Evaluation signals

- Produced grouped-target analysis CSV tables and PNG plots exist and are readable
- Summary counts match the values reported by the example analysis run in the Hybkit documentation for the GSE101978 qCLASH dataset
- hyb_check reports no errors when parsing each hyb and fold file (data integrity verified before analysis)
- Artifactual ribosomal- and mitochondrial-RNA hybrids are absent from the filtered records fed into the grouped analysis
- Each record entering the analysis has segment types and miRNA identities assigned via eval_types() and eval_mirna()

## Limitations

- The grouped target analysis pools replicates; it analyzes targets of a set of miRNAs from pooled experimental replicates, not per-replicate statistics
- hybkit requires Python3.8+ with matplotlib >= 3.7.1, BioPython >= 1.79, and typing_extensions >= 4.8.0
- Hybrid data must be in the 'hyb' file format (with optional associated vienna or CT fold files) as defined with the Hyb software package

## Evidence

- [methods] Grouped Target Analysis: Analyze and plot targets of a set of miRNAs from pooled experimental replicates: "Grouped Target Analysis       Analyze and plot targets of a set of miRNAs from
                                      pooled experimental replicates"
- [methods] Each analysis performs quality control by checking data integrity then removing artifactual ribosomal- and mitochondrial-RNA hybrids: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] README describes example analyses implemented in both command-line scripts and the hybkit Python API: "Example analyses for publicly available qCLASH hybrid
       sequence data implemented in each of the command-line scripts and hybkit Python API."
- [readme] Analyses provide results in both tabular and graph form: "These analyses provide analysis results in both tabular and graph form."
- [abstract] hybkit enables customizable evaluation and annotation of hybrid characteristics from miRNA/target ligation experiments: "analysis of hybrid sequence data in the "hyb" file format to enable customizable evaluation and annotation of hybrid characteristics"
