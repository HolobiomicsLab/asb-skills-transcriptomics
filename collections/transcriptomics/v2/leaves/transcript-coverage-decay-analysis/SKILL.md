---
name: transcript-coverage-decay-analysis
description: Use when evaluating whether a 10x Chromium 3' (v2) or 5' (v1) workflow
  supports calling variants distal to the capture end, or when reproducing coverage-decay
  curves from per-cell UMI counts.
when_to_use_negative:
- Full-length cDNA platforms such as Fluidigm C1/SMART-seq, where coverage is not
  end-biased and decay-from-capture-end analysis does not apply
- Transcriptomes containing multi-isoform genes, where a single unambiguous TSS/TTS
  cannot be assigned per gene (the analysis restricts to single-isoform genes)
- Determining whether an individual cell is truly wild-type for a mutation — dropout
  makes this impossible regardless of coverage modeling
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: python3
  role: Analysis language used to collapse UMI counts and aggregate coverage by distance
    from transcript ends
- name: pysam
  role: Python library used with python3 to collapse reads to unique cell-barcode/UMI
    pairs per genomic position
- name: sridnona/python3:180925.v1
  role: Docker image providing the pinned python3/pysam environment for the coverage
    analysis
- name: 10x Genomics Chromium Single Cell 5' (v1) Gene Expression
  role: Library kit generating 5'-captured scRNA-seq data whose coverage is quantified
    relative to the TSS
- name: 10x Genomics Chromium Single Cell 3' (v2) Gene Expression
  role: Library kit generating 3'-captured scRNA-seq data whose coverage is quantified
    relative to the TTS for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:9b50ac5827e01bb9e67494a4d0daac2b6bddb4dd44292ec2da8e01e9789fb9fd
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# transcript-coverage-decay-analysis

## Summary

Quantifies how single-cell transcript coverage decays as a function of distance from the capture site (TSS for 5' kits, TTS for 3' kits) in 10x Chromium scRNA-seq data, to establish how far from the transcript end variant detection remains feasible.

## When to use

Use when evaluating whether a 10x Chromium 3' (v2) or 5' (v1) workflow supports calling variants distal to the capture end, or when reproducing coverage-decay curves from per-cell UMI counts.

## When NOT to use

- Full-length cDNA platforms such as Fluidigm C1/SMART-seq, where coverage is not end-biased and decay-from-capture-end analysis does not apply
- Transcriptomes containing multi-isoform genes, where a single unambiguous TSS/TTS cannot be assigned per gene (the analysis restricts to single-isoform genes)
- Determining whether an individual cell is truly wild-type for a mutation — dropout makes this impossible regardless of coverage modeling

## Inputs

- Per-cell UMI count tables (_counts_UB.tsv) from 10x Chromium Single Cell 5' (v1) and 3' (v2) Gene Expression libraries for sample UPN 508084
- Gene annotations for 20,090 single-isoform genes with strand and TSS/TTS coordinates
- CellRanger-aligned BAM files (for read-to-molecule collapse via pysam)

## Outputs

- Per-bin molecule counts as a fraction of total molecules per base, relative to distance from TSS/TTS, computed separately for the 5' and 3' kits
- Coverage-decay curves (coverage vs. signed distance from transcript end) plotted side by side for both kits, extending to at least 10 kbp

## How to apply

Load the per-cell UMI tables (_counts_UB.tsv, counts of the Chromium molecular barcode UB_tag) generated from the 5' (v1) and 3' (v2) Chromium libraries. In a python3 session using pysam inside the Docker image sridnona/python3:180925.v1, collapse reads to unique cell-barcode/UMI pairs per genomic position within the 20,090 single-isoform genes. Annotate each counted molecule with its signed distance from the gene's transcription start site (5' kit) or transcription termination site (3' kit), orienting by gene strand. Aggregate molecule counts into genomic-position bins relative to the transcript end and normalize to the fraction of total molecules per base for each kit, then plot coverage vs. distance out to at least 10 kbp (Fig. 1b style decay curve). The rationale is that Chromium is end-biased, and non-linear coverage decay determines which SNVs are detectable; validating that both kits show consistent distal coverage justifies end-biased capture for expressed-mutation detection.

## Related tools

- **python3** (Analysis language used to collapse UMI counts and aggregate coverage by distance from transcript ends)
- **pysam** (Python library used with python3 to collapse reads to unique cell-barcode/UMI pairs per genomic position)
- **sridnona/python3:180925.v1** (Docker image providing the pinned python3/pysam environment for the coverage analysis)
- **10x Genomics Chromium Single Cell 5' (v1) Gene Expression** (Library kit generating 5'-captured scRNA-seq data whose coverage is quantified relative to the TSS)
- **10x Genomics Chromium Single Cell 3' (v2) Gene Expression** (Library kit generating 3'-captured scRNA-seq data whose coverage is quantified relative to the TTS for comparison)

## Examples

```
docker run -v $(pwd):/work sridnona/python3:180925.v1 python3 -c "import pysam; sam=pysam.AlignmentFile('/work/possorted_genome_bam.bam','rb'); print(sum(1 for _ in sam))"
```

## Evaluation signals

- Both 5' and 3' workflows exhibit consistent coverage extending at least 10 kbp from the transcript end
- For the average gene, the 5' kit retains at least 2.5% of unique sequenced transcripts per base at positions up to 10 kbp distal to the 5' transcription start site
- Coverage is aggregated only over the 20,090 single-isoform genes and each molecule is oriented by gene strand so distances are signed relative to the correct transcript end
- Counts are collapsed to unique cell-barcode/UMI pairs per position, so duplicate PCR reads do not inflate the per-base coverage fractions
- The decay curve shape matches the Fig. 1b non-linear coverage-decay pattern reported for the Chromium platform

## Limitations

- Partial transcript coverage is specific to end-biased platforms such as the Chromium platform, and limits the sensitivity of variant detection
- Coverage drops non-linearly across transcript length, so some variants are much more easily detectable than others
- Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms and reduces the sensitivity of mutation detection

## Evidence

- [methods] Per-cell UMI tables are the raw input for the decay analysis: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [methods] The analysis environment is a pinned Docker image with python3 and pysam: "Docker image
----------
* sridnona/python3:180925.v1"
- [discussion] Both Chromium kits yield unexpectedly distal coverage, motivating the decay quantification: "the 10x Genomics Chromium Single Cell 3′ and 5′ Gene Expression workflows yield unexpectedly high transcript coverage far from the 3′ and 5′ ends of transcripts"
- [discussion] End-biased coverage limits variant detection sensitivity: "Partial transcript coverage is specific to end-biased platforms such as the Chromium platform, and also limits the sensitivity of variant detection."
- [results] Quantitative benchmark for the 5' kit coverage decay: [evidence span withheld — anchor status: not_found]
