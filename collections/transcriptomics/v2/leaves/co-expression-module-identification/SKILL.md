---
name: co-expression-module-identification
description: Use when given a gene-by-sample bulk RNA-seq expression matrix (e.g.
  100-sample ENCODE or MODEL-AD brain data) and the goal is to find co-expressed gene
  modules, relate them to traits like genotype or time point, and run GO/KEGG/REACTOME
  enrichment.
when_to_use_negative:
- The input is single-cell data or already a feature/summary table rather than a raw
  gene-by-sample expression matrix needing data cleaning and pre-processing
- The analysis requires differential expression or pathway testing on individual genes
  rather than co-expression module-level structure
- The dataset is small enough that runtime/memory are non-issues and an existing R
  WGCNA workflow is already in place
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0121
tools:
- name: PyWGCNA
  role: 'Primary library: loads expression data, identifies co-expression modules,
    computes eigengenes and module membership, relates modules to traits, and runs
    functional enrichment'
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python
  role: Runtime environment; Python 3.9 or greater required (README states 3.10 or
    greater)
- name: WGCNA
  role: R reference implementation used for runtime/accuracy benchmarking on the same
    expression subsets
- name: STRING
  role: 'Downstream: recover and plot protein-protein interactions for module genes'
  repo: https://github.com/mortazavilab/PyWGCNA
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:92e696d550c7a245ef70203eda0b67748df27d9474eb7f4cd765e62dfb39927f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - GO
  - PyWGCNA
  - Python
  - STRING
  - WGCNA
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# co-expression-module-identification

## Summary

Identifies clusters (modules) of highly correlated genes from bulk RNA-seq expression matrices using PyWGCNA's weighted correlation network analysis, then summarizes each module via its eigengene. Enables downstream module-trait correlation and functional enrichment on large datasets where the R WGCNA implementation is slower or exhausts memory.

## When to use

Use when given a gene-by-sample bulk RNA-seq expression matrix (e.g. 100-sample ENCODE or MODEL-AD brain data) and the goal is to find co-expressed gene modules, relate them to traits like genotype or time point, and run GO/KEGG/REACTOME enrichment.

## When NOT to use

- The input is single-cell data or already a feature/summary table rather than a raw gene-by-sample expression matrix needing data cleaning and pre-processing
- The analysis requires differential expression or pathway testing on individual genes rather than co-expression module-level structure
- The dataset is small enough that runtime/memory are non-issues and an existing R WGCNA workflow is already in place

## Inputs

- Gene-level or transcript-level bulk RNA-seq expression matrix (e.g. ENCODE gene/transcript matrices with 100 samples, or MODEL-AD 5xFAD/3xTg-AD brain bulk RNA-seq)
- Sample trait metadata (e.g. genotype, time point including 18 months)
- External gene marker lists (optional, for network comparison)

## Outputs

- PyWGCNA object with module assignments per gene
- Module eigengene values and module-trait correlation/significance tables
- GO/KEGG/REACTOME functional enrichment results per module (e.g. neutrophil degranulation, immune response; genes Csf1, Tyrobp, Trem2)
- Comparison tables of overlapping modules across datasets (e.g. darkgray_module_results.csv, runtime_comparison.csv)

## How to apply

Format the expression matrix (genes as features, samples as columns) using PyWGCNA's data input, cleaning and pre-processing conventions, then load it into a PyWGCNA object and run module identification ('Quick Start': load data, find modules, analyze them). PyWGCNA builds the weighted correlation network, finds clusters of highly correlated genes, and summarizes each cluster using the module eigengene; Python 3.9+ (README: 3.10+) is required and the package installs from PyPi via pip. Relate modules to external sample traits (e.g. genotype, 18-month time point) using eigengene network methodology, and run functional_enrichment_analysis on module gene lists against GO, KEGG, and REACTOME. For very large feature sets (e.g. 96,000 transcripts), PyWGCNA remains feasible where R WGCNA fails on memory, and it runs roughly 2x faster above ~16,000 genes. Modules across datasets can also be compared (two or more PyWGCNA objects, or against external gene marker lists) to assess conservation or enrichment.

## Related tools

- **PyWGCNA** (Primary library: loads expression data, identifies co-expression modules, computes eigengenes and module membership, relates modules to traits, and runs functional enrichment) — https://github.com/mortazavilab/PyWGCNA
- **Python** (Runtime environment; Python 3.9 or greater required (README states 3.10 or greater))
- **WGCNA** (R reference implementation used for runtime/accuracy benchmarking on the same expression subsets)
- **STRING** (Downstream: recover and plot protein-protein interactions for module genes) — https://github.com/mortazavilab/PyWGCNA

## Examples

```
import PyWGCNA; pywgcna = PyWGCNA.WGCNA(name='5xFAD', datExpr=data, save=True); pywgcna.analyzeWGCNA()
```

## Evaluation signals

- Module-trait table shows dark gray module (380 genes) significantly correlated with genotype and the 18-month time point
- Enrichment output recovers expected terms (neutrophil degranulation, immune response) and member genes (Csf1, Tyrobp, Trem2) reported for the 3xTg-AD dataset
- Scaling check: runtimes comparable to R WGCNA up to 16,000 genes, roughly 2x faster above that, and successful module identification on 96,000 transcripts where R WGCNA hits memory constraints
- Output file exists and is valid CSV containing both module-trait correlations and GO enrichment terms with associated genes
- Modules compared across datasets (e.g. two MODEL-AD brain RNA-seq sets) show significant overlap, consistent with the paper's shared co-expression signatures

## Limitations

- Requires Python 3.9+ (article methods) or 3.10+ per the current README; older environments are unsupported
- No discussion section text is present in the provided document, so paper-level caveats beyond the README are unavailable
- Runtime advantage over R WGCNA is dataset-size dependent: performance is similar up to 16,000 genes, so speedup claims do not apply to small matrices

## Evidence

- [intro] PyWGCNA is a Python library designed to do weighted correlation network analysis (WGCNA).: "PyWGCNA is a Python library designed to do weighted correlation network analysis (WGCNA)."
- [readme] README tutorial list: [evidence span withheld — anchor status: not_found]
- [readme] eigengene and trait relationships: "for relating modules to one another and to external sample traits (using eigengene network methodology), and for calculating module membership measures"
- [abstract] functional enrichment databases: "functional enrichment analysis using GO, KEGG, and REACTOME"
- [results] runtime benchmark finding: [evidence span withheld — anchor status: not_found]
