---
name: spatial-transcriptomics-splicing-concordance
description: Use when validating that exon-level splicing changes (delta-PSI) found
  between brain regions in single-cell ScISOrSeq data reproduce in spatial transcriptomics
  data with ONT long-read alignments, e.g. HIPP vs PFC on a defined exon set.
when_to_use_negative:
- The analysis goal is differential isoform expression testing itself (use scisorseqr
  DiffSplicingAnalysis with isoform tests, which outperform exon tests), rather than
  cross-platform concordance of already-derived delta-PSI values.
- Input is short-read-only spatial data without ONT (or PacBio) long-read alignments,
  since exon/isoform resolution cannot be derived from short reads alone.
- Only a handful of exons are available — the Bernoulli significance assessment (e.g.
  85% of 40 exons, p <= 3.5e-06) requires a sufficiently large exon set.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: scisorseqr
  role: R package used to quantify exon expression per cell type/region from long
    reads (ExonQuant) and applicable to long-read spatial transcriptomics to resolve
    exon expression at the spatial level
  repo: https://github.com/noush-joglekar/scisorseqr
- name: Minimap2
  role: Splice-aware alignment of ONT long reads for the spatial transcriptomics data
  repo: https://github.com/lh3/minimap2
- name: STARlong
  role: Alternative splice-aware aligner for PacBio long reads
  repo: https://github.com/alexdobin/STAR
- name: pandas
  role: Python library used for the delta-PSI computation, correlation and concordance
    table assembly
- name: R
  role: Language of the scisorseqr package and downstream statistical analysis (Bernoulli
    test, Pearson correlation)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:57469334597a7e8ea633e231f03badb5fcaa5dcdea710052c677712e403fe4e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - Minimap2
  - R
  - STARlong
  - pandas
  - scisorseqr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spatial-transcriptomics-splicing-concordance

## Summary

Reproduce region-specific splicing differences measured by single-cell long-read sequencing at spatial resolution by correlating single-cell and spatial (10X Visium + ONT long-read) delta-PSI values per exon, as demonstrated for HIPP-vs-PFC comparisons in the postnatal day 7 mouse brain.

## When to use

Use when validating that exon-level splicing changes (delta-PSI) found between brain regions in single-cell ScISOrSeq data reproduce in spatial transcriptomics data with ONT long-read alignments, e.g. HIPP vs PFC on a defined exon set.

## When NOT to use

- The analysis goal is differential isoform expression testing itself (use scisorseqr DiffSplicingAnalysis with isoform tests, which outperform exon tests), rather than cross-platform concordance of already-derived delta-PSI values.
- Input is short-read-only spatial data without ONT (or PacBio) long-read alignments, since exon/isoform resolution cannot be derived from short reads alone.
- Only a handful of exons are available — the Bernoulli significance assessment (e.g. 85% of 40 exons, p <= 3.5e-06) requires a sufficiently large exon set.

## Inputs

- Single-cell HIPP-vs-PFC exon quantifications (from scisorseqr ExonQuant on long-read data, grouped by region/cell type)
- List of non-overlapping region-specific exons (n=40 in the reference task)
- Deposited 10X Visium spatial counts with ONT spatial long-read alignments (mm10 reference genome)

## Outputs

- exon_delta_psi_concordance.csv with columns: exon ID, single-cell dPSI, spatial dPSI, same-direction flag
- Pearson r^2 between single-cell and spatial delta-PSI vectors
- Same-direction fraction with two-sided Bernoulli test p-value

## How to apply

Start from the single-cell HIPP-vs-PFC exon quantifications and select the non-overlapping region-specific exon set (here 40 exons), computing per-exon single-cell delta-PSI as HIPP minus PFC. Then load the deposited 10X Visium spatial counts with ONT spatial long-read alignments and compute per-exon spatial delta-PSI for the same exons. Correlate the two delta-PSI vectors with Pearson correlation (report r^2) and count same-direction changes, testing the same-direction fraction against a two-sided Bernoulli test. Write the per-exon concordance table (exon ID, single-cell dPSI, spatial dPSI, same-direction flag) to CSV. In the reference study, 85% of 40 exons showed same-direction changes (Bernoulli p <= 3.5e-06) and r^2 = 0.6, confirming that the spatially resolved splicing map reproduces single-cell findings.

## Related tools

- **scisorseqr** (R package used to quantify exon expression per cell type/region from long reads (ExonQuant) and applicable to long-read spatial transcriptomics to resolve exon expression at the spatial level) — https://github.com/noush-joglekar/scisorseqr
- **Minimap2** (Splice-aware alignment of ONT long reads for the spatial transcriptomics data) — https://github.com/lh3/minimap2
- **STARlong** (Alternative splice-aware aligner for PacBio long reads) — https://github.com/alexdobin/STAR
- **pandas** (Python library used for the delta-PSI computation, correlation and concordance table assembly)
- **R** (Language of the scisorseqr package and downstream statistical analysis (Bernoulli test, Pearson correlation))

## Examples

```
import pandas as pd; conc = pd.DataFrame({'exon': exons, 'dPSI_sc': sc_dpsi, 'dPSI_sp': sp_dpsi}); conc['same_dir'] = (sc_dpsi * sp_dpsi) > 0; conc.to_csv('exon_delta_psi_concordance.csv', index=False)
```

## Evaluation signals

- The concordance table has exactly 40 rows, one per selected region-specific exon.
- The same-direction fraction is ~0.85 with a two-sided Bernoulli test p-value <= 3.5e-06.
- Pearson r^2 between single-cell and spatial delta-PSI vectors falls in [0.5, 0.7].
- Single-cell and spatial delta-PSI are both computed as HIPP minus PFC for the identical exon IDs, so the table's exon IDs match the input exon list exactly.
- The output CSV contains the required columns: exon ID, single-cell dPSI, spatial dPSI, same-direction flag.

## Limitations

- The reference study tested generalizability only at postnatal day 7; whether the observed patterns persist across adult brain regions, in disease case-control studies, or across species is not established.
- Long-read isoform matrices are sparse due to lower long-read throughput, which can cause reproducibility discrepancies.
- The concordance is exon-level; the paper notes isoform tests for differential isoform expression perform better than exon tests.

## Evidence

- [abstract] Spatial transcriptomics and long-read sequencing yield a spatially resolved splicing map.: "Spatial transcriptomics and long-read sequencing yield a spatially resolved splicing map."
- [discussion] Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [methods] ExonQuant call from methods: "ExonQuant(AllInfoFile = 'LongReadInfo/AllInfo', 
          groupingFactor = "Celltype", threshold = 10)"
- [readme] scisorseqr spatial applicability from README: "The tools are also applicable to long-read spatial transcriptomics, and
can be used to resolve exon expression at the
[spatial](https://isoformatlas.com/) level"
- [methods] scisorseqr differential splicing analysis function: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
