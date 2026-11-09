---
name: in-silico-scrnaseq-mixture-assembly
description: Use when benchmarking single-cell genetic mixture deconvolution (e.g.
  de-goulash) against controlled imbalance levels, by pooling barcoded cells from
  two known scRNA-seq donors at defined ratios such as 1:10–1:99.
when_to_use_negative:
- When the goal is a real forensic mixture rather than a controlled benchmark — in-silico
  mixtures lack the live-cell processing artifacts of de-novo mixed blood samples
- When testing mixtures with more than two contributors or ratios beyond 1:99, which
  are outside the validated design space of this assembly scheme
- When input scRNA-seq data are not from a platform providing per-cell barcodes and
  a possorted genome BAM, since de-goulash genotyping is cell-barcode-driven
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: de-goulash
  role: Snakemake pipeline that genotypes each barcoded cell via freebayes, merges
    and clusters cells by genotype to separate the two synthetic contributors
  repo: https://github.com/genid/de-goulash.git
- name: freebayes
  role: Haplotype-based variant caller run in parallel over the regions file to call
    SNPs per cell from the mixture BAM
  repo: https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py
- name: Docker
  role: Container (geniderasmusmc/de-goulash:1) in which the Snakemake deconvolution
    step is executed with the working directory mounted at /single-cell
- name: Snakemake
  role: Workflow engine orchestrating the two threshold iterations, cell-merging python
    rule, and clustering Rscript
- name: 10X Genomics scRNA-seq platform
  role: Source of the single-individual scRNA-seq datasets (barcodes and possorted_genome_bam.bam)
    from which in-silico mixtures are assembled
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:9e513fa58f5e1b7d1c8b99883f032608c9ad4523d9423d542101c8fba91c9607
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genid/de-goulash.git
  tools:
  - 10X Genomics scRNA-seq platform
  - De-goulash
  - Docker
  - Python
  - R
  - Snakemake
  - de-goulash
  - freebayes
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# in-silico-scrnaseq-mixture-assembly

## Summary

Assembly of imbalanced in-silico two-person mixtures from single-individual 10X scRNA-seq datasets to benchmark the limits of de-goulash genetic deconvolution, testing mixture ratios from 1:10 up to 1:99 to find where minor contributors are no longer separable.

## When to use

Use when benchmarking single-cell genetic mixture deconvolution (e.g. de-goulash) against controlled imbalance levels, by pooling barcoded cells from two known scRNA-seq donors at defined ratios such as 1:10–1:99.

## When NOT to use

- When the goal is a real forensic mixture rather than a controlled benchmark — in-silico mixtures lack the live-cell processing artifacts of de-novo mixed blood samples
- When testing mixtures with more than two contributors or ratios beyond 1:99, which are outside the validated design space of this assembly scheme
- When input scRNA-seq data are not from a platform providing per-cell barcodes and a possorted genome BAM, since de-goulash genotyping is cell-barcode-driven

## Inputs

- Single-individual 10X scRNA-seq datasets (e.g. donors A2 and A4) with possorted_genome_bam.bam and barcodes.tsv per donor
- Human reference genome FASTA (hg19 or hg38) plus mitochondrial *MT.fasta of the same build
- regions.txt for parallel freebayes, generated with fasta_generate_regions.py from the genome .fai
- config.yaml specifying sample_bam, barcodes, reference, regions, and de-goulash threshold/filter parameters

## Outputs

- Per-ratio genotyped and clustered cell assignments in output/iter2/cells_merge_clusters.vcf
- Per-cluster VCF variant calls attributable to each synthetic contributor
- Deconvolution accuracy assessment per mixture ratio (clean separation vs. bleed-through vs. failure)

## How to apply

