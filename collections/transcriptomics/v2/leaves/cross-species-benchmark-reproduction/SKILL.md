---
name: cross-species-benchmark-reproduction
description: Use when scRNA-seq FASTQs from multiple organisms must be assessed for
  how much transcriptional signal falls outside current genome annotations, or when
  reproducing reported uTAR read fractions (e.g., hg38 2%, lemur 7.1%, mole rat 4.2%,
  sea urchin 10.7%).
when_to_use_negative:
- Reads are already aligned with cellranger count against a fixed annotation — use
  the from_cellranger workflow variant instead of from_fastqs, since the benchmark
  requires annotation-independent alignment.
- The goal is differential-expression or cell-clustering analysis on an existing expression
  matrix rather than quantifying reads outside gene annotations.
- Only a single organism is of interest and no cross-species comparison of annotation
  completeness is intended; the per-organism fraction aggregation and tolerance-band
  validation would be unnecessary overhead.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Snakemake
  role: 'Orchestrates the from_fastqs workflow: trimming, barcode tagging, STAR alignment,
    and matrix generation across all six datasets'
  repo: github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Hidden Markov model that divides each genome into transcribed/untranscribed
    states to call strand-specific TARs from uniquely aligned reads
  repo: github.com/fw262/TAR-scRNA-seq
- name: rtracklayer
  role: R package used alongside groHMM to load and overlap refFlat annotations when
    classifying TARs as aTARs or uTARs
  repo: github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:2cf3f606c75813d8e940c0b9a144338994f5e2897a8e0a1d96133fb13df3f624
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Snakemake
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-species-benchmark-reproduction

## Summary

Reproduces the TAR-scRNA-seq annotation-gap benchmark: quantifying the fraction of unique scRNA-seq reads that align outside gene annotations (assigned to uTARs) per organism, from raw FASTQs through groHMM TAR calling to a per-dataset fraction summary table.

## When to use

Use when scRNA-seq FASTQs from multiple organisms must be assessed for how much transcriptional signal falls outside current genome annotations, or when reproducing reported uTAR read fractions (e.g., hg38 2%, lemur 7.1%, mole rat 4.2%, sea urchin 10.7%).

## When NOT to use

- Reads are already aligned with cellranger count against a fixed annotation — use the from_cellranger workflow variant instead of from_fastqs, since the benchmark requires annotation-independent alignment.
- The goal is differential-expression or cell-clustering analysis on an existing expression matrix rather than quantifying reads outside gene annotations.
- Only a single organism is of interest and no cross-species comparison of annotation completeness is intended; the per-organism fraction aggregation and tolerance-band validation would be unnecessary overhead.

## Inputs

- Public scRNA-seq FASTQ files (human PBMC, mouse spleen/kidney, gray mouse lemur lung, naked mole rat spleen, sea urchin embryo, chicken heart)
- Genome assemblies (FASTA) for each organism
- refFlat-format gene annotations per organism (e.g., GENCODE v30 hg38)
- groHMM software parameters (50 bp window, gamma emissions, bedtools merge -s -d 500, 1/10,000,000 unique-read TAR cutoff, 5% downsampling for Tabula Muris)

## Outputs

- Gene and TAR single-cell expression matrices (Drop-seq tools format)
- groHMM bed files of strand-specific TARs per chromosome
- aTAR/uTAR classification of TARs by refFlat overlap
- CSV summary table with the uTAR read fraction for each of the six datasets

## How to apply

Download public scRNA-seq FASTQs (Tabula Muris mouse spleen/kidney, 10X human PBMC, gray mouse lemur lung, naked mole rat spleen, sea urchin embryo, chicken heart) plus each genome assembly and its refFlat-format gene annotation. Run the TAR-scRNA-seq Snakemake workflow in from_fastqs mode to trim adapters, tag barcodes, and align with STAR without gene-annotation indexing, producing gene and TAR expression matrices via Drop-seq tools. Call transcriptionally active regions strand-specifically with groHMM using a 50 bp non-overlapping window, Baum-Welch EM-learned emission probabilities, merging predicted regions within 500 bp via bedtools merge (-s -d 500), retaining TARs with at least 1/10,000,000 of uniquely mapped reads; for Tabula Muris, downsample the combined alignment to 5% of uniquely aligned reads to fit groHMM memory constraints. Classify HMM-derived TARs as aTARs (any refFlat transcript overlap, ignoring strandedness) or uTARs (no overlap), then compute the relative number of unique reads assigned to uTARs per cell per dataset and aggregate into a CSV summary table. Rationale: annotation-independent STAR alignment plus HMM-based TAR discovery captures signal that a fixed annotation would discard, and comparing uTAR fractions across species exposes annotation incompleteness in less-studied organisms.

## Related tools

- **Snakemake** (Orchestrates the from_fastqs workflow: trimming, barcode tagging, STAR alignment, and matrix generation across all six datasets) — github.com/fw262/TAR-scRNA-seq
- **groHMM** (Hidden Markov model that divides each genome into transcribed/untranscribed states to call strand-specific TARs from uniquely aligned reads) — github.com/fw262/TAR-scRNA-seq
- **rtracklayer** (R package used alongside groHMM to load and overlap refFlat annotations when classifying TARs as aTARs or uTARs) — github.com/fw262/TAR-scRNA-seq

## Examples

```
Rscript -e 'library(groHMM); library(rtracklayer)'  # verify groHMM/rtracklayer load before running the Snakemake from_fastqs workflow
```

## Evaluation signals

- Summary table exists in CSV format and contains a fraction value for each of the six datasets.
- Computed uTAR fractions fall within a tolerance band around the reported reference means: hg38 2%, gray mouse lemur 7.1%, naked mole rat 4.2%, purple sea urchin 10.7%.
- groHMM bed files are produced for every chromosome and the TAR retention cutoff (>= 1/10,000,000 of uniquely mapped reads) was applied.
- STAR alignment was run without gene-annotation indexing, so fractions are not biased by annotation-guided mapping.
- aTAR/uTAR classification is consistent with the overlap rule: any refFlat transcript overlap (ignoring strandedness) = aTAR; everything else = uTAR.

## Limitations

- groHMM imposes memory constraints: the Tabula Muris combined alignment had to be downsampled to 5% of uniquely aligned reads.
- A groHMM failure can silently break downstream steps: rule calcHMMrefFlat may error with 'no lines available in input' because chr*_HMM.bed files were never generated.
- groHMM and rtracklayer must be manually installed from Bioconductor; the workflow will fail if they cannot be loaded in R.
- Reported fractions depend on the annotation version used (e.g., GENCODE v30 hg38) and are strongly dependent on organism, tissue, cell type, and stage of development.

## Evidence

- [abstract] Genome annotations are often incomplete across species: "we show here with scRNA-seq experiments and analyses spanning human, mouse, chicken, mole rat, lemur and sea urchin, genome annotations are often incomplete"
- [intro] uTAR relevance depends on organism/tissue/cell type: "we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development"
- [intro] groHMM hidden Markov model TAR calling: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Workflow entry points: "Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*)."
- [readme] Manual Bioconductor install requirement: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
