---
name: spliced-read-alignment-with-uniquely-mapped-filtering
description: 'Use when raw paired-end spatially barcoded ST reads (e.g., MOB ST, SRA:
  SRR3382373) must be mapped to a reference genome before poly(A) site discovery,
  and downstream tools require unambiguous mapping.'
when_to_use_negative:
- Input is already an aligned BAM or a gene/poly(A)-site count matrix — alignment
  and uniquely-mapped filtering are already done
- The analysis requires multi-mapper retention (e.g., repetitive or paralogous regions
  where multimapping evidence matters), since this pipeline discards non-unique alignments
- Data are image-based spatial assays (MERFISH/seqFISH) — the article notes APA sites
  can only be identified from spatially barcoded ST data with tools like scAPAtrap
  or Sierra
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STAR
  role: Splice-aware aligner used to map read 2 to the GRCm38 reference genome, retaining
    only uniquely mapped reads
- name: umi_tools
  role: Extracts barcodes and UMIs from read 1 and appends them to read 2 headers;
    its dedup function removes PCR duplicates (one read per UMI per genomic coordinate)
    after alignment
- name: scAPAtrap
  role: Downstream consumer of the uniquely mapped, deduplicated alignments for poly(A)
    site identification
- name: stAPAminer
  role: Toolkit that consumes the resulting poly(A) site matrix for spatial APA pattern
    mining
  repo: https://github.com/BMILAB/stAPAminer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:a9801499eea2c8e8f5dd484e91d4c5bf8927ff1416ed1f9169b99a861eef7493
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - STAR
  - scAPAtrap
  - stAPAminer
  - umi_tools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Spliced read alignment with uniquely-mapped filtering

## Summary

Aligns spatially barcoded ST sequencing reads (read 2) to the mouse GRCm38 genome with STAR and retains only uniquely mapped reads, producing the BAM input required by downstream poly(A) site identification with scAPAtrap in the stAPAminer MOB workflow.

## When to use

Use when raw paired-end spatially barcoded ST reads (e.g., MOB ST, SRA: SRR3382373) must be mapped to a reference genome before poly(A) site discovery, and downstream tools require unambiguous mapping.

## When NOT to use

- Input is already an aligned BAM or a gene/poly(A)-site count matrix — alignment and uniquely-mapped filtering are already done
- The analysis requires multi-mapper retention (e.g., repetitive or paralogous regions where multimapping evidence matters), since this pipeline discards non-unique alignments
- Data are image-based spatial assays (MERFISH/seqFISH) — the article notes APA sites can only be identified from spatially barcoded ST data with tools like scAPAtrap or Sierra

## Inputs

- Paired-end FASTQ from spatially barcoded ST data (read 1: barcode/UMI; read 2: transcript sequence), e.g., SRA: SRR3382373
- GRCm38 mouse reference genome assembly and annotation from Ensembl

## Outputs

- BAM/SAM of uniquely mapped read 2 alignments to GRCm38 with barcode and UMI in the read header
- Deduplicated alignment file (one read per UMI per genomic coordinate) ready for scAPAtrap poly(A) site identification

## How to apply

First extract spatial barcodes and UMIs from read 1 with umi_tools and append them to the read 2 sequence header, since downstream duplicate removal keys on barcode/UMI. Align the read 2 FASTQ to the GRCm38 mouse genome (downloaded from Ensembl) using STAR, a splice-aware aligner appropriate for polyadenylated transcript reads spanning exon junctions. Restrict the output to uniquely mapped reads only, discarding multi-mappers to ensure each read contributes to at most one genomic coordinate. This unambiguous alignment is what allows umi_tools dedup to keep one read per UMI per genomic coordinate and lets scAPAtrap call reliable poly(A) sites, ultimately yielding 47,886 sites from the ST-MOB replicate.

## Related tools

- **STAR** (Splice-aware aligner used to map read 2 to the GRCm38 reference genome, retaining only uniquely mapped reads)
- **umi_tools** (Extracts barcodes and UMIs from read 1 and appends them to read 2 headers; its dedup function removes PCR duplicates (one read per UMI per genomic coordinate) after alignment)
- **scAPAtrap** (Downstream consumer of the uniquely mapped, deduplicated alignments for poly(A) site identification)
- **stAPAminer** (Toolkit that consumes the resulting poly(A) site matrix for spatial APA pattern mining) — https://github.com/BMILAB/stAPAminer

## Examples

```
STAR --genomeDir GRCm38_star_index --readFilesIn read2.fq.gz --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate --outFilterMultimapNmax 1
```

## Evaluation signals

- Read 2 headers contain the barcode and UMI extracted from read 1 (umi_tools append step succeeded)
- Alignment file contains only uniquely mapped reads — no alignments with multiple genomic hits remain
- After umi_tools dedup, exactly one read per UMI per genomic coordinate is retained
- Downstream scAPAtrap on this BAM reproduces the reported 47,886 poly(A) sites (average 14,436 sites per spot) when filtering to spots with ≥10 read counts (260 spots)
- Site-level expression correlates with SC-MOB (Pearson ≈ 0.7) and neural bulk 3'-seq data (Pearson ≈ 0.69)

## Limitations

- No version numbers are reported for STAR or umi_tools in the article, so exact reproducibility requires tool-version testing
- Discarding multi-mapped reads may lose signal from repetitive genomic regions or paralogous genes
- The pipeline is only applicable to spatially barcoded ST data; existing tools cannot identify APA sites from image-based spatial technologies such as MERFISH or seqFISH

## Evidence

- [discussion] barcodes and UMIs were extracted from read 1, and umi_tools [63] was adopted to append the barcode and UMI information to the sequence header of read 2: "barcodes and UMIs were extracted from read 1, and umi_tools [63] was adopted to append the barcode and UMI information to the sequence header of read 2"
- [discussion] Reads from the read 2 file were aligned to the reference genome using STAR [64], and only uniquely mapped reads were retained: "Reads from the read 2 file were aligned to the reference genome using STAR [64], and only uniquely mapped reads were retained"
- [discussion] PCR duplicates were removed using the dedup function in umi_tools, and one read per UMI for each genomic coordinate remained: "PCR duplicates were removed using the dedup function in umi_tools, and one read per UMI for each genomic coordinate remained"
- [discussion] The genome assembly (version GRCm38) and latest genome annotation of mice were downloaded from Ensembl: "The genome assembly (version GRCm38) and latest genome annotation of mice were downloaded from Ensembl"
- [readme] APA sites can only be identified from the spatial barcoded ST data for now, using existing scRNA-seq tools like scAPAtrap or Sierra.: [evidence span withheld — anchor status: external_doc]
