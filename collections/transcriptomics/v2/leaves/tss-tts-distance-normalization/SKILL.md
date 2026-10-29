---
name: tss-tts-distance-normalization
description: Use when profiling how far UMI-collapsed transcript coverage extends
  from transcript ends in 10x Chromium scRNA-seq data (e.g., to justify detecting
  SNVs up to ~10 kbp from the capture site).
when_to_use_negative:
- Genes with multiple isoforms — the distance normalization is restricted to the 20,090
  single-isoform genes, so applying it to multi-isoform genes confounds TSS/TTS assignment
- The goal is genotyping individual cells (e.g., assigning variants to barcodes with
  cb_sniffer or VarTrix) rather than characterizing transcript coverage distribution
- Input is bulk RNA-seq or full-length cDNA data (e.g., Fluidigm C1/SMART-seq) where
  end-biased coverage decay is not the limiting factor
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: python3
  role: Language used to collapse UMI counts, compute distances from TSS/TTS, bin,
    and normalize coverage
- name: pysam
  role: Library used in the python3 session to parse alignments and collapse reads
    to unique cell-barcode/UMI pairs per genomic position
- name: Docker
  role: Container runtime providing the pinned python3/pysam environment (sridnona/python3:180925.v1)
- name: sridnona/python3:180925.v1
  role: Docker image pinning the python3 + pysam environment used for the coverage-decay
    analysis
- name: 10x Genomics Chromium Single Cell 5' (v1) Gene Expression
  role: 5' end-biased library kit whose distal coverage is quantified
- name: 10x Genomics Chromium Single Cell 3' (v2) Gene Expression
  role: 3' end-biased library kit whose distal coverage is compared against the 5'
    kit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:03c00735203f4164cdf71320888362611665d49757171b136bf353f155f73bb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics Chromium Single Cell 3' (v2) Gene Expression
  - 10x Genomics Chromium Single Cell 5' (v1) Gene Expression
  - Docker
  - pysam
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# tss-tts-distance-normalization

## Summary

Quantifies single-cell transcript coverage as a function of signed distance from the transcription start site (TSS) or transcription termination site (TTS) for 10x Chromium 5' (v1) and 3' (v2) libraries, to assess whether end-biased capture retains enough coverage distal to transcript ends for variant detection.

## When to use

Use when profiling how far UMI-collapsed transcript coverage extends from transcript ends in 10x Chromium scRNA-seq data (e.g., to justify detecting SNVs up to ~10 kbp from the capture site).

## When NOT to use

- Genes with multiple isoforms — the distance normalization is restricted to the 20,090 single-isoform genes, so applying it to multi-isoform genes confounds TSS/TTS assignment
- The goal is genotyping individual cells (e.g., assigning variants to barcodes with cb_sniffer or VarTrix) rather than characterizing transcript coverage distribution
- Input is bulk RNA-seq or full-length cDNA data (e.g., Fluidigm C1/SMART-seq) where end-biased coverage decay is not the limiting factor

## Inputs

- Per-cell UMI (_counts_UB.tsv) tables of Chromium molecular barcode (UB_tag) read counts for 5' (v1) and 3' (v2) Chromium libraries from sample UPN 508084
- Gene annotations (TSS/TTS coordinates and strand) for the 20,090 single-isoform genes

## Outputs

- Per-kit coverage-decay table: fraction of unique sequenced molecules per base as a function of signed distance from TSS/TTS
- Side-by-side coverage vs. distance-from-TSS/TTS decay plot (Fig. 1b style) out to at least 10 kbp for the 5' and 3' kits

## How to apply

Load the per-cell UMI count tables (_counts_UB.tsv) generated from the 5' (v1) and 3' (v2) Chromium libraries, parsing Chromium molecular barcode (UB_tag) counts. Using pysam in a python3 session (Docker image sridnona/python3:180925.v1), collapse reads to unique cell-barcode/UMI pairs per genomic position within the 20,090 single-isoform genes. Annotate each counted molecule with its signed distance from the gene's TSS or TTS, orienting by gene strand. Aggregate molecule counts in bins relative to the transcript end and normalize to the fraction of total molecules per base for each kit separately, then plot coverage vs. distance for the two kits side by side out to at least 10 kbp.

## Related tools

- **python3** (Language used to collapse UMI counts, compute distances from TSS/TTS, bin, and normalize coverage)
- **pysam** (Library used in the python3 session to parse alignments and collapse reads to unique cell-barcode/UMI pairs per genomic position)
- **Docker** (Container runtime providing the pinned python3/pysam environment (sridnona/python3:180925.v1))
- **sridnona/python3:180925.v1** (Docker image pinning the python3 + pysam environment used for the coverage-decay analysis)
- **10x Genomics Chromium Single Cell 5' (v1) Gene Expression** (5' end-biased library kit whose distal coverage is quantified)
- **10x Genomics Chromium Single Cell 3' (v2) Gene Expression** (3' end-biased library kit whose distal coverage is compared against the 5' kit)

## Examples

```
docker run -it sridnona/python3:180925.v1 python3 -c "import pysam; ...
```

## Evaluation signals

- For the average gene in the 5' kit, at least 2.5% of unique sequenced transcripts map to any given base up to 10 kbp away from the 5' transcription start site
- Both 5' (v1) and 3' (v2) workflows exhibit consistent coverage extending at least 10 kbp from the transcript end
- Molecules are uniquely assigned to cell-barcode/UMI pairs per genomic position — no duplicate molecule counted at a position
- Signed distances orient correctly by gene strand relative to the TSS or TTS
- Per-base values are normalized to fractions of total molecules per kit, so each coverage curve integrates to 1 within the plotted window

## Limitations

- Partial transcript coverage is specific to end-biased platforms such as the Chromium platform and limits the sensitivity of variant detection
- Coverage drops non-linearly across transcript length, so some variants are much more easily detectable than others
- Analysis is restricted to single-isoform genes; distance-to-TSS/TTS is ambiguous for multi-isoform genes

## Evidence

- [methods] read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [methods] Docker image sridnona/python3:180925.v1: "Docker image
----------
* sridnona/python3:180925.v1"
- [discussion] the 10x Genomics Chromium Single Cell 3′ and 5′ Gene Expression workflows yield unexpectedly high transcript coverage far from the 3′ and 5′ ends of transcripts: "the 10x Genomics Chromium Single Cell 3′ and 5′ Gene Expression workflows yield unexpectedly high transcript coverage far from the 3′ and 5′ ends of transcripts"
- [discussion] Partial transcript coverage is specific to end-biased platforms such as the Chromium platform: "Partial transcript coverage is specific to end-biased platforms such as the Chromium platform, and also limits the sensitivity of variant detection."
- [results] at least 2.5% of unique sequenced transcripts mapped to any given base up to 10 kbp away from the 5' transcription start site: [evidence span withheld — anchor status: not_found]
