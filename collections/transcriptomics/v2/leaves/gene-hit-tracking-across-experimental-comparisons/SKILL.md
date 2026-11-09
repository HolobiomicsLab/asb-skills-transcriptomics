---
name: gene-hit-tracking-across-experimental-comparisons
description: Use when several pairwise Treatment-vs-Control RNA-seq comparisons (e.g.,
  oxacillin, loratadine, Cmpd8 co-treatments vs Control in MRSA USA100) need screening
  for genes extreme in log2 fold-change and padj<=0.05.
when_to_use_negative:
- Input is raw sequencing output lacking pre-computed log2 fold-change and adjusted
  p-value columns; the README requires these columns (with matching header names)
  in the CSV before data_loading.
- Only one comparison is planned and the multiplication factor would be set to 1,
  which the README states produces a four-way plot, not the scorecard.
- The goal is experience-driven tracking of a preselected gene list; the article positions
  the scorecard to bypass gene preselection and stay data-driven.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scorecard_functions
  role: 'Core Python library for this skill: generate_parameters tunes th_significance/multiplication
    factor, data_loading ingests the FC/padj CSV, scorecard stores significant quadrant
    hits, reconstruct_scorecard rebuilds each comparison, and multiple_view/count_frequencies/common_entries
    track hits across comparisons'
  repo: https://github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the CSV of log2 fold changes and adjusted p-values into the DataFrame
    consumed by the scorecard functions
- name: TRANSIT
  role: Upstream generation of the log2 fold-change inputs for dataset 1 via resampling
    with beta-geometric normalization and Benjamini-Hochberg correction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:2ff2b3c06e37d52e7826e9c8c11f69c4d16c0befb1b14ca1e06eefb76a7cdf55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/m89p067/Scorecard
  tools:
  - Pandas
  - Python
  - TRANSIT
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gene-hit-tracking-across-experimental-comparisons

## Summary

Applies the Scorecard Python library (scorecard_functions) to pre-computed log2 fold-change/adjusted p-value CSVs so that genes marking a condition signature — e.g., the loratadine-oxacillin (Lor_Ox) adjuvant subgroup in MRSA USA100 Dataset 2 — are captured as statistically significant quadrant entries and tracked across multiple pairwise comparisons via frequency and commonality reports.

## When to use

Use when several pairwise Treatment-vs-Control RNA-seq comparisons (e.g., oxacillin, loratadine, Cmpd8 co-treatments vs Control in MRSA USA100) need screening for genes extreme in log2 fold-change and padj<=0.05.

## When NOT to use

- Input is raw sequencing output lacking pre-computed log2 fold-change and adjusted p-value columns; the README requires these columns (with matching header names) in the CSV before data_loading.
- Only one comparison is planned and the multiplication factor would be set to 1, which the README states produces a four-way plot, not the scorecard.
- The goal is experience-driven tracking of a preselected gene list; the article positions the scorecard to bypass gene preselection and stay data-driven.

## Inputs

- Preprocessed CSV of log2 fold changes and adjusted p-values (e.g., FPKM-derived MRSA USA100 Dataset 2) with a gene-ID column and per-comparison FC/padj header columns
- param_dict produced by scorecard_functions.generate_parameters() (base_dir, filename, gene_name, FC/padj column names, th_significance=0.05, multiplication factor, save_dir)
- main_folder path hosting one subfolder per pairwise experimental comparison containing scorecard plots and JSON quadrant outcomes

## Outputs

- Per-quadrant plots plus stored statistically significant (padj <= 0.05) gene entries for each quadrant of each comparison
- Reconstructed full scorecard images per comparison and a multiple_view overview aggregating all comparisons
- count_frequencies summaries and common_entries/uncommon-entries occurrence reports (CSV or Excel) listing gene hits per scorecard region

## How to apply

