---
name: pairwise-celltype-comparison-statistics
description: Use when per-cell-type isoform or TSS count tables (e.g. 'Modality-ModalityID.csv',
  NumModalityPerCluster) are available and the goal is to test whether one cell type
  (e.g. choroid plexus epithelium) shows distinctive isoform regulation versus another
  via pairwise DIE tests.
when_to_use_negative:
- Input is raw fastq or unaligned long reads — barcode deconvolution, alignment (STARlong/Minimap2),
  MapAndFilter, and IsoQuant must run first before pairwise testing is possible
- The question is brain-region-level (HIPP vs PFC) rather than cell-type-pairwise
  isoform testing, or the analysis requires exon-level quantification (ExonQuant)
  rather than isoform/TSS modalities
- Cell types being compared lack the minimum read support (minNumReads=25) for reliable
  proportion-based isoform testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: scisorseqr
  role: Provides DiffSplicingAnalysis for pairwise differential isoform/TSS testing
    between cell types, taking a config file and minimum-read threshold
  repo: github.com/noush-joglekar/scisorseqr
- name: IsoQuant
  role: Sister package producing the per-cell-type isoform, TSS, and polyA modality
    count tables that feed the pairwise comparisons
  repo: github.com/ablab/isoquant
- name: R
  role: Language of the scisorseqr package; used to run DiffSplicingAnalysis and post-process
    results tables into DIE-fraction and TSS-direction summaries
- name: pandas
  role: Python library for reading and reshaping the 'Modality-ModalityID.csv' count
    tables and writing CSV summaries
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:2326b628fe3ee8a7eaaa830a81925130637bcd389096aff8349811013098f718
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DiffSplicingAnalysis
  - IsoQuant
  - R
  - pandas
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pairwise-celltype-comparison-statistics

## Summary

Statistical testing of differential isoform, TSS, and polyA-site usage between pairs of cell types using scisorseqr's DiffSplicingAnalysis on IsoQuant-derived modality count tables, with DIE quantified as deltaPI fractions and directionality assessed via Bernoulli enrichment (e.g. the CPE upstream-TSS signature, 70/93 genes, p = 3e-7).

## When to use

Use when per-cell-type isoform or TSS count tables (e.g. 'Modality-ModalityID.csv', NumModalityPerCluster) are available and the goal is to test whether one cell type (e.g. choroid plexus epithelium) shows distinctive isoform regulation versus another via pairwise DIE tests.

## When NOT to use

- Input is raw fastq or unaligned long reads — barcode deconvolution, alignment (STARlong/Minimap2), MapAndFilter, and IsoQuant must run first before pairwise testing is possible
- The question is brain-region-level (HIPP vs PFC) rather than cell-type-pairwise isoform testing, or the analysis requires exon-level quantification (ExonQuant) rather than isoform/TSS modalities
- Cell types being compared lack the minimum read support (minNumReads=25) for reliable proportion-based isoform testing

## Inputs

- Per-cell-type isoform and TSS count tables (IsoQuantOutput-style 'Modality-ModalityID.csv' and NumModalityPerCluster files) from the isoformAtlas deposit
- Tab-separated config file of pairwise cell-type comparisons in Comparison1/Celltypes/Comparison2/Celltypes format
- List of genes of interest for directional enrichment (e.g. the 93 CPE-relevant genes)

## Outputs

- Per-comparison DiffSplicingAnalysis results tables (Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR)
- Pairwise DIE-fraction table per cell-type comparison at deltaPI=0.1
- TSS-direction summary (upstream/downstream counts and Bernoulli p-value, e.g. cpe_tss_direction_summary.csv)

## How to apply

Load the deposited per-cell-type isoform and TSS count tables from the isoformAtlas deposit, then write a tab-separated config file listing each pairwise hippocampal cell-type comparison in the Comparison1/Celltypes/Comparison2/Celltypes format. Run scisorseqr's DiffSplicingAnalysis with typeOfTest set to 'Iso' or 'TSS', minNumReads=25, and is.hier=FALSE to obtain per-comparison results tables with columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, and FDR. Classify a gene as differentially isoform-expressed using the deltaPI=0.1 threshold, then compute pairwise DIE fractions per cell-type comparison. For transcript-feature attribution (TSS vs splice-site vs polyA-site), run the test separately per feature type and compare which feature explains the cell-type-specific isoform regulation. Directionality of TSS shifts is assessed by classifying shifts as upstream/downstream in the gene set of interest (e.g. 93 CPE-relevant genes) and evaluating Bernoulli enrichment of the upstream direction against a 50/50 expectation.

## Related tools

- **scisorseqr** (Provides DiffSplicingAnalysis for pairwise differential isoform/TSS testing between cell types, taking a config file and minimum-read threshold) — github.com/noush-joglekar/scisorseqr
- **IsoQuant** (Sister package producing the per-cell-type isoform, TSS, and polyA modality count tables that feed the pairwise comparisons) — github.com/ablab/isoquant
- **R** (Language of the scisorseqr package; used to run DiffSplicingAnalysis and post-process results tables into DIE-fraction and TSS-direction summaries)
- **pandas** (Python library for reading and reshaping the 'Modality-ModalityID.csv' count tables and writing CSV summaries)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'TSS', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- cpe_tss_direction_summary.csv exists, contains TSS-direction fields, and reports 70/93 upstream-TSS with Bernoulli p <= 3e-7
- DiffSplicingAnalysis results tables contain the expected columns: Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- All pairwise comparisons in the config file produce a results table; genes below minNumReads=25 are excluded
- DIE classification is reproducible at the deltaPI=0.1 threshold across feature types (Iso, TSS, PolyA)
- FDR-corrected p-values fall within [0,1] and dPI within [-1,1] for every comparison

## Limitations

- Isoform-matrix sparsity due to lower long-read throughput can cause reproducibility discrepancies in cell-type clustering
- Observed DIE patterns in P7 mouse brain have not been shown to persist in disease case-control studies, across adult brain regions, or across species
- Traditional transcriptional cell-type definitions may not correspond to isoform expression programs, so cell-type labels may need isoform-based re-definition

## Evidence

- [methods] DiffSplicingAnalysis performs pairwise statistical tests: "DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)"
- [methods] IsoQuant generates modality counts: "IsoQuant(AllInfoFile = 'LongReadInfo/AllInfo',
          Iso = TRUE, TSS = TRUE, PolyA = TRUE)"
- [abstract] CPE shows distinctive TSS usage: "Cell types indigenous to one anatomic structure display distinctive DIE, e.g. the choroid plexus epithelium manifests distinct transcription-start-site usage."
- [readme] scisorseqr scope: [evidence span withheld — anchor status: external_doc]
