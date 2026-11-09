---
name: qclash-count-table-aggregation
description: Use when you have multiple qCLASH hybrid (.hyb) files (e.g. replicates
  from GEO GSE101978) and need per-replicate plus combined counts of hybrid segment
  types and 5p/3p miRNA segments with tabular and plotted output.
when_to_use_negative:
- Input is raw sequencing reads (FASTQ) rather than parsed .hyb-format hybrid records
  — hybkit expects the hyb file format defined by the Hyb software package
- The goal is target-level analysis of a specific miRNA set (use hybkit's Target Analysis
  or Grouped Target Analysis instead of the type/miRNA summary)
- Only a single replicate is available and no cross-replicate aggregation is needed
  — a single hyb_analyze type run suffices without combined tables
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: hybkit
  role: Provides HybFile/HybRecord parsing, hyb_check/hyb_filter/hyb_analyze command-line
    utilities, the analysis.Analysis class for tallying type/miRNA counts, and built-in
    matplotlib plotting
  repo: http://www.github.com/RenneLab/hybkit
- name: matplotlib
  role: Backend for hybkit's built-in plotting of the type pie chart and miRNA bar
    charts (>= 3.7.1)
  repo: https://matplotlib.org/
- name: Python3
  role: Runtime for the hybkit API variant of the workflow (Python 3.8+)
  repo: https://pypi.org/project/hybkit/
- name: BioPython
  role: Dependency of the hybkit API (>= 1.79)
  repo: https://biopython.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:89a294cb22a987a0fe384eea96b85262eb7ec7d9ca73e7b017e2f6622fec60a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Python3
  - hybkit
  - matplotlib
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# qclash-count-table-aggregation

## Summary

Aggregates hybrid sequence-type and miRNA-type counts across multiple qCLASH .hyb replicates into combined CSV tables and pie/bar charts using hybkit's hyb_analyze command-line utility or the hybkit.analysis.Analysis class.

## When to use

Use when you have multiple qCLASH hybrid (.hyb) files (e.g. replicates from GEO GSE101978) and need per-replicate plus combined counts of hybrid segment types and 5p/3p miRNA segments with tabular and plotted output.

## When NOT to use

- Input is raw sequencing reads (FASTQ) rather than parsed .hyb-format hybrid records — hybkit expects the hyb file format defined by the Hyb software package
- The goal is target-level analysis of a specific miRNA set (use hybkit's Target Analysis or Grouped Target Analysis instead of the type/miRNA summary)
- Only a single replicate is available and no cross-replicate aggregation is needed — a single hyb_analyze type run suffices without combined tables

## Inputs

- Three qCLASH hybrid (.hyb) files (replicates from Gay et al. 2018, NCBI GEO accession GSE101978)
- Optionally associated vienna fold files for integrated fold analysis

## Outputs

- Per-replicate hybrid-type count table (CSV)
- Per-replicate miRNA-type count table (CSV)
- Combined type and miRNA count tables aggregated across replicates (CSV)
- Hybrid-type distribution pie chart (matplotlib figure)
- miRNA characteristics bar charts (matplotlib figures)

## How to apply

First run hyb_check on each .hyb file to confirm the 15-16 tab-delimited columns parse cleanly, then use hyb_filter to remove artifactual ribosomal- and mitochondrial-RNA hybrids. On the filtered records, call eval_types() to assign segment types (e.g. miRNA, mRNA) and eval_mirna() to identify which segment is the miRNA (5p/3p/both/none). Tally counts per replicate with hyb_analyze (or the hybkit.analysis.Analysis class via the API), then aggregate the per-replicate type and miRNA tables into combined tables for the dataset. Plot the hybrid-type distribution as a pie chart and miRNA characteristics as bar charts with hybkit's built-in matplotlib-based plotting, saving figures alongside the CSV tables.

## Related tools

- **hybkit** (Provides HybFile/HybRecord parsing, hyb_check/hyb_filter/hyb_analyze command-line utilities, the analysis.Analysis class for tallying type/miRNA counts, and built-in matplotlib plotting) — http://www.github.com/RenneLab/hybkit
- **matplotlib** (Backend for hybkit's built-in plotting of the type pie chart and miRNA bar charts (>= 3.7.1)) — https://matplotlib.org/
- **Python3** (Runtime for the hybkit API variant of the workflow (Python 3.8+)) — https://pypi.org/project/hybkit/
- **BioPython** (Dependency of the hybkit API (>= 1.79)) — https://biopython.org/

## Examples

```
hyb_analyze -i replicate1.hyb -f replicate1.vienna --analysis type --make_plots; hyb_record.eval_types(); hyb_record.eval_mirna()
```

## Evaluation signals

- Each replicate produces a type and miRNA count table whose totals match the number of filtered hybrid records (post ribosomal/mitochondrial filtration)
- The combined hybrid-type pie chart renders with the expected segment-type categories mirroring the example_01 outputs shipped in the hybkit repository
- Combined tables match the reference CSV (combined_analysis_type_hybrid_types.csv) from example_01_type_mirna_analysis/example_output
- hyb_check reports no integrity errors — every line in each .hyb file carries the expected 15-16 tab-delimited columns
- miRNA eval assignments fall into the expected 5p/3p/both/none categories for every record

## Limitations

- Requires hyb-format input as defined by the Hyb software package; other chimeric-RNA formats need conversion first
- Requires Python3.8+, matplotlib >= 3.7.1, BioPython >= 1.79, and typing_extensions >= 4.8.0
- The workflow depends on prior artifact removal of ribosomal- and mitochondrial-RNA hybrids via hyb_filter; skipping this biases type counts

## Evidence

- [methods] Each analysis performs quality control steps on the data by checking data integrity (hyb_check): "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [methods] removing artifactual ribosomal- and mitochondrial-RNA hybrids: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [methods] Type/miRNA Analysis description: "Type/miRNA Analysis           Quantify sequence types and miRNA types in a hyb file"
- [readme] Example analysis returns hybrid sequence types as csv table and pie chart: "the example summary analysis includes the return of the contained hybrid sequence types as both a csv table and as a pie chart"
- [readme] HybRecord eval methods for assigning segment types and miRNAs: "# Analyze each record to assign segment types
                hyb_record.eval_types()"
