---
name: wgcna-module-trait-association-testing
description: Use when a PyWGCNA co-expression network has already been built and you
  need module–trait correlations (e.g., module eigengene vs. genotype or age/time
  point) plus GO/KEGG/REACTOME enrichment of the correlated module.
when_to_use_negative:
- No co-expression modules have been detected yet — module identification (task_003-style
  WGCNA network construction) must come first
- Input is a single-gene differential expression table rather than a PyWGCNA object
  with eigengenes; use standard DE tools instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Implements eigengene-based module–trait correlation and functional_enrichment_analysis
    on the dark gray module gene list
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Gene Ontology (GO)
  role: Reference database used by PyWGCNA's functional enrichment to recover terms
    such as neutrophil degranulation and immune response
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:d2f06374671d1e8a3d869adc3bc23578826d352f93d1b12f325acb8ed3c4fc56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - GO
  - Gene Ontology (GO)
  - PyWGCNA
  - Python
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# wgcna-module-trait-association-testing

## Summary

PyWGCNA-based workflow for correlating co-expression module eigengenes with external sample traits (e.g., genotype and 18-month time point in 5xFAD/3xTg-AD brain bulk RNA-seq) and for running GO/KEGG/REACTOME functional enrichment on trait-associated modules such as the dark gray module containing Csf1, Tyrobp, and Trem2.

## When to use

Use when a PyWGCNA co-expression network has already been built and you need module–trait correlations (e.g., module eigengene vs. genotype or age/time point) plus GO/KEGG/REACTOME enrichment of the correlated module.

## When NOT to use

- No co-expression modules have been detected yet — module identification (task_003-style WGCNA network construction) must come first
- Input is a single-gene differential expression table rather than a PyWGCNA object with eigengenes; use standard DE tools instead

## Inputs

- PyWGCNA object (pickled .p file) with module assignments from the prior module-detection run
- Sample trait metadata table mapping samples to genotype and time point (including 18 months) for the brain bulk RNA-seq dataset

## Outputs

- Module–trait correlation and significance table (dark gray eigengene vs. genotype and 18-month time point)
- GO/KEGG/REACTOME enrichment table with term names and associated genes
- Single CSV output file, darkgray_module_results.csv, containing both tables

## How to apply

Load the PyWGCNA object produced in the prior module-detection step together with the sample trait metadata (genotype and time point including 18 months). Relate modules to external sample traits using the eigengene network methodology implemented in PyWGCNA, extracting the correlation and significance of the dark gray module eigengene with genotype and with the 18-month time point. Then run functional_enrichment_analysis on the dark gray module's 380-gene list against GO (optionally KEGG/REACTOME), confirming recovery of terms such as neutrophil degranulation and immune response among genes including Csf1, Tyrobp, and Trem2. Write both the module–trait correlation table and the enrichment table (term names plus associated genes) into a single output file, darkgray_module_results.csv.

## Related tools

- **PyWGCNA** (Implements eigengene-based module–trait correlation and functional_enrichment_analysis on the dark gray module gene list) — https://github.com/mortazavilab/PyWGCNA
- **Gene Ontology (GO)** (Reference database used by PyWGCNA's functional enrichment to recover terms such as neutrophil degranulation and immune response)

## Examples

```
import PyWGCNA; pywgcna = PyWGCNA.readWGCNA('3xTg-AD.p'); moduleTraitCor = pywgcna.analyseWGCNA(traitData=trait_data); pywgcna.functional_enrichment_analysis()
```

## Evaluation signals

- Output file darkgray_module_results.csv exists and parses as valid CSV
- CSV contains both a module–trait correlation table (dark gray vs. genotype and 18-month time point) and a GO enrichment table with term names and associated genes
- Enrichment terms include neutrophil degranulation and immune response
- Module genes include Csf1, Tyrobp, and Trem2
- Dark gray module eigengene shows strong correlation with genotype and the 18-month time point, reported for expert review against the paper's dark gray findings

## Limitations

- PyWGCNA requires Python 3.9 or greater per the article (README states 3.10 or greater for current releases) — check the environment version before installing
- Module labels (e.g., 'dark gray') and gene composition depend on the upstream network construction parameters from the prior PyWGCNA run; changing those changes the trait associations
- Trait metadata must include the time points of interest (e.g., 18 months); missing trait rows will prevent correlation extraction

## Evidence

- [readme] for relating modules to one another and to external sample traits (using eigengene network methodology), and for calculating module membership measures: "for relating modules to one another and to external sample traits (using eigengene network methodology), and for calculating module membership measures"
- [abstract] We apply PyWGCNA to two distinct datasets of brain bulk RNA-seq from MODEL-AD to identify modules associated with the genotypes.: "We apply PyWGCNA to two distinct datasets of brain bulk RNA-seq from MODEL-AD to identify modules associated with the genotypes."
- [readme] How to calculate correlation between modules and traits: [evidence span withheld — anchor status: not_found]
- [abstract] functional enrichment analysis using GO, KEGG, and REACTOME: "PyWGCNA has a faster implementation than the R version of WGCNA and several additional downstream analysis modules for functional enrichment analysis using GO, KEGG, and REACTOME"
