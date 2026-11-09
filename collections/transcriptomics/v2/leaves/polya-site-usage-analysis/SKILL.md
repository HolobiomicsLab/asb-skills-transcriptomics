---
name: polya-site-usage-analysis
description: Use when long-read (PacBio/ONT) reads are annotated with polyA sites
  against a reference such as the polyasite.unibas.ch atlas or FANTOM CAGE peaks (mm10),
  and the goal is to find genes whose 3' end site choice differs between biological
  groups.
when_to_use_negative:
- Input is a bulk short-read RNA-seq quantification without full-length read polyA-site
  annotation — the chi-square framework needs per-read 3' end assignments against
  a polyA-site atlas
- The goal is cell-type-resolved hierarchical DIE testing (is.hier = TRUE tree traversal)
  rather than a pairwise bulk-condition polyA comparison
- Genes with fewer than minNumReads = 25 reads per condition — the chi-square test
  is underpowered and these genes are filtered out
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scisorseqr
  role: Runs IsoQuant with PolyA = TRUE to assign polyA sites per long read and DiffSplicingAnalysis
    with minNumReads = 25 for gene-wise chi-square testing of polyA-site proportions
  repo: github.com/noush-joglekar/scisorseqr
- name: R
  role: Platform for the scisorseqr R-package used to run the polyA-site usage tests
    and BH correction
- name: minimap2
  role: Spliced long-read alignment of ONT (or PacBio) reads to mm10 prior to polyA-site
    annotation
  repo: github.com/lh3/minimap2
- name: STARlong
  role: Spliced alignment of PacBio long reads to the reference genome prior to polyA-site
    annotation
  repo: github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:edbcbef76170cd1a0b143942ce422ccb0f9eab4d9f33d4f2b4c25d8f2e19ee88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - STARlong
  - minimap2
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# polya-site-usage-analysis

## Summary

Quantifies per-gene polyadenylation (polyA) site usage from long-read ScISOrSeq data and tests for differential polyA-site usage between conditions (e.g., hippocampus vs prefrontal cortex) using gene-wise n x 2 chi-square tests with FDR and delta-PI thresholds.

## When to use

Use when long-read (PacBio/ONT) reads are annotated with polyA sites against a reference such as the polyasite.unibas.ch atlas or FANTOM CAGE peaks (mm10), and the goal is to find genes whose 3' end site choice differs between biological groups.

## When NOT to use

- Input is a bulk short-read RNA-seq quantification without full-length read polyA-site annotation — the chi-square framework needs per-read 3' end assignments against a polyA-site atlas
- The goal is cell-type-resolved hierarchical DIE testing (is.hier = TRUE tree traversal) rather than a pairwise bulk-condition polyA comparison
- Genes with fewer than minNumReads = 25 reads per condition — the chi-square test is underpowered and these genes are filtered out

## Inputs

- Full-length spliced long-read FASTQ/BAM alignments (PacBio via STARlong, ONT/PacBio via minimap2)
- Per-read info table from scisorseqr's InfoPerLongRead (barcodes, gene/isoform assignments, polyA-site calls)
- Annotated polyA-site reference (polyasite.unibas.ch atlas) and FANTOM CAGE peaks (mm10)
- Condition/group metadata (e.g., bulk hippocampus vs prefrontal cortex per-gene polyA count table from www.isoformAtlas.com)

## Outputs

- Per-gene polyA-site count table per condition
- DiffSplicingAnalysis results TSV with per-gene chi-square p-values, BH-adjusted FDR, and delta-PI for polyA-site usage
- List of genes with significant differential polyA-site usage (FDR <= 0.05, delta-PI >= 0.1)

## How to apply

Annotate full-length spliced long reads with polyA sites using scisorseqr's IsoQuant step (PolyA = TRUE), which leverages annotated polyA sites (polyasite.unibas.ch atlas, FANTOM CAGE peaks mm10) rather than raw read ends. Build per-gene polyA-site count tables per condition, then run DiffSplicingAnalysis with typeOfTest set to the polyA test and minNumReads = 25, is.hier = FALSE, which performs per-gene n x 2 chi-square tests on site proportions between groups. Apply Benjamini-Hochberg FDR <= 0.05 correction and require delta-PI (change in percent-isoform/site usage) >= 0.1 to call differential polyA-site usage. In the mouse P7 brain study, repeating this framework on bulk HIPP vs PFC tables showed 141 of the 395 differentially isoform-expressed genes (out of 3958 tested) owed their DIE to differential TSS or polyA-site usage.

## Related tools

- **scisorseqr** (Runs IsoQuant with PolyA = TRUE to assign polyA sites per long read and DiffSplicingAnalysis with minNumReads = 25 for gene-wise chi-square testing of polyA-site proportions) — github.com/noush-joglekar/scisorseqr
- **R** (Platform for the scisorseqr R-package used to run the polyA-site usage tests and BH correction)
- **minimap2** (Spliced long-read alignment of ONT (or PacBio) reads to mm10 prior to polyA-site annotation) — github.com/lh3/minimap2
- **STARlong** (Spliced alignment of PacBio long reads to the reference genome prior to polyA-site annotation) — github.com/alexdobin/STAR

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'PolyA', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- Bulk HIPP-vs-PFC reproduction must yield 395 significant DIE genes out of 3958 tested (FDR <= 0.05, delta-PI >= 0.1) at the gene level
- 141 of the 395 DIE genes must be flagged as attributable to differential TSS or polyA-site usage under the same chi-square framework
- No gene with fewer than minNumReads = 25 reads appears in the significance table
- All reported delta-PI values are >= 0.1; all reported FDR values are <= 0.05 after Benjamini-Hochberg correction
- PolyA-site assignments map to entries in the polyasite.unibas.ch atlas / FANTOM CAGE peaks (mm10), not arbitrary read-end coordinates

## Limitations

- PolyA-site usage signals depend on long-read throughput; sparsity of the isoform/polyA matrix can affect power and reproducibility
- Observed polyA-site usage patterns are established only for P7 mouse HIPP/PFC; persistence in adult brain, disease case-control studies, and other species is untested
- Only genes passing minNumReads = 25 are testable, so lowly expressed genes with differential polyA usage are missed

## Evidence

- [methods] DiffSplicingAnalysis is configured with typeOfTest and minNumReads thresholds: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
- [methods] IsoQuant annotates TSS and polyA sites per long read: "IsoQuant(AllInfoFile = 'LongReadInfo/AllInfo',
          Iso = TRUE, TSS = TRUE, PolyA = TRUE)"
- [discussion] Published result on regional DIE and its TSS/polyA attribution: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [methods] PolyA sites atlas used as annotation source: "[PolyA sites](https://polyasite.unibas.ch/atlas) for our study"
- [readme] scisorseqr README describes package purpose: [evidence span withheld — anchor status: external_doc]
