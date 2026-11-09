---
name: consensus-exon-coordinate-tuning
description: Use when a VIsoQLR automatic run misses known or expected isoforms for
  a single-gene locus, or when isoform sets must be compared across read-threshold
  values (e.g. 2% vs 3%) on GMAP- or Minimap2-aligned BAM/GFF3/BED6 long-read data.
when_to_use_negative:
- When a fully automated transcriptome-wide analysis is needed rather than close inspection
  of one gene at a time — use reference-guided pipelines like wf-transcriptomes/bambu
  instead
- When reads are unaligned raw FASTQ with no reference index available — build the
  GMAP index and map first, since CEC detection operates on aligned reads
- When the aligner is Minimap2 and the locus contains small exons — Minimap2 has documented
  problems with small exons, so GMAP alignment should be used before tuning CECs
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: VIsoQLR
  role: Interactive analyzer whose Read threshold (%), Padding, and Merge close splice
    sites parameters are tuned to define consensus exon coordinates and quantify isoforms
    per gene
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used to generate the BAM/GFF3 alignments (via a built reference index)
    on which CEC detection runs; preferred over Minimap2 for small exons
- name: Mini-IsoQLR
  role: Command-line pipeline implementing the analogous consensus-breakpoint logic
    (>5% of reads by default) for ONT minigene data
  repo: https://github.com/TBLabFJD/Mini-IsoQLR
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:74b4f37b6ce13020a8ca7648d2cc272c293bb6f0ad76a75a3c50e435d9e445c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Mini-IsoQLR
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Consensus exon coordinate tuning

## Summary

Fine-tunes VIsoQLR's consensus exon coordinate (CEC) detection parameters — Read threshold (%), Padding, and Merge close splice sites — on long-read RNA-seq data to recover isoforms missed by the automatic analysis (default 2% read threshold) and control the trade-off between sensitivity and noise.

## When to use

Use when a VIsoQLR automatic run misses known or expected isoforms for a single-gene locus, or when isoform sets must be compared across read-threshold values (e.g. 2% vs 3%) on GMAP- or Minimap2-aligned BAM/GFF3/BED6 long-read data.

## When NOT to use

- When a fully automated transcriptome-wide analysis is needed rather than close inspection of one gene at a time — use reference-guided pipelines like wf-transcriptomes/bambu instead
- When reads are unaligned raw FASTQ with no reference index available — build the GMAP index and map first, since CEC detection operates on aligned reads
- When the aligner is Minimap2 and the locus contains small exons — Minimap2 has documented problems with small exons, so GMAP alignment should be used before tuning CECs

## Inputs

- Mapped long-read alignments in BAM (plus BAI index), GFF3, or BED6 format, produced by GMAP (preferred for small exons) or Minimap2
- Optional reference FASTA used to build the GMAP index for re-alignment of raw FASTQ reads
- Known isoform references (e.g. Lexogen SIRV transcripts) for comparison of detected vs expected isoforms

## Outputs

- Tuned per-gene isoform table with read counts and abundance percentages per isoform (e.g. visoqlr_uhr_pax6_tp53_isoforms.tsv, pax6_minigene_isoform_abundances.tsv)
- Consensus exon coordinate (CEC/CES) definitions reflecting the chosen threshold, padding, and splice-site merge parameters
- Threshold-sensitivity comparison of isoform detections across read-threshold values

## How to apply

Upload the aligned reads (BAM with BAI, GFF3, or BED6 from GMAP or Minimap2) to VIsoQLR's Docker web interface (port 8888:8888) and, in the Read threshold (%) panel, select candidate CECs by the proportion of exon coordinates among all reads. If expected isoforms are absent, lower the threshold below the 2% default — noting most missing SIRV isoforms fell below 2% — while accepting that a lower threshold detects more isoforms and more noise (e.g. low-abundance isoforms at 4% and 1.7% that are hard to distinguish from electropherogram noise). Use Padding (# of bases) to merge non-candidate CECs just outside the frequency filter on each side of a candidate CES, and Merge close splice sites (# of bases) to collapse candidate CESs closer than the given distance into the most frequent one. Iterate thresholds manually per gene (as in isoform_analysis_visoqlr_perc.R) until detected isoform counts, exon coordinates, and abundances match reference values (e.g. PAX6 45 reads / 9 reads at 45% at a 3% threshold).

## Related tools

- **VIsoQLR** (Interactive analyzer whose Read threshold (%), Padding, and Merge close splice sites parameters are tuned to define consensus exon coordinates and quantify isoforms per gene) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used to generate the BAM/GFF3 alignments (via a built reference index) on which CEC detection runs; preferred over Minimap2 for small exons)
- **Mini-IsoQLR** (Command-line pipeline implementing the analogous consensus-breakpoint logic (>5% of reads by default) for ONT minigene data) — https://github.com/TBLabFJD/Mini-IsoQLR

## Examples

```
docker run -it -p 8888:8888 tblabfjd/visoqlr:latest  # then upload BAM and set Read threshold (%) to 3% in the CEC panel
```

## Evaluation signals

- Detected isoform counts and top-isoform abundances match reference values at a given threshold (e.g. PAX6 45 mapped reads, canonical isoform 9 reads at 45%; TP53 450 reads, top isoform 166 reads at 88.8% at a 3% threshold)
- SIRV benchmark at default parameters reproduces 49/68 isoforms (72%) with 99% exon coordinate accuracy and cosine similarity 0.78 to the Lexogen gold standard
- Lowering the threshold below 2% recovers isoforms previously missing from the automatic analysis, confirming those isoforms' read proportions fell below the default cutoff
- Exon coordinates of detected isoforms agree with the alignments present in the input BAM (no StringTie2-style boundary-extension artifacts)

## Limitations

- Most isoforms missed by the automatic analysis are below the default 2% threshold, and lowering the threshold to detect them increases the number of low-abundance, possibly noise-derived isoforms
- Small isoform abundances (e.g. 4% and 1.7%) may be overrepresented by long-read sequencing and hard to distinguish from electropherogram noise
- Per-gene isoform collection requires close inspection and sometimes manual configuration of parameters — the automatic default run is not sufficient

## Evidence

- [methods] Read threshold selects candidate CECs above a proportion cutoff; lowering recovers isoforms below the 2% default: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [methods] Padding and merge-close-splice-sites parameters govern CEC merging: "The `Padding (# of bases)` are the number of bases at each side of a candidate CES where other non-candidate CEC (below range in the frequency filter) are merged."
- [discussion] Missing isoforms attributed to reads below default threshold; manual tuning needed: "the description of the isoforms collection of individual genes seems to require a close inspection and sometimes a manual configuration of certain parameters"
- [readme] README: command-line analog defines consensus breakpoints above 5% of reads: [evidence span withheld — anchor status: external_doc]
- [discussion] Benchmark validation at default 2% threshold on SIRVs: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
