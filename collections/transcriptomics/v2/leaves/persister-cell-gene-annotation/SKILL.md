---
name: persister-cell-gene-annotation
description: Use when a scorecard run on antibiotic-exposed bacterial biofilm transcriptomics
  (e.g. VAN comparisons yielding Q3 down-regulated SAUSA300 entries) needs gene-level
  functional interpretation as persistence or survival strategies.
when_to_use_negative:
- Differential expression has not yet been filtered through the scorecard with statistical-significance-aware
  quadrants — typical four-way or volcano plots alone do not provide the occurrence
  reports this annotation step consumes
- The input transcriptome lacks gene IDs or locus tags in a dedicated column (gene_name
  parameter cannot map entries to known genes)
- The goal is raw expression quantification rather than data-driven extraction of
  extreme or moderate fold-change variations across multiple experimental conditions
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0203
tools:
- name: scorecard_functions
  role: Python library whose scorecard, reconstruct_scorecard, count_frequencies and
    common_entries functions produce the quadrant entries and occurrence reports used
    to select and track candidate persister genes
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the preprocessed Dataset 1 CSV (log2 fold-change and adjusted p-value
    columns per comparison) into a DataFrame consumed by the scorecard functions
- name: TRANSIT
  role: Computed the dataset 1 log2 fold-changes via resampling with beta-geometric
    normalization and Benjamini-Hochberg correction, upstream of scorecard annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:75e5c7bbc14f8e919477cae4519b2ba6dd1b7624296d7b95a39087d3e146e364
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - TRANSIT
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# persister-cell-gene-annotation

## Summary

Annotate genes surfaced by the Scorecard library (e.g. SAUSA300 loci from Dataset 1 transposon-sequencing comparisons under VAN, LZD, CPT, DEL) as candidate persister-cell or biofilm-survival factors by linking scorecard quadrant/occurrence reports to known mechanisms such as altered penicillin-protein binding affinity, cell wall thickening, efflux pumps, and enzymatic drug inactivation.

## When to use

Use when a scorecard run on antibiotic-exposed bacterial biofilm transcriptomics (e.g. VAN comparisons yielding Q3 down-regulated SAUSA300 entries) needs gene-level functional interpretation as persistence or survival strategies.

## When NOT to use

- Differential expression has not yet been filtered through the scorecard with statistical-significance-aware quadrants — typical four-way or volcano plots alone do not provide the occurrence reports this annotation step consumes
- The input transcriptome lacks gene IDs or locus tags in a dedicated column (gene_name parameter cannot map entries to known genes)
- The goal is raw expression quantification rather than data-driven extraction of extreme or moderate fold-change variations across multiple experimental conditions

## Inputs

- Scorecard quadrant JSON/CSV outcomes and reconstructed scorecards per VAN comparison (from scorecard_functions.scorecard and reconstruct_scorecard)
- Total-occurrence reports of common and uncommon entries (from count_frequencies and common_entries on main_folder)
- Preprocessed Dataset 1 CSV with pre-computed log2 fold-change and adjusted p-value columns per comparison (SAUSA300 locus-tag column as gene_name)

## Outputs

- Gene-level findings report saved as dataset1_gene_findings_report.csv in the save_dir
- Per-gene annotation of scorecard entries (quadrant, direction, occurrence count) mapped to persister-cell mechanisms

## How to apply

After running the scorecard pipeline (generate_parameters with fold-change thresholds 2-3, multiplication factor 1.5, p<=0.05; data_loading; scorecard; reconstruct_scorecard; count_frequencies; common_entries), extract the quadrant and occurrence report entries for candidate genes such as the Gra/Aps cell-envelope stress loci (SAUSA300_RS03455, SAUSA300_RS03465/GraS-ApsS, SAUSA300_RS03475), the quorum-sensing gene AgrB (SAUSA300_RS10935), and RsbW (SAUSA300_RS11125). Check directionality: VAN comparisons mainly place down-regulated genes in quadrant Q3 (e.g. GraS markedly downregulated under vancomycin versus CPT and LZA), while other antibiotics yield markedly over-expressed genes (e.g. SAUSA300_RS03525, SAUSA300_RS04560, SAUSA300_RS08475). Annotate each confirmed entry against known persistence mechanisms — modified binding affinity of penicillin-related proteins or ribosomal subunits, alternative metabolic pathways, cell wall thickening, reduced intracellular drug concentration via efflux pumps, and enzymatic inactivation by hydrolyzation, phosphorylation, or adenylation — and compile the gene-level findings report as dataset1_gene_findings_report.csv in the save_dir.

## Related tools

- **scorecard_functions** (Python library whose scorecard, reconstruct_scorecard, count_frequencies and common_entries functions produce the quadrant entries and occurrence reports used to select and track candidate persister genes) — github.com/m89p067/Scorecard
- **Pandas** (Loads the preprocessed Dataset 1 CSV (log2 fold-change and adjusted p-value columns per comparison) into a DataFrame consumed by the scorecard functions)
- **TRANSIT** (Computed the dataset 1 log2 fold-changes via resampling with beta-geometric normalization and Benjamini-Hochberg correction, upstream of scorecard annotation)

## Examples

```
param_dict=scorecard_functions.generate_parameters(); scorecard_functions.common_entries(main_folder)
```

## Evaluation signals

- GraS/ApsS (SAUSA300_RS03465) appears as markedly downregulated only in VAN comparisons versus CPT and LZA, with negative enhancement under vancomycin
- GraX/ApsX loci (SAUSA300_RS03455, SAUSA300_RS03475) fall in extreme VAN quadrant Q3 (down-regulated side) of the scorecard
- AgrB (SAUSA300_RS10935) is differential in all VAN quorum-sensing comparisons and RsbW (SAUSA300_RS11125) is captured in the occurrence reports
- dataset1_gene_findings_report.csv exists in the save_dir and contains total-occurrence entries consistent with count_frequencies output
- Scorecard thresholds match the paper: fold-change thresholds between 2 and 3 (multiplication factor 1.5) and significance level p<=0.05

## Limitations

- Typical four-way plots are not filtered by statistical significance and cannot merge information across multiple plots, so pre-scorecard plots cannot substitute for the occurrence reports
- Volcano plots can be crowded because minor differences can be statistically significant with large sample sizes, and they show one condition per graph
- Experience-driven tracking of preselected genes might hide relevant information arising from actual data; the scorecard's data-driven extraction is the intended route, not preselecting genes
- No version number is stated for the Python library or for Python itself

## Evidence

- [results] The scorecard identified SAUSA300_RS03465 (histidine kinase GraS/ApsS) as markedly downregulated in VAN compared with CPT and LZA only, showing negative enhancement under vancomycin.: "the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed"
- [abstract] Persistence mechanisms enumerated in the abstract motivate the gene annotations.: "Analyses detected factors and strategies to persist in the environment by bacterial aggregates, such as modifications in the binding affinity of penicillin-related proteins"
- [readme] README describes the analysis functions feeding the annotation step.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] README describes the final per-gene report generation.: "To create a final report on single genes/entries run the following function."
- [results] Scorecard application thresholds on dataset 1.: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5)"
