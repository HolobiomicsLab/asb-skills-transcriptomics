---
name: hybrid-analysis-result-plotting
description: Use when you have completed a hybkit analysis (e.g., Target Analysis
  of kshv-miR-k12-5 qCLASH hybrids) and need the counts exported as CSV tables and
  distributions visualized as plots.
when_to_use_negative:
- Input is raw hyb/vienna data that has not yet passed hyb_check QC and hyb_filter
  artifact removal — run the analysis first, then plot
- You need interactive or publication-custom figure editing beyond hybkit's built-in
  matplotlib rendering (>= 3.7.1)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0659
tools:
- name: hybkit
  role: Provides built-in plotting of analysis results and CSV export via the Analysis
    class and hyb_analyze command-line utility
  repo: https://github.com/RenneLab/hybkit
- name: matplotlib
  role: Rendering backend (>= 3.7.1) for hybkit's built-in analysis plots such as
    pie charts and target distributions
  repo: https://matplotlib.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:57813fced7cf984e7f1f1ed1f86d428a12a8fd30fcff10813a325c0ad8d1d430
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RenneLab/hybkit
  tools:
  - Python3
  - hybkit
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hybrid-analysis-result-plotting

## Summary

Render results of hybkit analyses (type/miRNA, target, grouped target, fold) of hyb-format chimeric RNA data as tabular CSV outputs and matplotlib-based graphs such as pie charts of hybrid sequence types.

## When to use

Use when you have completed a hybkit analysis (e.g., Target Analysis of kshv-miR-k12-5 qCLASH hybrids) and need the counts exported as CSV tables and distributions visualized as plots.

## When NOT to use

- Input is raw hyb/vienna data that has not yet passed hyb_check QC and hyb_filter artifact removal — run the analysis first, then plot
- You need interactive or publication-custom figure editing beyond hybkit's built-in matplotlib rendering (>= 3.7.1)

## Inputs

- hybkit analysis result objects/tables from Type/miRNA, Target, Grouped Target, or Fold Analysis
- Filtered hyb-format records (e.g., miRNA-containing hybrids after eval_types/eval_mirna)
- CSV count tables produced by hyb_analyze or the hybkit Analysis class

## Outputs

- CSV tables of analysis counts (e.g., combined_analysis_type_hybrid_types.csv)
- PNG/matplotlib plots of analysis results (e.g., pie chart of hybrid sequence types, target distribution plots)

## How to apply

After running a hybkit analysis with the Analysis class or hyb_analyze (type/miRNA, target, grouped target, or fold mode), export the count results as CSV tables and render distribution plots via hybkit's built-in plotting, which relies on matplotlib >= 3.7.1. The example analyses return results 'in both tabular and graph form'; e.g., the type/miRNA summary analysis returns contained hybrid sequence types as both a CSV table and a pie chart. Plot target distributions for the selected miRNA-containing subset (e.g., kshv-miR-k12-5 hybrids from GSE101978 qCLASH data) after QC (hyb_check) and artifact removal (hyb_filter). Validate that the CSV count tables and PNG plots are parseable and internally consistent with the number of analyzed records.

## Related tools

- **hybkit** (Provides built-in plotting of analysis results and CSV export via the Analysis class and hyb_analyze command-line utility) — https://github.com/RenneLab/hybkit
- **matplotlib** (Rendering backend (>= 3.7.1) for hybkit's built-in analysis plots such as pie charts and target distributions) — https://matplotlib.org/

## Examples

```
$ hyb_analyze target -i filtered.hyb --plot
```

## Evaluation signals

- CSV output files exist and are parseable as tabular data (e.g., hybrid-type count columns match record totals)
- PNG plot files exist and render a legible distribution chart of the analysis results
- Per-miRNA target counts in the CSV are internally consistent with the number of miRNA-containing hybrid records analyzed
- Plot reflects the filtered record subset (ribosomal-/mitochondrial-RNA hybrids removed before counting)
- matplotlib version >= 3.7.1 satisfied so hybkit plotting calls execute without error

## Limitations

- Plots are produced by hybkit's built-in matplotlib rendering; customization beyond provided options requires direct matplotlib work
- Results are 'in both tabular and graph form' only for the provided example analyses; custom analyses must assemble their own plotting via the API
- Target Analysis plots targets 'from a single experimental replicate'; pooled replicates require Grouped Target Analysis instead

## Evidence

- [abstract] The Hybkit API includes a suite of python objects for developing custom analyses of hybrid data as well as miRNA-specific analysis methods, built-in plotting of analysis results: "miRNA-specific analysis methods, built-in plotting of analysis results, and incorporation of"
- [readme] These analyses provide analysis results in both tabular and graph form.: "These analyses provide analysis results in both tabular and graph form."
- [readme] the example summary analysis includes the return of the contained hybrid sequence types as both a csv table and as a pie chart: "the example summary analysis includes the return of the contained hybrid sequence types as both a csv table and as a pie chart"
- [methods] Analyze and plot targets of a set of miRNAs from pooled experimental replicates: "Analyze and plot targets of a set of miRNAs from pooled experimental replicates"
- [methods] matplotlib >= 3.7.1 dependency for hybkit plotting: "`matplotlib <https://matplotlib.org/>`_ >= 3.7.1 (|Hunter2007|_)"
