---
name: hierarchical-cell-type-model-classification
description: Use when bulk DIE genes (from an Iso-based differential isoform test)
  must be attributed to constituent cell types — deciding whether a region-specific
  isoform shift originates in one cell type, multiple cell types, or no cell type.
when_to_use_negative:
- Input is raw fastq or barcoded reads needing barcode deconvolution, alignment (STARlong/Minimap2),
  or filtering — run the upstream scisorseqr steps (GetBarcodes, STARalign/MMalign,
  MapAndFilter, InfoPerLongRead, IsoQuant) first.
- The question is bulk gene-expression (not isoform-level) differential expression,
  since the classification requires per-cell-type isoform proportion (PI) shifts.
- No per-cell-type isoform counts are available (e.g. only unsorted bulk long-read
  data), because delta-PI(subtype) cannot be computed.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scisorseqr
  role: Upstream R-package producing the gene-wise DIE results table via DiffSplicingAnalysis
    (typeOfTest='Iso') that this classification consumes
  repo: https://github.com/noush-joglekar/scisorseqr
- name: R
  role: Language for the classification workflow and aggregation of model-class fractions
- name: Python
  role: Language used with pandas to compute delta-PI(subtype)/delta-PI(bulk) ratios
    and write die_model_classification.csv
- name: pandas
  role: Python library for loading the DIE results table and per-cell-type isoform
    counts and emitting the classification table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:62e563c59b9fe85e372b280547b75e3d5e5bc60a051cb707be38a8b90bc77a5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - Python
  - R
  - pandas
  - scisorseqr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# hierarchical-cell-type-model-classification

## Summary

Classifies bulk differential isoform expression (DIE) genes between mouse hippocampus (HIPP) and prefrontal cortex (PFC) into Single-Cell-Type-Model, Both-Cell-Types-Model, or No-Cell-Type-Model by comparing per-cell-type delta-PI shifts to the bulk delta-PI, revealing which cell types drive region-specific splicing.

## When to use

Use when bulk DIE genes (from an Iso-based differential isoform test) must be attributed to constituent cell types — deciding whether a region-specific isoform shift originates in one cell type, multiple cell types, or no cell type.

## When NOT to use

- Input is raw fastq or barcoded reads needing barcode deconvolution, alignment (STARlong/Minimap2), or filtering — run the upstream scisorseqr steps (GetBarcodes, STARalign/MMalign, MapAndFilter, InfoPerLongRead, IsoQuant) first.
- The question is bulk gene-expression (not isoform-level) differential expression, since the classification requires per-cell-type isoform proportion (PI) shifts.
- No per-cell-type isoform counts are available (e.g. only unsorted bulk long-read data), because delta-PI(subtype) cannot be computed.

## Inputs

- Gene-wise DIE results table from scisorseqr DiffSplicingAnalysis (columns: Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR)
- Neuron and non-neuron isoform count tables from the deposited isoformAtlas dataset (per-cell-type isoform usage)
- Bulk HIPP-vs-PFC isoform proportion (PI) values per gene

## Outputs

- die_model_classification.csv — one row per DIE gene with Gene ID, model class (Single-/Both-/No-Cell-Type-Model), and delta-PI(subtype)/delta-PI(bulk) ratios
- Class-level summary statistics: fraction of genes per model (Single-Cell-Type 75% ± 2.3 SE, Both 24.4%, No 0.3%) and significant-gene counts per class (151/81/26)

## How to apply

Load the gene-wise DIE results table (columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR) produced by the upstream scisorseqr DiffSplicingAnalysis (typeOfTest='Iso'), then load the neuron and non-neuron isoform count tables from the isoformAtlas dataset to obtain per-cell-type isoform usage. Compute per-gene delta-PI between HIPP and PFC for the bulk comparison and for each constituent cell type. Classify each DIE gene by the delta-PI(subtype)/delta-PI(bulk) ratio: multiple subtypes reproducing the bulk shift → Both-Cell-Type-Model; one subtype reproducing it → Single-Cell-Type-Model; no subtype reproducing it → No-Cell-Type-Model. Aggregate tracing fractions (reported: 75% ± 2.3 SEp Single, 24.4% Both, 0.3% No, of 395 bulk DIE genes, yielding 151/81/26 significant genes respectively) and write per-gene classifications with ratios to die_model_classification.csv. The rationale is that regional isoform identity can override cell-type specificity only rarely, so most bulk shifts should trace to a single cell type.

## Related tools

- **scisorseqr** (Upstream R-package producing the gene-wise DIE results table via DiffSplicingAnalysis (typeOfTest='Iso') that this classification consumes) — https://github.com/noush-joglekar/scisorseqr
- **R** (Language for the classification workflow and aggregation of model-class fractions)
- **Python** (Language used with pandas to compute delta-PI(subtype)/delta-PI(bulk) ratios and write die_model_classification.csv)
- **pandas** (Python library for loading the DIE results table and per-cell-type isoform counts and emitting the classification table)

## Examples

```
df = pd.read_csv('die_results.csv'); ratios = compute_subtype_over_bulk_deltaPI(df, isoformAtlas_counts); df['model'] = ratios.apply(classify_model); df.to_csv('die_model_classification.csv', index=False)
```

## Evaluation signals

- The output table contains the Gene and model-class fields with one row per DIE gene, matching the input DIE result count (395 bulk DIE genes between HIPP and PFC).
- Class fractions fall within reported ranges: ~75% (± 2.3 SEp) Single-Cell-Type-Model, ~24.4% Both-Cell-Types-Model, ~0.3% No-Cell-Type-Model.
- Significant-gene counts per class total 26 (No) + 151 (Single) + 81 (Both) = 258, consistent with the reported breakdown.
- Exactly one gene is expected in the No-Cell-Type-Model class (0.3%), reflecting the finding that regional identity rarely overrides cell-type specificity.
- Every classified gene carries a finite delta-PI(subtype)/delta-PI(bulk) ratio traceable to neurons or non-neurons.

## Limitations

- Long-read clustering discrepancies may stem from sparsity of the isoform matrix due to lower long-read throughput, affecting per-cell-type delta-PI estimates.
- Whether observed DIE tracing patterns persist in disease case-control studies, across adult brain regions, and across species is not established.
- Traditional transcriptional cell-type definitions may not correspond to isoform expression programs, so neuron/non-neuron groupings used in the classification may need re-definition.

## Evidence

- [results] Bulk DIE gene cell-type model classification outcome: [evidence span withheld — anchor status: not_found]
- [methods] Upstream DIE testing that produces the classification input: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
- [abstract] Biological rationale for the Single-Cell-Type-Model predominance: "Mostly, one cell type is responsible for brain-region specific DIE. However, for fewer genes, multiple cell types influence DIE."
- [discussion] Population of DIE genes being classified: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [methods] Input schema of the DIE results table: [evidence span withheld — anchor status: not_found]
