---
name: promoter-sequence-extraction
description: Use when csRNA-seq (or TSS-seq) peaks have been called (e.g. with HOMER
  findPeaks on GSE221852 Drosophila embryo or GSE135498 K562/BMDM data) and ±100 bp
  TSS-centered sequences are needed as FASTA input for ElemeNT 2023 element detection.
when_to_use_negative:
- Input is already a FASTA file of candidate promoter sequences — ElemeNT 2023 can
  be run directly without re-extraction.
- The goal is element detection outside a core promoter context (e.g. generic DNA-binding
  site search); ElemeNT 2023 itself can search user-defined motifs, so full TSS-centered
  extraction is not required.
- No called TSS coordinates exist — run peak calling (e.g. HOMER) on the csRNA-seq
  data first, since extraction is anchored on TSS positions.
tools:
- name: HOMER
  role: Calls high-confidence TSS peaks from csRNA-seq data to define the coordinates
    from which promoter sequences are extracted
- name: ElemeNT 2023
  role: Consumes the extracted TSS-centered FASTA sequences to detect core promoter
    elements and positional enrichment
  repo: https://github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:fba79281ea9da2a329fe8f263ad79b3b43a972836e9ad753f753457be38bf19b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT 2023
  - HOMER
  - Perl
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# promoter-sequence-extraction

## Summary

Extracts ±100 bp core promoter sequences centered on called transcription start sites (TSSs) into FASTA format so that core promoter elements (TATA, dInr, PB, DPE, GAGA, Motif 1, dTCT) can be detected with ElemeNT 2023. It is the required input-preparation step between csRNA-seq peak calling and ElemeNT element-position analysis.

## When to use

Use when csRNA-seq (or TSS-seq) peaks have been called (e.g. with HOMER findPeaks on GSE221852 Drosophila embryo or GSE135498 K562/BMDM data) and ±100 bp TSS-centered sequences are needed as FASTA input for ElemeNT 2023 element detection.

## When NOT to use

- Input is already a FASTA file of candidate promoter sequences — ElemeNT 2023 can be run directly without re-extraction.
- The goal is element detection outside a core promoter context (e.g. generic DNA-binding site search); ElemeNT 2023 itself can search user-defined motifs, so full TSS-centered extraction is not required.
- No called TSS coordinates exist — run peak calling (e.g. HOMER) on the csRNA-seq data first, since extraction is anchored on TSS positions.

## Inputs

- BED file of TSS peak coordinates called from csRNA-seq data (e.g. via HOMER findPeaks on GSE221852 or GSE135498)
- Reference genome assembly FASTA (e.g. D. melanogaster genome; human/mouse assemblies for K562/BMDM)

## Outputs

- FASTA file(s) of ±100 bp (201 bp) TSS-centered core promoter sequences, one per TSS, ready for ElemeNT 2023 input

## How to apply

First call high-confidence TSS coordinates from the csRNA-seq data with HOMER findPeaks (e.g. for GSE221852 D. melanogaster embryos or GSE135498 human K562 / mouse BMDM samples), then write the peak coordinates to a BED file. Extract ±100 bp genomic sequences centered on each TSS from the appropriate reference genome assembly using bedtools getfasta into a FASTA file. Feed this FASTA to the ElemeNT 2023 command-line implementation (Perl, from github.com/OritAdato/ElemeNT) to detect TATA, dInr, PB, DPE, GAGA, Motif 1 and dTCT elements. The rationale for the ±100 bp window is that ElemeNT 2023 evaluates preferential motif positioning around the TSS — e.g. dInr at −2 and DPE at +28 — so the window must be large enough to contain downstream elements at +26/+28 yet centered on the annotated TSS for positional scoring.

## Related tools

- **HOMER** (Calls high-confidence TSS peaks from csRNA-seq data to define the coordinates from which promoter sequences are extracted)
- **ElemeNT 2023** (Consumes the extracted TSS-centered FASTA sequences to detect core promoter elements and positional enrichment) — https://github.com/OritAdato/ElemeNT

## Examples

```
bedtools getfasta -fi dm6.fa -bed TSS_peaks.bed -fo promoter_seqs.fasta
```

## Evaluation signals

- Each extracted FASTA record is 201 bp long (±100 bp around the TSS) and its center corresponds to the called TSS coordinate.
- Downstream ElemeNT 2023 analysis shows element frequency peaks at expected positions relative to the TSS: dInr at −2, DPE at +28, Motif 1 at −5, dTCT at −2, PB at +26.
- Median PWM score maxima for each element coincide with these expected positional peaks, indicating the TSS anchoring is correct.
- For cross-species application (human K562, mouse BMDM), detected elements show preferential positional enrichment around the TSS, confirming dataset quality as intended by ElemeNT 2023.

## Limitations

- Extraction quality depends entirely on the accuracy of the upstream TSS peak calls; mis-called TSSs shift the −2/+28 positional frame and degrade ElemeNT 2023 element-position signals.
- The ±100 bp window assumes elements lie within the core promoter region; elements reported at +26/+28 (PB, DPE) sit near the window edge, so larger offsets are not captured.
- The extracted sequences must come from the correct species assembly and strand; ElemeNT 2023's CORE predictions span 10 species (worms to human), so assembly mismatch invalidates the positional analysis.

## Evidence

- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] which provides novel capabilities for assessing evolutionary conservation and for readily evaluating the quality of high-throughput transcription start site (TSS) datasets, leveraging preferential: "which provides novel capabilities for assessing evolutionary conservation and for readily evaluating the quality of high-throughput transcription start site (TSS) datasets, leveraging preferential"
- [readme] It can be executed via the following web interface: [ElemeNT 2023](https://www.juven-gershonlab.org/resources/element-v2023/) or via a command line.: "It can be executed via the following web interface: [ElemeNT 2023](https://www.juven-gershonlab.org/resources/element-v2023/) or via a command line."
