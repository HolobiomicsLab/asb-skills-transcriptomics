---
name: delta-psi-calculation
description: Use when comparing alternative-splicing changes between two regions/conditions
  (e.g. hippocampus vs prefrontal cortex) at exon level, especially when validating
  that single-cell ScISOrSeq dPSI values reproduce in spatial long-read data.
when_to_use_negative:
- When the goal is differential isoform (transcript-level) testing rather than exon-level
  PSI differences — the article reports that isoform tests for DIE show better performance
  than exon tests.
- When exon quantifications have insufficient read support per group (below the scisorseqr
  threshold = 10 reads), as PSI estimates will be unreliable.
- When the comparison set of exons is not defined/non-overlapping — dPSI concordance
  testing requires a fixed exon list shared across modalities.
edam_operation: http://edamontology.org/operation_0446
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scisorseqr
  role: R package used for ExonQuant quantification of per-celltype/per-region exon
    inclusion and DiffSplicingAnalysis of alternative splicing patterns
  repo: https://github.com/noush-joglekar/scisorseqr
- name: R
  role: Implementation environment for scisorseqr, dPSI computation, correlation,
    and Bernoulli significance testing
- name: pandas
  role: Python library used to assemble the per-exon concordance table and write exon_delta_psi_concordance.csv
- name: Minimap2
  role: Splice-aware alignment of ONT spatial long reads to the reference genome,
    upstream of exon quantification
  repo: https://github.com/lh3/minimap2
- name: STARlong
  role: Alternative aligner for PacBio long reads feeding the scisorseqr quantification
    pipeline
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:f4fab13cf3b049f526b72094e7aa4c64abcb8d6babc34acd3be60edc1305a8f2
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
  edam_operation: http://edamontology.org/operation_0446
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# delta-psi-calculation

## Summary

Compute per-exon delta-PSI (percent-spliced-in difference between conditions, e.g. HIPP minus PFC) from scisorseqr exon quantifications, and compare dPSI vectors across data modalities (single-cell vs spatial long-read) to test that region-specific splicing changes reproduce.

## When to use

Use when comparing alternative-splicing changes between two regions/conditions (e.g. hippocampus vs prefrontal cortex) at exon level, especially when validating that single-cell ScISOrSeq dPSI values reproduce in spatial long-read data.

## When NOT to use

- When the goal is differential isoform (transcript-level) testing rather than exon-level PSI differences — the article reports that isoform tests for DIE show better performance than exon tests.
- When exon quantifications have insufficient read support per group (below the scisorseqr threshold = 10 reads), as PSI estimates will be unreliable.
- When the comparison set of exons is not defined/non-overlapping — dPSI concordance testing requires a fixed exon list shared across modalities.

## Inputs

- Single-cell HIPP-vs-PFC exon quantifications from scisorseqr ExonQuant (groupingFactor e.g. 'Celltype', threshold = 10)
- Deposited 10X Visium spatial counts with ONT spatial long-read alignments for the same brain regions
- Curated list of non-overlapping region-specific exons (40 exons in the reference workflow)
- Reference annotation (e.g. GENCODE GTF) for exon definitions

## Outputs

- Per-exon delta-PSI values (HIPP minus PFC) for single-cell and spatial modalities
- Concordance table exon_delta_psi_concordance.csv with columns: exon ID, single-cell dPSI, spatial dPSI, same-direction flag
- Pearson r^2 between single-cell and spatial dPSI vectors and two-sided Bernoulli test p-value for same-direction fraction

## How to apply

Quantify exon inclusion with scisorseqr's ExonQuant, grouping reads by the factor of interest (e.g. Celltype or region) with a read-support threshold (the article uses threshold = 10). From the quantifications, compute per-exon delta-PSI as the PSI difference between conditions (HIPP minus PFC) for a defined non-overlapping set of region-specific exons (40 in the reference task). Independently compute spatial delta-PSI for the same exons from 10X Visium counts aligned with ONT long reads. Then correlate the single-cell and spatial dPSI vectors with Pearson correlation (expected r^2 ≈ 0.6) and count same-direction changes, testing significance with a two-sided Bernoulli test (85/40 concordance gives p ≤ 3.5e-06). Write the per-exon concordance table (exon ID, single-cell dPSI, spatial dPSI, same-direction flag) to exon_delta_psi_concordance.csv and validate it has exactly 40 rows, ~0.85 same-direction fraction, and r^2 in [0.5, 0.7].

## Related tools

- **scisorseqr** (R package used for ExonQuant quantification of per-celltype/per-region exon inclusion and DiffSplicingAnalysis of alternative splicing patterns) — https://github.com/noush-joglekar/scisorseqr
- **R** (Implementation environment for scisorseqr, dPSI computation, correlation, and Bernoulli significance testing)
- **pandas** (Python library used to assemble the per-exon concordance table and write exon_delta_psi_concordance.csv)
- **Minimap2** (Splice-aware alignment of ONT spatial long reads to the reference genome, upstream of exon quantification) — https://github.com/lh3/minimap2
- **STARlong** (Alternative aligner for PacBio long reads feeding the scisorseqr quantification pipeline) — https://github.com/alexdobin/STAR

## Examples

```
ExonQuant(AllInfoFile = 'LongReadInfo/AllInfo', groupingFactor = "Celltype", threshold = 10)
```

## Evaluation signals

- Concordance table has exactly 40 rows (one per region-specific exon).
- Same-direction fraction ≈ 0.85 with two-sided Bernoulli test p ≤ 3.5e-06.
- Pearson r^2 between single-cell and spatial dPSI vectors falls in [0.5, 0.7] (expected ≈ 0.6).
- Each exon appears with both a single-cell and a spatial dPSI value and a boolean same-direction flag.
- Exon quantification inputs meet the scisorseqr read-support threshold (threshold = 10).

## Limitations

- Long-read data sparsity: lower long-read throughput makes the isoform/exon matrix sparse, which can drive discrepancies between modalities.
- Exon-level tests underperform isoform-level tests for differential isoform expression in this dataset.
- Concordance was established for P7 mouse HIPP vs PFC only; generalization to adult regions, disease case-control studies, or other species is untested.

## Evidence

- [methods] ExonQuant call with grouping and threshold parameters: "ExonQuant(AllInfoFile = 'LongReadInfo/AllInfo', 
          groupingFactor = "Celltype", threshold = 10)"
- [readme] scisorseqr purpose and applicability to spatial exon expression: "The tools are also applicable to long-read spatial transcriptomics, and
can be used to resolve exon expression at the
[spatial](https://isoformatlas.com/) level"
- [abstract] Spatial splicing map from spatial transcriptomics plus long reads: "Spatial transcriptomics and long-read sequencing yield a spatially resolved splicing map."
- [discussion] Sparsity limitation of the long-read isoform matrix: "which could be partially due to the sparsity of the isoform matrix due to lower long-read throughput"
- [abstract] Exon tests vs isoform tests performance: "Isoform tests for DIE show better performance than exon tests."
