---
name: counting-method-contrast-validation
description: Use when scRNA-Seq allele-specific expression counts could be produced
  either by keeping only uniquely mapping reads or by resolving multi-reads, and you
  need to quantify the bias introduced by discarding multi-reads.
when_to_use_negative:
- Data are UMI-based with unique molecular identifiers already resolving multi-reads
  (e.g. the Larsson et al. 224-cell dataset), where read-level multi-mapping handling
  differs
- Goal is burst-kinetics inference rather than counting-method comparison (use txburst
  on allelic UMI counts instead)
- Reads were aligned restricting to a single best alignment, so multi-read alignments
  were never enumerated and weighted allocation is impossible
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: g2gtools
  role: Incorporates known SNPs and short indels (Sanger REL-1505) into the reference
    mouse genome to reconstruct the CAST genome for the diploid transcriptome
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Aligns single-cell reads to the diploid transcriptome index with --all --best
    --strata and -v 3, enumerating all best-scoring alignments including multi-reads
- name: EMASE
  role: Runs the EM algorithm that resolves multi-reads via weighted allocation based
    on estimated alignment probability, producing the second count matrix
- name: scBASE
  role: Beta-binomial mixture model (via Stan) classifying each gene in each cell
    into paternal monoallelic, bi-allelic, or maternal monoallelic states on both
    count matrices
  repo: https://github.com/kbchoi-jax/scbase
- name: Stan
  role: Probabilistic programming backend for the scBASE mixture-model inference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:d0db1f4b9e0ce0c9769ed3ce2da06f32107bfac3b8f858fb3b800c32320a72ff
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# counting-method-contrast-validation

## Summary

Contrasts unique-reads allele-specific counting (discarding multi-reads) with EM-based weighted allocation of multi-reads (EMASE) to quantify how discarding multi-mapping reads inflates monoallelic expression calls in single-cell RNA-Seq.

## When to use

Use when scRNA-Seq allele-specific expression counts could be produced either by keeping only uniquely mapping reads or by resolving multi-reads, and you need to quantify the bias introduced by discarding multi-reads.

## When NOT to use

- Data are UMI-based with unique molecular identifiers already resolving multi-reads (e.g. the Larsson et al. 224-cell dataset), where read-level multi-mapping handling differs
- Goal is burst-kinetics inference rather than counting-method comparison (use txburst on allelic UMI counts instead)
- Reads were aligned restricting to a single best alignment, so multi-read alignments were never enumerated and weighted allocation is impossible

## Inputs

- FASTQ single-cell reads from 286 Deng et al. pre-implantation embryo cells (CAST×B6 F1 hybrid)
- Reference mouse genome plus CAST SNP/indel calls (Sanger REL-1505) for diploid transcriptome reconstruction
- Allele-specific read alignments with all best-scoring mappings enumerated (bowtie --all --best --strata -v 3)

## Outputs

- Unique-reads allele-specific count matrix (multi-reads discarded) for 286 cells
- Weighted-allocation allele-specific count matrix (EMASE EM algorithm) for 286 cells
- Per-cell and per-gene contrasts of P/B/M classifications, e.g. monoallelic genes per cell and Mtdh 2x2 table
- Summary written to allelic_counts_comparison.csv

## How to apply

Reconstruct the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) with g2gtools, build a bowtie (v1.0.0) index of the diploid transcriptome, and align the 286 Deng et al. CAST×B6 embryo scRNA-Seq reads with bowtie using '--all --best --strata -v 3' so all best-scoring alignments (including multi-reads) are enumerated. From these alignments, build two count matrices for the same cells: (1) unique-reads counts that discard all genomic and allelic multi-reads, counting only reads mapping unambiguously to one allele of one gene, and (2) weighted-allocation counts from the EM algorithm in EMASE that resolves each multi-read by estimated alignment probability. Classify each gene in each cell as paternal monoallelic, bi-allelic, or maternal monoallelic with the scBASE beta-binomial mixture model (via Stan), treating an allele as expressed only if its estimated abundance exceeds one read. The rationale is that multi-reads carry real allelic information, so their loss inflates sampling zeros and variability in allelic proportions; the contrast quantifies the resulting classification bias. Judge correctness by reproducing the article's reported contrasts: unique-reads retains only 14.9% of available reads, calls ~66 more monoallelic genes per cell, misses ~1,908 bi-allelic genes, and yields 88 vs 7 monoallelic cells for Mtdh (Table 1).

## Related tools

- **g2gtools** (Incorporates known SNPs and short indels (Sanger REL-1505) into the reference mouse genome to reconstruct the CAST genome for the diploid transcriptome) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Aligns single-cell reads to the diploid transcriptome index with --all --best --strata and -v 3, enumerating all best-scoring alignments including multi-reads)
- **EMASE** (Runs the EM algorithm that resolves multi-reads via weighted allocation based on estimated alignment probability, producing the second count matrix)
- **scBASE** (Beta-binomial mixture model (via Stan) classifying each gene in each cell into paternal monoallelic, bi-allelic, or maternal monoallelic states on both count matrices) — https://github.com/kbchoi-jax/scbase
- **Stan** (Probabilistic programming backend for the scBASE mixture-model inference)

## Examples

```
bowtie -v 3 --all --best --strata diploid_transcriptome reads.fq > alns.bowtie  # then: unique-read counts vs EMASE weighted allocation, classify with scBASE/Stan
```

## Evaluation signals

- Unique-reads counting retains 14.9% of available reads; a substantially higher retention indicates the multi-read filter was misapplied
- Unique-reads method calls ~66 more monoallelic genes per cell (Fig. 2a) and misses ~1,908 bi-allelic genes relative to weighted allocation
- Mtdh 2x2 check: 88 cells called monoallelic under unique-reads vs 7 under weighted allocation (Table 1)
- Both pipelines must share identical alignment input (same bowtie --all --best --strata -v 3 alignments); differences should arise only from counting, not alignment
- Contrast metrics (reads retained, monoallelic genes per cell, bi-allelic genes missed, per-gene monoallelic-cell counts) match values in allelic_counts_comparison.csv

## Limitations

- Unique-reads counting leads to higher variability in allelic proportion estimates, increased sampling zeros, and spurious findings of dynamic and monoallelic expression
- The gene-level ASE class thresholds (majority of cells with πg⋅s>0.7) are acknowledged as arbitrary, limiting reproducibility of gene classification
- No software version stated for EMASE or g2gtools; MCMC settings deferred to Supplementary Methods

## Evidence

- [intro] Unique-reads method excludes multi-mapping reads and counts only reads mapping unambiguously to one allele of one gene: "we exclude multi-mapping reads (multi-reads) and count only reads that map unambiguously to one allele of one gene"
- [discussion] Alignment parameters for enumerating all best alignments: [evidence span withheld — anchor status: not_found]
- [discussion] Weighted allocation via EMASE resolves multi-reads: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
- [discussion] Unique-read counts obtained by discarding all genomic and allelic multi-reads: "Unique-read counts are obtained directly from counting reads after discarding all genomic and allelic multi-reads"
- [discussion] Allele expressed threshold: "we regarded it as expressed if its estimated abundance is greater than one reads (or one UMI as in Larsson et al.22)"
