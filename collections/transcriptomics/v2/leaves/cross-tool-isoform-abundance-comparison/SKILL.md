---
name: cross-tool-isoform-abundance-comparison
description: Use when a single-locus long-read splicing assay (e.g. a minigene) has
  been aligned once and you need to compare how two isoform callers (VIsoQLR vs StringTie2)
  disagree on isoform counts, abundances, and isoform lengths.
when_to_use_negative:
- The goal is whole-transcriptome discovery across many samples — VIsoQLR is tailored
  to low-throughput, one-gene-at-a-time splicing assays, not genome-scale transcriptome
  profiling
- The BAM alignment has not yet been produced — first build the GMAP reference index
  and align the FASTQ reads (workflow step, not a comparison step)
- A known reference annotation should guide quantification — VIsoQLR is designed to
  work without prior knowledge of splice sites, so an annotation-guided workflow is
  a different skill
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: VIsoQLR
  role: Interactive analyzer run via Docker (default 2% read threshold) to define
    consensus splice sites and quantify isoforms from the BAM; one side of the comparison
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: StringTie2
  role: Transcriptome-based isoform caller run on the same BAM with default parameters;
    second side of the comparison
- name: GMAP
  role: cDNA-to-reference aligner used to produce the shared BAM/BAI input so both
    tools analyze identical alignments; preferred over Minimap2 for small exons
- name: Minimap2
  role: Alternative aligner explicitly avoided here because of documented problems
    with small exons in minigene data
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:c16aaed5136cce1969a88721b6d4767033f03d96d235365b36aaa68b558e6b7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Minimap2
  - StringTie2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-tool-isoform-abundance-comparison

## Summary

Side-by-side comparison of isoform detection and abundance rankings produced by VIsoQLR and StringTie2 on the same long-read BAM alignment (e.g. the PAX6 minigene splicing assay), to quantify tool-specific differences in isoform counts, abundances, and lengths.

## When to use

Use when a single-locus long-read splicing assay (e.g. a minigene) has been aligned once and you need to compare how two isoform callers (VIsoQLR vs StringTie2) disagree on isoform counts, abundances, and isoform lengths.

## When NOT to use

- The goal is whole-transcriptome discovery across many samples — VIsoQLR is tailored to low-throughput, one-gene-at-a-time splicing assays, not genome-scale transcriptome profiling
- The BAM alignment has not yet been produced — first build the GMAP reference index and align the FASTQ reads (workflow step, not a comparison step)
- A known reference annotation should guide quantification — VIsoQLR is designed to work without prior knowledge of splice sites, so an annotation-guided workflow is a different skill

## Inputs

- BAM alignment (plus BAI index) of long-read cDNA sequences from a single-locus splicing assay, e.g. a PAX6 minigene
- Minigene reference sequence (FASTA) used to build the GMAP index

## Outputs

- Side-by-side TSV of ranked isoform abundances and lengths from both tools (e.g. pax6_minigene_isoform_abundances.tsv)
- Per-tool isoform counts: total isoforms and number above 1% abundance
- Canonical (top-ranked) isoform abundance and length per tool

## How to apply

Align the long-read data once (GMAP is preferred over Minimap2 because Minimap2 has documented problems with small exons) and run both tools on the same BAM. Run VIsoQLR via its Docker container (port 8888:8888) with default parameters, including the 2% default read threshold for consensus exon coordinates, and record the ranked isoform abundances and lengths. Run StringTie2 on the identical BAM with default parameters and record its ranked isoform abundances and lengths. For the PAX6 minigene, expect VIsoQLR to detect 40 isoforms (9 above 1%) with a canonical isoform at 53.1% (776 bp) versus StringTie2's 58 isoforms (8 above 1%) with a canonical isoform at 60.5% (816 bp); also compare second- and third-ranked isoforms (StringTie2: 17.4%/15.0% and 6.3%/6.4%). Tabulate both ranked lists side by side in a TSV and attribute discrepancies to known tool behaviors: StringTie2 extends isoform boundaries (an artifact not present in the BAM files) and fails to call isoforms lacking certain exons.

## Related tools

- **VIsoQLR** (Interactive analyzer run via Docker (default 2% read threshold) to define consensus splice sites and quantify isoforms from the BAM; one side of the comparison) — https://github.com/TBLabFJD/VIsoQLR
- **StringTie2** (Transcriptome-based isoform caller run on the same BAM with default parameters; second side of the comparison)
- **GMAP** (cDNA-to-reference aligner used to produce the shared BAM/BAI input so both tools analyze identical alignments; preferred over Minimap2 for small exons)
- **Minimap2** (Alternative aligner explicitly avoided here because of documented problems with small exons in minigene data) — https://github.com/lh3/minimap2

## Examples

```
docker pull tblabfjd/visoqlr:latest && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest  # then upload the PAX6 BAM; run StringTie2 on the same BAM and tabulate both ranked isoform lists
```

## Evaluation signals

- VIsoQLR on the PAX6 minigene yields 40 isoforms with 9 above 1% abundance and a canonical isoform at 53.1% (776 bp)
- StringTie2 on the same BAM yields 58 isoforms with 8 above 1% and a canonical isoform at 60.5% (816 bp)
- StringTie2's second- and third-ranked isoform abundances are 17.4%/15.0% and 6.3%/6.4%, comparable to VIsoQLR's rankings
- Both tools ran on the identical BAM (same GMAP alignment), so boundary-extension artifacts in StringTie2 can be checked against the BAM as ground truth
- Output table contains both tools' ranked abundances and lengths side by side in a single TSV

## Limitations

- StringTie2 extends isoform boundaries relative to the BAM files (an artifact, also seen in the SIRVs analysis) and does not call isoforms without certain exons (missed Iso4 and Iso9 in the PAX6 comparison)
- Missing isoforms in VIsoQLR's automatic analysis are often reads below the default 2% threshold; lowering it detects more isoforms but requires manual parameter tuning
- Small isoform abundances (e.g. Iso5 at 4% and Iso8 at 1.7%) may be overrepresented by long-read sequencing and are hard to distinguish from noise in electropherograms
- No software version numbers were reported for VIsoQLR, StringTie2, or FLAIR, so exact version reproducibility is not established

## Evidence

- [supplementary] VIsoQLR detects 40 isoforms (nine above 1% abundance) in the PAX6 minigene assay, while StringTie2 detects 58 isoforms (eight above 1%): [evidence span withheld — anchor status: not_found]
- [discussion] StringTie2 boundary extension is an artifact: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [discussion] StringTie2 misses isoforms lacking certain exons: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [methods] Default 2% read threshold defines candidate CECs: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [readme] GMAP preferred over Minimap2 for small exons: [evidence span withheld — anchor status: not_found]
- [readme] VIsoQLR scope: low-throughput splicing assays: "VIsoQLR is tailored to thoroughly analyze mRNA expression and maturation in low-throughput splicing assays."
