---
name: module-eigengene-trait-correlation
description: Use when you have a PyWGCNA network with detected modules plus sample
  trait metadata (genotype, time point, brain region) and need to identify which modules
  are significantly associated with those traits before running module-level functional
  enrichment.
when_to_use_negative:
- No module assignments exist yet — run PyWGCNA module detection first (e.g., 17 modules
  with dark gray at 380 genes for the 3xTg-AD tutorial data) before correlating eigengenes
  with traits
- Sample traits are not defined or cannot be encoded per sample — eigengene-trait
  correlation requires external sample trait metadata
- Input is a single-cell expression matrix or a pre-computed feature table rather
  than a bulk RNA-seq count matrix suitable for weighted co-expression network construction
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Computes module eigengenes and their correlations/significance against external
    sample traits via eigengene network methodology, and supports module trait relationships
    heatmap visualization
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python
  role: Runtime environment for PyWGCNA (Python 3.9+ required per paper; README states
    3.10+); used to load trait metadata and write correlation results to CSV
  repo: https://github.com/mortazavilab/PyWGCNA
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:499d52ca6d0d2de9d4d187228a71e901aaba8f8ba222ff06f1b0bf3f17430a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - GO
  - PyWGCNA
  - Python
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# module-eigengene-trait-correlation

## Summary

PyWGCNA's eigengene network methodology for correlating co-expression module eigengenes with external sample traits (e.g., age progression, genotype, brain region) in MODEL-AD brain bulk RNA-seq datasets, retaining significant associations (P < 0.05) for downstream GO enrichment of trait-linked modules.

## When to use

Use when you have a PyWGCNA network with detected modules plus sample trait metadata (genotype, time point, brain region) and need to identify which modules are significantly associated with those traits before running module-level functional enrichment.

## When NOT to use

- No module assignments exist yet — run PyWGCNA module detection first (e.g., 17 modules with dark gray at 380 genes for the 3xTg-AD tutorial data) before correlating eigengenes with traits
- Sample traits are not defined or cannot be encoded per sample — eigengene-trait correlation requires external sample trait metadata
- Input is a single-cell expression matrix or a pre-computed feature table rather than a bulk RNA-seq count matrix suitable for weighted co-expression network construction

## Inputs

- PyWGCNA object with detected modules and gene-to-module assignments (e.g., module_assignments.csv from module detection)
- Sample trait metadata table for the RNA-seq samples (e.g., age progression, brain region, genotype, time point including 18 months)
- Count matrix-derived PyWGCNA network (e.g., 38 female hippocampus 3xTg-AD/WT bulk RNA-seq samples across three ages)

## Outputs

- module-trait correlation table with correlation values and P values per module eigengene (e.g., module_trait_correlations.csv, darkgray_module_results.csv)
- Set of significant (P < 0.05) module-trait associations used to select modules for GO/KEGG/REACTOME enrichment

## How to apply

Load the PyWGCNA object containing gene-to-module assignments and the sample trait metadata table (e.g., age progression and brain region for the 5xFAD MODEL-AD bulk RNA-seq dataset, or genotype and time point including 18 months for 3xTg-AD female hippocampus samples). Use PyWGCNA's eigengene network methodology to compute the correlation and significance between each module eigengene and the external traits, following the module trait relationships heatmap tutorial. Retain only significant associations (P < 0.05) — e.g., the coral module against age progression and the white module against hippocampal region in 5xFAD, or the dark gray module (380 genes) against genotype and the 18-month time point in 3xTg-AD. Write the correlation table (e.g., module_trait_correlations.csv or darkgray_module_results.csv) and feed the significant modules' gene lists into GO/KEGG/REACTOME functional enrichment to characterize the trait-linked biology.

## Related tools

- **PyWGCNA** (Computes module eigengenes and their correlations/significance against external sample traits via eigengene network methodology, and supports module trait relationships heatmap visualization) — https://github.com/mortazavilab/PyWGCNA
- **Python** (Runtime environment for PyWGCNA (Python 3.9+ required per paper; README states 3.10+); used to load trait metadata and write correlation results to CSV) — https://github.com/mortazavilab/PyWGCNA

## Examples

```
pip install PyWGCNA; pyWGCNA.module_trait_relationships_heatmap()
```

## Evaluation signals

- Each retained module-trait association meets the P < 0.05 criterion (e.g., coral module vs. age progression and white module vs. hippocampal region in 5xFAD; dark gray vs. genotype and 18-month time point in 3xTg-AD)
- Output CSV is valid and contains both correlation values and significance values per module-trait pair
- Recovered associations match paper-reported results: 5xFAD coral module strongly correlated with age progression (1335 genes), 3xTg-AD dark gray module (380 genes) strongly correlated with genotype and the 18-month time point
- Selected modules' subsequent GO enrichment reproduces expected term categories (neutrophil activation for coral; cilium organization for white; neutrophil degranulation and immune response for dark gray)
- Module sizes and module counts match tutorial expectations (e.g., 17 modules, dark gray = 380 genes) so eigengene inputs are traceable to the correct detection run

## Limitations

- Significance filtering is a hard cutoff (P < 0.05); non-significant module-trait pairs are discarded and not reported in the significant-association outputs
- Correlations are computed only between eigengenes and the external traits supplied in sample metadata; trait encoding choices (e.g., age progression, brain region labels) directly shape which associations can be recovered
- Results depend on upstream module detection matching the reference run (e.g., exactly 17 modules with a 380-gene dark gray module for the 3xTg-AD tutorial dataset); divergent soft-thresholding or preprocessing changes eigengene-trait associations

## Evidence

- [intro] Relate gene modules to one another and to external sample traits using eigengene network methodology: "for relating modules to one another and to external sample traits (using eigengene network methodology)"
- [readme] README tutorial for module trait relationships: [evidence span withheld — anchor status: external_doc]
- [abstract] PyWGCNA downstream analyses in abstract: [evidence span withheld — anchor status: not_found]
- [abstract] Data source for the 5xFAD tutorial: "The data underlying this article are available in GitHub at github.com/mortazavilab/PyWGCNA/tutorials/5xFAD_paper"
