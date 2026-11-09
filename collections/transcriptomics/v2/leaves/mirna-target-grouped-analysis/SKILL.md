---
name: mirna-target-grouped-analysis
description: Use when you have miRNA/target hybrid data in hyb format (optionally
  with fold/vienna files) from multiple experimental replicates and need to quantify
  targets of a defined miRNA set across pooled replicates with tabular and plotted
  summaries.
when_to_use_negative:
- You need a Type/miRNA Analysis (quantifying sequence types and miRNA types in a
  single hyb file) rather than targets of a miRNA set across pooled replicates.
- The analysis is a Target Analysis of a single experimental replicate rather than
  pooled experimental replicates.
- The goal is analyzing predicted miRNA folding patterns in miRNA-containing hybrids
  (Fold Analysis), not miRNA target quantification.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Hybkit
  role: Provides the hyb_analyze command-line utility and hybkit.analysis.Analysis
    API for grouped target analysis of pooled replicates, plus hyb_check/hyb_filter
    for QC and filtering and hybkit.plot for plotting
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API (Python3.8+), used for the API-script implementation
    of the grouped target analysis
- name: matplotlib
  role: Plotting dependency (>= 3.7.1) used by hybkit.plot for the grouped target-analysis
    plots
- name: BioPython
  role: Dependency (>= 1.79) of the hybkit package; the hybkit API was inspired by
    the BioPython project
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:7fd95e3f3dbe75df5ef1c7534037d1eaf9fe8b7ab97a302daf19a34d1dbeb505
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Hybkit
  - Python3
  - matplotlib
  - typing_extensions
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mirna-target-grouped-analysis

## Summary

Hybkit's grouped target analysis quantifies and plots targets of a set of miRNAs (e.g., all KSHV miRNAs) from pooled qCLASH experimental replicates in hyb file format, producing summary CSV tables and plots. It is one of hybkit's documented example analyses, implemented via both the Python3 API and the hyb_analyze command-line utility.

## When to use

Use when you have miRNA/target hybrid data in hyb format (optionally with fold/vienna files) from multiple experimental replicates and need to quantify targets of a defined miRNA set across pooled replicates with tabular and plotted summaries.

## When NOT to use

- You need a Type/miRNA Analysis (quantifying sequence types and miRNA types in a single hyb file) rather than targets of a miRNA set across pooled replicates.
- The analysis is a Target Analysis of a single experimental replicate rather than pooled experimental replicates.
- The goal is analyzing predicted miRNA folding patterns in miRNA-containing hybrids (Fold Analysis), not miRNA target quantification.

## Inputs

- hyb-format chimeric RNA files from miRNA/target ligation experiments (e.g., qCLASH) for multiple experimental replicates
- associated fold files (vienna or CT format) for the hybrids
- a defined set of miRNAs to analyze (e.g., all KSHV miRNAs)

## Outputs

- summary CSV tables of hybrid-target counts and miRNA target distributions across pooled replicates
- plot files (e.g., PNG plots via hybkit.plot / matplotlib) of grouped target analysis results

## How to apply

Load the replicate hyb files and associated fold files (e.g., the six qCLASH replicates of KSHV data from GEO accession GSE101978) using hybkit's HybFile/FoldFile iterators, first running quality control with hyb_check to verify data integrity. Remove artifactual ribosomal- and mitochondrial-RNA hybrids with hyb_filter (segment-type filter), then annotate each remaining record with HybRecord.eval_types() and eval_mirna() to assign segment types and miRNA identities. Pool the filtered, evaluated records across all replicates and run the grouped target analysis (hyb_analyze with target analysis mode, or hybkit.analysis.Analysis via the API) to quantify targets of the miRNA set. Generate plots with the hybkit.plot module and write hybrid-target count and miRNA target distribution tables to CSV; the workflow exists as parallel Python API scripts and shell scripts so either interface can be used.

## Related tools

- **Hybkit** (Provides the hyb_analyze command-line utility and hybkit.analysis.Analysis API for grouped target analysis of pooled replicates, plus hyb_check/hyb_filter for QC and filtering and hybkit.plot for plotting) — github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API (Python3.8+), used for the API-script implementation of the grouped target analysis)
- **matplotlib** (Plotting dependency (>= 3.7.1) used by hybkit.plot for the grouped target-analysis plots)
- **BioPython** (Dependency (>= 1.79) of the hybkit package; the hybkit API was inspired by the BioPython project)

## Examples

```
hyb_analyze -i pooled_hybs.hyb -f pooled_folds.vienna --analysis target
```

## Evaluation signals

- The produced grouped-target analysis CSV tables and PNG plots exist and are readable.
- Summary counts match the values reported by the example analysis run in the Hybkit documentation for the GSE101978 qCLASH dataset.
- hyb_check parsing of each hyb and fold file completes without data-integrity errors before analysis.
- Artifactual ribosomal- and mitochondrial-RNA hybrids are absent from the pooled records prior to target quantification.
- Each replicate record has segment types and miRNA identities assigned via eval_types() and eval_mirna() before pooling.

## Limitations

- The command-line scripts have been tested on the example qCLASH dataset; behavior on hyb files from other ligation protocols may require custom filter/evaluation criteria (hyb_eval supports custom criteria).
- Dependencies have minimum versions (Python3.8+, matplotlib >= 3.7.1, BioPython >= 1.79, typing_extensions >= 4.8.0) that must be installed.
- No changelog was found for the hybkit source repository, so tracking feature changes may require release badges/history.

## Evidence

- [methods] Grouped Target Analysis — Analyze and plot targets of a set of miRNAs from pooled experimental replicates: [evidence span withheld — anchor status: not_found]
- [methods] QC and filtering steps precede analysis: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [readme] Example analyses are provided as command-line scripts and Python API implementations: "Example analyses for publicly available qCLASH hybrid sequence data implemented in each of the command-line scripts and hybkit Python API."
- [methods] The example dataset is qCLASH data with fold information: "data acquired from the NCBI Gene Expression Omnibus (GEO) accession `GSE101978 <https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE101978>`_"
- [readme] API record evaluation methods used for annotation: "# Analyze each record to assign segment types
hyb_record.eval_types()"
- [methods] Filtering of artifactual hybrids is part of the workflow: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