Assemble in-silico two-person mixtures by pooling single-individual scRNA-seq datasets (e.g. donors A2 and A4) at increasingly imbalanced ratios — 1:10, 1:20, 1:40, 1:60, 1:80, and 1:99 — retaining each mixture's possorted_genome_bam.bam and 10x barcodes.tsv so each synthetic mixture is processed exactly like an experimental one. Point config.yaml at each mixture's sample_bam, barcodes, reference genome.fasta, and the freebayes regions file generated with fasta_generate_regions.py, keeping default SNP filters (dp: 50, qual: 60). Run the de-goulash Snakemake deconvolution step in the geniderasmusmc/de-goulash:1 docker container with two threshold iterations (thr_cell_1: 10, thr_cell_2: 20 minimal SNPs per cell), then apply the cell-merging rule (threshold_coverage: 10, threshold_coverage_pos: 5, threshold_base_calling: 90) and the UMAP clustering Rscript (n_neighbors: 5, n_components: 300, clusters: 0). Judge performance by whether cells cluster by true donor: clean separation at 1:10–1:40, minor bleed-through at 1:60 (about five minor-component cells misassigned), degraded assignment at 1:80, and total failure at 1:99.

## Related tools

- **de-goulash** (Snakemake pipeline that genotypes each barcoded cell via freebayes, merges and clusters cells by genotype to separate the two synthetic contributors) — https://github.com/genid/de-goulash.git
- **freebayes** (Haplotype-based variant caller run in parallel over the regions file to call SNPs per cell from the mixture BAM) — https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py
- **Docker** (Container (geniderasmusmc/de-goulash:1) in which the Snakemake deconvolution step is executed with the working directory mounted at /single-cell)
- **Snakemake** (Workflow engine orchestrating the two threshold iterations, cell-merging python rule, and clustering Rscript)
- **10X Genomics scRNA-seq platform** (Source of the single-individual scRNA-seq datasets (barcodes and possorted_genome_bam.bam) from which in-silico mixtures are assembled)

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml --cores 1
```

## Evaluation signals

- At ratios 1:10–1:40 the UMAP clustering (n_neighbors: 5, n_components: 300) must yield two genotype clusters matching the two known donors with no misassigned cells
- At 1:60 only minor bleed-through is acceptable — about five minor-component cells misassigned to the major cluster
- At 1:80 assignment is degraded and at 1:99 separation should fail entirely; results outside this pattern indicate a pipeline or configuration error
- Each merged cell must pass the merging thresholds: ≥10 total coverage over selected SNPs, ≥5 coverage per selected SNP, ≥90% base-calling agreement
- Cells retained per iteration must meet the minimal-SNP thresholds (thr_cell_1: 10 and thr_cell_2: 20 SNPs per cell)

## Limitations

- Correct deconvolution is possible only up to about 1:60; at 1:80 assignment degrades and at 1:99 the minor contributor cannot be separated at all
- The 10X platform requires live cells, and in-silico assembly bypasses real mixture creation, so results may not transfer directly to de-novo mixed samples
- The 10X scRNA-seq platform requires live cells for successful genetic separation, which consequently limits the application to biological mixtures containing living cells
- Transferability to biological mixtures other than blood is not yet demonstrated

## Evidence

- [abstract] Clean separation up to 1:60, failure at 1:99: "For all 2 up to 9-person balanced and imbalanced blood mixtures with ratios up to 1:60, we achieved a clear single-cell separation according to the contributing individuals."
- [abstract] In-silico mixture assembly from public single-individual datasets: "tested it with scRNA-seq data that we de-novo generated from multi-person blood mixtures, and also in-silico mixtures we assembled from public single individual scRNA-seq datasets"
- [readme] Docker execution of the deconvolution step: "docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml --cores 1"
- [readme] SNP filtering and per-cell thresholds in config.yaml: "* dp: 50 # SNP filtering depth
* qual: 60 # SNP filtering quality"
- [methods] Cell-merging coverage and base-calling thresholds: "* threshold_coverage: 10 #treshhold total coverage of selected SNPs per cell
* threshold_coverage_pos: 5 #treshold coverage per selected SNP per cell"
- [discussion] Live-cell requirement of the 10X platform: "requires live cells for successful genetic separation, which consequently limits the application to biological mixtures containing living cells"
