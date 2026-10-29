---
name: empirical-vaf-parameter-estimation
description: 'Use when parameterizing simulated single-cell mtDNA data: you need empirically
  grounded background SNPs, depths, and heteroplasmy distributions sampled from real
  cells rather than arbitrary values.'
when_to_use_negative:
- You already have empirical VAF distributions or a validated simulation parameter
  file for the assay — re-estimating from scratch is unnecessary
- Your data is bulk or scATAC-seq with different depth/error characteristics — the
  depth >50 and VAF >1% in >5% thresholds were tuned for Smart-seq2-like deep scRNA-seq
  mtDNA pileups
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: cellsnp-lite
  role: Piles up raw reads from BAM files to generate the SNP-by-cell AD/DP matrix
    from which empirical VAFs and depths are computed
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: SomatoSim
  role: Consumes the sampled empirical VAFs/background SNPs to add background noise
    and clonal mutations in simulated mtDNA reads
- name: NEAT-genReads
  role: Synthesizes paired-end mtDNA reads (152 bp, hg19) at the coverage levels informed
    by empirical depth estimates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:e2f3e50f426793c600025b5fa99b3e2ca1166f16d2582b06128dbdb1ae946932
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - MQuad
  - Monovar
  - NEAT-genReads
  - SomatoSim
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# empirical-vaf-parameter-estimation

## Summary

Estimates empirical variant allele frequency (VAF/heteroplasmy) and depth parameters from a real single-cell dataset (hematopoietic colony scRNA-seq, 536 cells, via cellsnp-lite) so that simulated mtDNA sequencing benchmarks reflect realistic background noise and heteroplasmy distributions.

## When to use

Use when parameterizing simulated single-cell mtDNA data: you need empirically grounded background SNPs, depths, and heteroplasmy distributions sampled from real cells rather than arbitrary values.

## When NOT to use

- You already have empirical VAF distributions or a validated simulation parameter file for the assay — re-estimating from scratch is unnecessary
- Your data is bulk or scATAC-seq with different depth/error characteristics — the depth >50 and VAF >1% in >5% thresholds were tuned for Smart-seq2-like deep scRNA-seq mtDNA pileups

## Inputs

- Aligned single-cell BAM files (e.g., Smart-seq2 hematopoietic colony scRNA-seq, 536 cells, STAR-aligned to hg19)
- Mitochondrial reference genome (hg19 rCRS region)
- cellsnp-lite pileup parameters (cell barcodes, --chrom chrM/chrMT)

## Outputs

- Cell-by-SNP VAF/AD/DP matrix from cellsnp-lite (SNP-by-cell counts)
- Filtered list of ~3500 background SNPs (avg depth >50, heteroplasmy VAF >1% in >5% of cells)
- Sampled empirical VAF values for simulated cells (e.g., 90 cells) used in SomatoSim simulation setup

## How to apply

Run cellsnp-lite on the BAM files of the scRNA-seq dataset (e.g., the hematopoietic colony dataset of 536 cells from Lareau et al.) to build a cell-by-SNP VAF matrix over mitochondrial positions. Filter candidate background SNPs to those with average depth >50 and heteroplasmy (VAF >1%) present in more than 5% of the total cells, yielding ~3500 SNPs. From this filtered set, randomly sample VAFs and assign them to simulated cells (e.g., 90 simulated cells) so background noise in the synthetic reads matches real Smart-seq2-like distributions. These empirical VAFs and depths then feed the read synthesizer (NEAT-genReads) and mutation injector (SomatoSim), which add clonal variants at log-normal allele frequencies Lognormal(mu, 0.005). The rationale is that benchmark results (AUPRC/AUROC for MQuad vs mgatk vs Monovar) are only trustworthy if the simulated sequencing-error background matches empirical mtDNA noise.

## Related tools

- **cellsnp-lite** (Piles up raw reads from BAM files to generate the SNP-by-cell AD/DP matrix from which empirical VAFs and depths are computed) — https://github.com/single-cell-genetics/cellsnp-lite
- **SomatoSim** (Consumes the sampled empirical VAFs/background SNPs to add background noise and clonal mutations in simulated mtDNA reads)
- **NEAT-genReads** (Synthesizes paired-end mtDNA reads (152 bp, hg19) at the coverage levels informed by empirical depth estimates)

## Examples

```
cellsnp-lite -s sample.bam -b barcodes.tsv -O vaf_out/ -R mt_positions.vcf --chrom chrM -p 20
```

## Evaluation signals

- The filtered SNP set contains only loci with average depth >50 and heteroplasmy (VAF >1%) in >5% of total cells (~3500 SNPs from 536 cells)
- Simulated background VAF distribution matches the empirical hematopoietic-colony pileup distribution (sampled, not parametric)
- Resulting simulated BAMs reach ~1000x coverage in high-coverage BED regions with ~2% off-target coverage, mirroring real Smart-seq2 mtDNA depth
- Downstream benchmark at default settings (M=10 variants/clone) reproduces expected relative ordering: MQuad AUPRC ~0.976 > mgatk 0.800 > Monovar 0.147

## Limitations

- Thresholds (avg depth >50, VAF >1% in >5% cells) are empirical choices tied to the hematopoietic colony Smart-seq2 dataset; other assays may need recalibration
- Error-prone rCRS regions 302-315, 513-525, 3105-3109 harbor frequent false positives and must be excluded (per Xu et al.)
- Simulations use hg19; comparisons with data aligned to hg38 require care since reference differs

## Evidence

- [discussion] Simulation SNP selection criteria: average depth >50 and heteroplasmy (VAF >1%) in >5% of cells: "we selected variants that are 1) deeply covered with average depth >50 and 2) heteroplasmic (VAF > 1%) in more than 5% of the total cells"
- [discussion] cellsnp-lite pileup produces SNP-by-cell matrix: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [discussion] Simulated reads with NEAT-genReads and SomatoSim: "Mitochondria reads were synthesized in silico with NEAT-genReads v3.038 and SomatoSim (v1.0.0)39"
- [readme] cellsnp-lite output directly usable for MQuad clonal substructure discovery: [evidence span withheld — anchor status: external_doc]
