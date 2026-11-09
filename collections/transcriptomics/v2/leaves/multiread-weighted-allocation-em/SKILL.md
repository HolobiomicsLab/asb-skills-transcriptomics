---
name: multiread-weighted-allocation-em
description: Use when allele-specific count matrices must be generated from diploid-transcriptome
  alignments containing genomic or allelic multi-mapping reads, and discarding multi-reads
  would lose the majority of available reads.
when_to_use_negative:
- When the alignment set contains no multi-mapping reads — unique-reads counting is
  then equivalent and simpler
- When reads must be counted for a haplotype-unaware analysis (e.g. aggregate expression
  quantification), where genomic multi-read handling via EMASE allele-weighting is
  unnecessary
- When working with UMI-based counts where the expression-threshold convention differs
  (e.g. Larsson et al. UMI data, where expression is judged against one UMI)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EMASE
  role: Runs the EM algorithm that resolves each multi-read with weights based on
    the estimated probability of each alignment
- name: g2gtools
  role: Reconstructs the CAST diploid reference by incorporating known SNPs and short
    indels (Sanger REL-1505) into the mouse genome
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Aligns reads to the diploid transcriptome index with --all --best --strata
    and -v 3 to enumerate all multi-read alignments
- name: scBASE
  role: Downstream beta-binomial mixture classification (P/B/M) and partial pooling
    applied to the weighted-allocation counts
  repo: github.com/churchill-lab/scBASE
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:867dac42ce3c80136cae2517a001858de646db0501568a4ca43e827f72ee2387
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - Stan
  - bowtie
  - bowtie v1.0.0
  - g2gtools
  - scBASE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# multiread-weighted-allocation-em

## Summary

Resolve genomic and allelic multi-mapping reads from allele-specific scRNA-Seq into gene- and allele-specific counts via the EM-based weighted allocation implemented in EMASE, instead of discarding multi-reads (unique-reads counting). Weighted allocation retains reads that unique-reads counting throws away, reducing spurious monoallelic and dynamic expression calls.

## When to use

Use when allele-specific count matrices must be generated from diploid-transcriptome alignments containing genomic or allelic multi-mapping reads, and discarding multi-reads would lose the majority of available reads.

## When NOT to use

- When the alignment set contains no multi-mapping reads — unique-reads counting is then equivalent and simpler
- When reads must be counted for a haplotype-unaware analysis (e.g. aggregate expression quantification), where genomic multi-read handling via EMASE allele-weighting is unnecessary
- When working with UMI-based counts where the expression-threshold convention differs (e.g. Larsson et al. UMI data, where expression is judged against one UMI)

## Inputs

- Single-cell RNA-Seq reads (e.g. Deng et al. 286 CAST×B6 F1 pre-implantation embryo cells)
- CAST genome reconstruction inputs: reference mouse genome plus known SNPs and short indels (Sanger REL-1505)
- Diploid transcriptome bowtie index built with g2gtools
- All best-scoring alignments per read (--all --best --strata, up to 3 mismatches), including multi-reads

## Outputs

- Weighted-allocation allele-specific count matrix (gene × cell × allele) with multi-reads fractionally assigned by EMASE
- Unique-reads count matrix (for contrast), obtained by discarding all genomic and allelic multi-reads
- Summary statistics: fraction of reads retained, monoallelic genes per cell, missed bi-allelic genes, per-gene monoallelic-cell tables (e.g. allelic_counts_comparison.csv)

## How to apply

Reconstruct the phase-known diploid reference first: incorporate known CAST SNPs and short indels (Sanger REL-1505) into the reference mouse genome with g2gtools, then build a bowtie (v1.0.0) index of the diploid transcriptome. Align reads with bowtie using --all, --best, and --strata and -v 3 so that all best-scoring alignments, including multi-reads, are enumerated. For each multi-read, run the EM algorithm in EMASE, which assigns weights to each candidate alignment based on its estimated probability, and allocate the read fractionally across those alignments to produce allele-specific counts. Contrast this with the unique-reads method, which discards all genomic and allelic multi-reads and retains only unambiguous reads (only 14.9% of available reads in the Deng et al. 286-cell data). Feed the resulting weighted-allocation counts into scBASE (Stan-backed beta-binomial mixture) for P/B/M classification and partial pooling, and expect fewer spurious monoallelic calls than with unique-reads counts (e.g. Mtdh: 88 monoallelic cells with unique-reads vs 7 with weighted allocation).

## Related tools

- **EMASE** (Runs the EM algorithm that resolves each multi-read with weights based on the estimated probability of each alignment)
- **g2gtools** (Reconstructs the CAST diploid reference by incorporating known SNPs and short indels (Sanger REL-1505) into the mouse genome) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Aligns reads to the diploid transcriptome index with --all --best --strata and -v 3 to enumerate all multi-read alignments)
- **scBASE** (Downstream beta-binomial mixture classification (P/B/M) and partial pooling applied to the weighted-allocation counts) — github.com/churchill-lab/scBASE

## Evaluation signals

- Unique-reads counting retains only ~14.9% of available reads; weighted allocation should retain substantially more
- Monoallelic calls per cell are ~66 higher with unique-reads than with weighted allocation (Fig 2a); the weighted-allocation matrix should show fewer monoallelic calls
- Per-gene spot checks match reported contrasts, e.g. Mtdh called monoallelic in 88 unique-reads cells vs 7 weighted-allocation cells (Table 1)
- Roughly ~1,908 bi-allelic genes missed by unique-reads should be recovered under weighted allocation
- Weighted-allocation counts feed scBASE partial pooling with lower MSE than unique-reads (avg MSE diff ~0.016 on 6,751 genes in the 1% down-sampled 122 blastocyst-cell test)

## Limitations

- Unique-reads counting, if used instead, leads to higher variability in allelic proportion estimates, more sampling zeros, and spurious findings of dynamic and monoallelic expression
- No software version is stated for EMASE, limiting exact reproducibility
- Discarding multi-reads is only defensible when multi-reads are rare; in these F1 hybrid datasets the loss (85.1% of reads) is too large to ignore

## Evidence

- [discussion] EMASE EM weighted allocation: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
- [discussion] EMASE implements the EM step: [evidence span withheld — anchor status: not_found]
- [discussion] unique-reads discards multi-reads: "Unique-read counts are obtained directly from counting reads after discarding all genomic and allelic multi-reads"
- [discussion] bowtie alignment parameters: [evidence span withheld — anchor status: not_found]
- [results] unique-reads retains few reads and inflates monoallelic calls: [evidence span withheld — anchor status: not_found]