For every pairwise comparison (Ox, Cmpd8, Lor, and co-treatments vs Control, plus Lor_Ox vs Cmpd8_Ox), build param_dict with scorecard_functions.generate_parameters(), setting base_dir/filename to the preprocessed FPKM CSV, gene_name to the gene-ID column, the exact 'FC cond x/y' and 'padj cond x/y' header names, th_significance to 0.05, a th_fold_change with multiplication factor >1 to flag extreme values (factor 1 would collapse to an unfiltered four-way plot), and a per-comparison save_dir. Load the CSV via scorecard_functions.data_loading(param_dict) into a Pandas DataFrame — the file's first row must carry header names matching the dictionary. After drawing the color/region legend with scorecard_legend, run scorecard(df,param_dict) so only statistically significant (padj <= 0.05) entries are stored per quadrant, then reconstruct_scorecard to rebuild each full scorecard. Place one subfolder per comparison under main_folder and aggregate all wet-lab conditions with scorecard_functions.multiple_view(main_folder). Summarize with count_frequencies(main_folder) and common_entries(main_folder) — the latter must follow the former because it re-uses its outputs — to get occurrence reports in CSV or Excel. Correct application on Dataset 2 is confirmed by the 1126 outer-region gene count and by the Lor_Ox-specific hits SA_RS09375 (peptidylprolyl isomerase), SA_RS09760 (sensor histidine kinase), SA_RS09765 (LiaF), and SA_RS09835 (RlmD) that stay nearly silent under Cmpd8_Ox co-treatment.

## Related tools

- **scorecard_functions** (Core Python library for this skill: generate_parameters tunes th_significance/multiplication factor, data_loading ingests the FC/padj CSV, scorecard stores significant quadrant hits, reconstruct_scorecard rebuilds each comparison, and multiple_view/count_frequencies/common_entries track hits across comparisons) — https://github.com/m89p067/Scorecard
- **Pandas** (Loads the CSV of log2 fold changes and adjusted p-values into the DataFrame consumed by the scorecard functions)
- **TRANSIT** (Upstream generation of the log2 fold-change inputs for dataset 1 via resampling with beta-geometric normalization and Benjamini-Hochberg correction)

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict)
```

## Evaluation signals

- Only padj <= th_significance (0.05) entries are stored per quadrant; JSON outcomes must exclude non-significant genes.
- count_frequencies reproduces the Dataset 2 outer-region count of 1126 genes.
- common_entries report shows SA_RS03110/CwrA/DUF4930 (oxacillin-specific), SA_RS05690/SA_RS05695/SA_RS01280/AdhP (co-treatment-specific), and SA_RS09375/LiaF (Lor_Ox vs Cmpd8_Ox) in their expected scorecard regions.
- The Lor_Ox-specific subgroup SA_RS09375, SA_RS09760, SA_RS09765 (LiaF), SA_RS09835 (RlmD) is markedly over-expressed versus untreated but nearly silent under Cmpd8_Ox.
- main_folder contains one subfolder per comparison (Ox, Cmpd8, Lor, co-treatments vs Control, plus Lor_Ox vs Cmpd8_Ox) before multiple_view is invoked.

## Limitations

- No version number is stated for the Python library or Python itself, hindering reproducibility.
- Setting the multiplication factor to 1 reproduces a typical four-way plot that is not filtered by statistical significance.
- Plots can become crowded — volcano plots because minor differences are statistically significant with large sample sizes, and scorecards likewise; individual quadrant plots are provided for crowded cases.
- common_entries must be called after count_frequencies because it re-uses its outputs (same dependency applies to track_over_exper).

## Evidence

- [abstract] The scorecard identified and tracked, over time or experiments, genes meaningful for bacterial metabolism and survival: "The scorecard identified and tracked, over time or experiments, genes meaningful for bacterial metabolism and survival"
- [intro] To overcome the limitations of four-way plots in DE analyses, a scorecard has been released as a free-to-use Python software library.: "To overcome the limitations of four-way plots in DE analyses, a scorecard has been released as a free-to-use Python software library."
- [methods] Now load the CSV data into the computer memory as Pandas Dataframe by typing: "Now load the CSV data into the computer memory as Pandas Dataframe by typing"
- [results] there is a specific function included in the software library that produces a general visualization of all comparisons: "there is a specific function included in the software library that produces a general visualization of all comparisons"
- [discussion] extreme variations selected during pairwise comparisons based on: "extreme variations selected during pairwise comparisons based on"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
