---
name: matplotlib-categorical-plotting
description: Use when hyb-format records have already been evaluated with eval_types()/eval_mirna()
  and you need publication-ready categorical plots (pie/bar) of hybrid segment types
  or miRNA characteristics across qCLASH replicates.
when_to_use_negative:
- Input records have not yet been evaluated — plotting requires prior eval_types()/eval_mirna()
  annotation and hyb_filter removal of ribosomal/mitochondrial hybrids
- The goal is visualizing predicted miRNA folding patterns rather than categorical
  type/miRNA counts — use hybkit's Fold Analysis instead
- The analysis is a target-level (Target or Grouped Target Analysis) comparison rather
  than a whole-dataset type/miRNA distribution
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: hybkit
  role: Provides the built-in analysis and plotting functions (hybkit.analysis.Analysis,
    hyb_analyze) that turn tallied type/miRNA counts into pie and bar charts
  repo: http://www.github.com/RenneLab/hybkit
- name: matplotlib
  role: Backend rendering library (>= 3.7.1) underlying hybkit's built-in plotting
    of analysis results
  repo: https://matplotlib.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:7ddd61f0ceebc24785901c5c10cc22fa61ea130c6e620c36f899e7c9cbe26496
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# matplotlib-categorical-plotting

## Summary

Plot categorical distributions of qCLASH hybrid-sequence annotations — segment types as a pie chart and miRNA characteristics (5p/3p/both/none) as bar charts — using hybkit's built-in matplotlib-backed plotting functions, saving figures alongside the CSV count tables.

## When to use

Use when hyb-format records have already been evaluated with eval_types()/eval_mirna() and you need publication-ready categorical plots (pie/bar) of hybrid segment types or miRNA characteristics across qCLASH replicates.

## When NOT to use

- Input records have not yet been evaluated — plotting requires prior eval_types()/eval_mirna() annotation and hyb_filter removal of ribosomal/mitochondrial hybrids
- The goal is visualizing predicted miRNA folding patterns rather than categorical type/miRNA counts — use hybkit's Fold Analysis instead
- The analysis is a target-level (Target or Grouped Target Analysis) comparison rather than a whole-dataset type/miRNA distribution

## Inputs

- Filtered and annotated .hyb records (HybRecord objects carrying assigned segment types and miRNA segment assignments)
- Combined type/miRNA count tables (CSV) aggregated across the three qCLASH replicates from GEO GSE101978

## Outputs

- Pie chart of hybrid segment-type distribution (mirroring example_01 output combined_analysis_type_hybrid_types.csv)
- Bar charts of miRNA characteristics (5p/3p/both/none counts) per replicate and combined
- CSV count tables saved alongside the figures

## How to apply

After parsing each .hyb file with hybkit's HybFile iterator, filtering artifactual ribosomal- and mitochondrial-RNA hybrids via hyb_filter, and annotating each HybRecord with eval_types() (assigning segment types such as miRNA and mRNA) and eval_mirna() (identifying the miRNA segment as 5p/3p/both/none), tally counts per replicate with hyb_analyze or the hybkit.analysis.Analysis class. Plot the aggregated hybrid-type distribution as a pie chart and miRNA characteristics as bar charts with hybkit's built-in matplotlib-based plotting, saving figures next to the combined CSV tables. Validate that each replicate's type and miRNA count tables sum to the number of filtered records and that the combined pie chart shows the same segment-type categories as the example_01 outputs shipped in the hybkit repository.

## Related tools

- **hybkit** (Provides the built-in analysis and plotting functions (hybkit.analysis.Analysis, hyb_analyze) that turn tallied type/miRNA counts into pie and bar charts) — http://www.github.com/RenneLab/hybkit
- **matplotlib** (Backend rendering library (>= 3.7.1) underlying hybkit's built-in plotting of analysis results) — https://matplotlib.org/

## Examples

```
hyb_analyze -i my_hyb_file.hyb --analysis type
```

## Evaluation signals

- Each replicate's type and miRNA count tables total exactly the number of filtered hybrid records
- The combined hybrid-type pie chart renders with the expected segment-type categories mirroring the example_01 outputs shipped in the hybkit repository
- Both a CSV table and a graph are produced for each analysis (tabular and graph form)
- Figures are saved in the same output directory as the combined_analysis_type_hybrid_types.csv-equivalent tables
- MiRNA bar charts cover the 5p/3p/both/none assignment categories produced by eval_mirna()

## Limitations

- Categorical categories depend on hybkit's eval_types()/eval_mirna() custom criteria; mis-assigned segment types propagate directly into the plots
- Plots should be made after hyb_filter removal of artifactual ribosomal- and mitochondrial-RNA hybrids, or totals will not match filtered-record counts
- Requires matplotlib >= 3.7.1 as a hybkit dependency (Python3.8+)

## Evidence

- [abstract] The Hybkit API includes a suite of python objects for developing custom analyses of hybrid data as well as miRNA-specific analysis methods, built-in plotting of analysis results: "The Hybkit API includes a suite of python objects for developing custom analyses of hybrid data as well as miRNA-specific analysis methods, built-in plotting of analysis results"
- [readme] example summary analysis returns hybrid sequence types as csv table and pie chart: "As an illustration, the example summary analysis includes the
    return of the contained hybrid sequence types as both a csv table and as a pie chart:"
- [readme] Analyses provide results in tabular and graph form: "These analyses provide analysis results in both tabular and graph form."
- [readme] hyb_analyze performs type or miRNA analysis and plots results: "hyb_analyze                         Perform an energy, type, miRNA, target, or fold analysis
                                            on hyb (and fold) files and plot results"
- [methods] filtering artifactual ribosomal- and mitochondrial-RNA hybrids before analysis: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
