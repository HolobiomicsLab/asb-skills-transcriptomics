---
name: minigene-splicing-isoform-validation
description: Use when you have nanopore long-read cDNA from a single-locus minigene
  (e.g. PAX6) splicing assay and need per-isoform detection, abundance and length
  estimates, plus a cross-check of splice-site calls against a transcriptome-based
  tool.
when_to_use_negative:
- Input is whole-transcriptome RNA-seq rather than a single-locus minigene assay —
  use a reference-guided transcriptome workflow (e.g. bambu-based wf-transcriptomes)
  instead
- Only short-read data is available — VIsoQLR is designed for long-read sequencing
  data and will not detect the full spectrum of splicing events
- Alignment was produced with Minimap2 on a locus containing small exons — realign
  with GMAP before isoform calling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIsoQLR
  role: Interactive analyzer/viewer/editor that defines consensus splice sites from
    the aligned BAM and quantifies isoforms with a tunable read threshold
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: cDNA aligner used to map minigene FASTQ reads to the minigene reference and
    produce the BAM/BAI input for VIsoQLR
- name: StringTie2
  role: Transcriptome-based comparison tool run on the same BAM to cross-validate
    isoform counts, abundances and lengths
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:47059c00b1ad6721e20f97354f32b0d3f5d673de474bccf89fdffb1ffdbcf034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - StringTie2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# minigene-splicing-isoform-validation

## Summary

Detects, quantifies and cross-validates isoforms from a single-gene minigene splicing assay sequenced with Oxford Nanopore long reads, using VIsoQLR (Docker) on GMAP-aligned BAM input and comparing results against StringTie2, as demonstrated on the PAX6 minigene case study.

## When to use

Use when you have nanopore long-read cDNA from a single-locus minigene (e.g. PAX6) splicing assay and need per-isoform detection, abundance and length estimates, plus a cross-check of splice-site calls against a transcriptome-based tool.

## When NOT to use

- Input is whole-transcriptome RNA-seq rather than a single-locus minigene assay — use a reference-guided transcriptome workflow (e.g. bambu-based wf-transcriptomes) instead
- Only short-read data is available — VIsoQLR is designed for long-read sequencing data and will not detect the full spectrum of splicing events
- Alignment was produced with Minimap2 on a locus containing small exons — realign with GMAP before isoform calling

## Inputs

- Nanopore long-read FASTQ from a minigene splicing assay (or pre-aligned BAM plus BAI index)
- Minigene reference sequence (FASTA) for GMAP index building
- Optional: known isoform references (GFF3/BED6/GTF) for comparison

## Outputs

- VIsoQLR ranked isoform abundance table with isoform lengths (per gene)
- StringTie2 isoform abundance table from the same BAM
- Side-by-side comparison table (e.g. pax6_minigene_isoform_abundances.tsv)

## How to apply

Retrieve the minigene nanopore reads and reference (e.g. Supplementary Files S4, Tables S8/S9 of the PAX6 study); if starting from raw FASTQ, build a GMAP reference index from the minigene reference sequence and align with GMAP to obtain a BAM plus BAI index, preferring GMAP over Minimap2 because Minimap2 has documented problems with small exons. Launch VIsoQLR via its Docker container with port mapping 8888:8888, upload the BAM (GFF3 or BED6 also accepted), and run with default parameters, including the 2% default read threshold for consensus exon coordinates. Record VIsoQLR's ranked isoform abundances and lengths (PAX6 expectation: 40 isoforms, 9 above 1%, canonical isoform at 53.1% and 776 bp). Run StringTie2 on the same BAM for comparison, noting its boundary-extension artifact and missed-exon isoforms (PAX6 expectation: 58 isoforms, 8 above 1%, canonical at 60.5% and 816 bp). Tabulate both tools' ranked abundances and lengths side by side for validation.

## Related tools

- **VIsoQLR** (Interactive analyzer/viewer/editor that defines consensus splice sites from the aligned BAM and quantifies isoforms with a tunable read threshold) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (cDNA aligner used to map minigene FASTQ reads to the minigene reference and produce the BAM/BAI input for VIsoQLR)
- **StringTie2** (Transcriptome-based comparison tool run on the same BAM to cross-validate isoform counts, abundances and lengths)

## Examples

```
docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- VIsoQLR isoform count for the PAX6 minigene is 40 with 9 isoforms above 1% abundance
- VIsoQLR canonical isoform matches 53.1% abundance at 776 bp; StringTie2 canonical matches 60.5% at 816 bp
- Abundance ranking of isoforms correlates with the independent electropherogram/capillary method
- StringTie2 boundary extension beyond the BAM-supported read ends flags an artifact, not a real isoform variant
- Comparison table contains ranked isoform abundances and lengths for both tools, non-empty for all above-threshold isoforms

## Limitations

- Missing isoforms are usually reads below the default 2% CEC threshold; lowering it detects more isoforms but manual parameter tuning and close inspection are often required
- StringTie2 extends isoform boundaries and does not call isoforms lacking certain exons, so it can miss real isoforms (e.g. Iso4 and Iso9 in the PAX6 minigene)
- Low-abundance isoforms (~4% and ~1.7%) may be overrepresented by LRS and can be hard to distinguish from noise in the electropherogram
- No software version numbers were reported for VIsoQLR, StringTie2, or FLAIR in the benchmark

## Evidence

- [methods] Upload aligned reads (GFF3, BED6, or BAM): "navigate to your file system to upload your aligner reads in `GFF3`, `BED6`, or `BAM` file format."
- [discussion] Default 2% threshold trade-off: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [discussion] StringTie2 artifact: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [readme] Minigene scope of the pipeline: [evidence span withheld — anchor status: external_doc]
- [discussion] SIRV benchmark accuracy: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
