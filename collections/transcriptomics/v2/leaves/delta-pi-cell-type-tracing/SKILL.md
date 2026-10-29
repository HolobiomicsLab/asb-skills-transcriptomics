---
name: delta-pi-cell-type-tracing
description: Use when gene-wise bulk DIE results (e.g., HIPP vs PFC) must be attributed
  to the cell types that drive them, i.e., deciding Single-/Both-/No-Cell-Type-Model
  for each DIE gene from per-cell-type isoform usage.
when_to_use_negative:
- Input lacks cell-type-resolved long-read isoform counts (e.g., bulk short-read RNA-seq),
  so delta-PI(subtype)/delta-PI(bulk) ratios cannot be computed
- The question is exon-level spatial usage from ExonQuant/spatial maps rather than
  tracing bulk DIE genes to cell types
- Data are single-cell 3' short-read only with no isoform-level (full-length spliced
  long-read) quantification, since isoform tests require long-read ScISOrSeq data
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3304
tools:
- name: scisorseqr
  role: R-package whose DiffSplicingAnalysis function (typeOfTest='Iso') produces
    the gene-wise DIE results table that this skill traces to cell types
  repo: github.com/noush-joglekar/scisorseqr
- name: Python
  role: Scripting environment (with pandas) used to compute per-gene delta-PI(subtype)/delta-PI(bulk)
    ratios and write die_model_classification.csv
- name: pandas
  role: Library for loading the DIE results and isoformAtlas count tables and aggregating
    class fractions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:ffca0133424b7f9cf59e7a3a8b7d6900b8cf894df3b2e72f34a93f0050c9d687
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - R
  - pandas
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3304
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# delta-pi-cell-type-tracing

## Summary

Classifies each bulk differential isoform expression (DIE) gene by whether its brain-region dPI shift is recapitulated in a single cell type, multiple cell types, or no cell type, using the delta-PI(subtype)/delta-PI(bulk) ratio from the ScISOrSeq isoform atlas.

## When to use

Use when gene-wise bulk DIE results (e.g., HIPP vs PFC) must be attributed to the cell types that drive them, i.e., deciding Single-/Both-/No-Cell-Type-Model for each DIE gene from per-cell-type isoform usage.

## When NOT to use

- Input lacks cell-type-resolved long-read isoform counts (e.g., bulk short-read RNA-seq), so delta-PI(subtype)/delta-PI(bulk) ratios cannot be computed
- The question is exon-level spatial usage from ExonQuant/spatial maps rather than tracing bulk DIE genes to cell types
- Data are single-cell 3' short-read only with no isoform-level (full-length spliced long-read) quantification, since isoform tests require long-read ScISOrSeq data

## Inputs

- Gene-wise DIE results table from DiffSplicingAnalysis with columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- Neuron and non-neuron isoform count tables from the deposited isoformAtlas dataset (per-cell-type isoform usage)
- Bulk brain-region comparison (HIPP vs PFC, mouse P7) cell-type composition

## Outputs

- die_model_classification.csv with one row per DIE gene: Gene, model class (Single-/Both-/No-Cell-Type-Model), delta-PI(subtype)/delta-PI(bulk) ratios
- Class-level summary statistics: fraction of DIE genes per model with standard error (Single 75% ±2.3 SEp, Both 24.4%, No 0.3%) and significant-gene counts (151/81/26)

## How to apply

Load the gene-wise DIE results table (columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR) produced by DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE), and load the neuron and non-neuron isoform count tables from the deposited isoformAtlas dataset (www.isoformAtlas.com). Compute per-gene delta-PI (change in percent-isoform usage) between brain regions for the bulk comparison and independently for each constituent cell type. Classify each DIE gene by the delta-PI(subtype)/delta-PI(bulk) ratio: Both-Cell-Type-Model when multiple subtypes jointly account for the bulk shift, Single-Cell-Type-Model when one subtype (e.g., only neurons or only non-neurons) accounts for it, and No-Cell-Type-Model when no subtype accounts for it (shift arises purely from gene-expression or cell-type abundance differences). Aggregate the class fractions across genes and write die_model_classification.csv with gene ID, model class, per-subtype ratios, and class-level summary statistics.

## Related tools

- **scisorseqr** (R-package whose DiffSplicingAnalysis function (typeOfTest='Iso') produces the gene-wise DIE results table that this skill traces to cell types) — github.com/noush-joglekar/scisorseqr
- **Python** (Scripting environment (with pandas) used to compute per-gene delta-PI(subtype)/delta-PI(bulk) ratios and write die_model_classification.csv)
- **pandas** (Library for loading the DIE results and isoformAtlas count tables and aggregating class fractions)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- die_model_classification.csv contains Gene and model-class fields with exactly one row per DIE gene matching the input DIE result count
- Aggregated class fractions fall within reported ranges: Single-Cell-Type 75% ±2.3 SE, Both ~24.4%, No ~0.3%
- Significant-gene counts per class reproduce 151 (Single), 81 (Both), 26 (No)
- Per-gene delta-PI(subtype)/delta-PI(bulk) ratios are populated for every classified gene and each ratio is consistent with its assigned model class
- Only one gene (0.3%) should fall into the No-Cell-Type-Model; an excess of No-Cell-Type classifications indicates miscomputed bulk vs subtype dPI

## Limitations

- The isoform matrix is sparse due to lower long-read throughput, so cell-type-level dPI estimates for lowly covered genes may be unreliable
- Generalizability is untested: whether these tracing fractions persist in disease case-control studies, across adult brain regions, and across species requires further studies
- Regional identity can rarely override cell-type specificity (only one gene followed the No-Cell-Type-Model), so the classification is dominated by single-cell-type attribution rather than region-intrinsic splicing

## Evidence

- [results] Single vs both cell type tracing fractions: [evidence span withheld — anchor status: not_found]
- [intro] Motivation for cell-type attribution of regional DIE: "Brain-region-specific isoform expression can either originate from molecular regulation in one or multiple cell types, or can arise purely from gene-expression or cell-type abundance differences"
- [abstract] Dominant single-cell-type responsibility: "Mostly, one cell type is responsible for brain-region specific DIE. However, for fewer genes, multiple cell types influence DIE."
- [methods] Upstream DIE test configuration: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
- [readme] Tool producing the input DIE table: [evidence span withheld — anchor status: external_doc]
