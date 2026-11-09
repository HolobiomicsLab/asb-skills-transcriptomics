---
name: single-cell-barcode-downsampling
description: Use when you need to determine the minimum number of cells from a deconvoluted
  single-cell cluster that still supports reliable genetic identification, e.g. testing
  identification sensitivity on 10X barcodes_reduced.txt files.
when_to_use_negative:
- The mixture sample has fewer cells per cluster than the lowest tested point (article
  notes deconvolution from fewer than 150 cells with deeper sequencing was not empirically
  confirmed).
- Input is bulk DNA/VCF data rather than 10X single-cell barcodes/BAM — the downsampling
  operates on cell-level barcode files.
- Cells are fixed or non-living — the 10X Genomics scRNA-seq platform used here requires
  live cells for successful genetic separation.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0199
tools:
- name: De-goulash
  role: Snakemake pipeline whose step 1 (deconvolution/genotyping/clustering) and
    step 2 (statistical analysis, LR computation) are executed on each barcode subset
  repo: https://github.com/genid/de-goulash.git
- name: Snakemake
  role: Orchestrates deconvolution (Snakefile) and statistical analysis (Snakefile_analysis)
    per subset
- name: Docker
  role: Runs the de-goulash pipeline via the geniderasmusmc/de-goulash:1 image with
    the input directory mounted at /single-cell
- name: freebayes
  role: Haplotype-based variant caller invoked in parallel regions during de-goulash
    step 1 on each subsampled cell set
  repo: https://github.com/nh13/freebayes
- name: Yleaf
  role: 'Used in step 2 with read_depth: 1, quality: 20, base_calling: 90 and WGS_hg38
    position file for paternal haplogroup determination on subsampled clusters'
- name: Python
  role: Subsampling barcodes from barcodes_reduced.txt and generating reference region
    files; step-2 analysis
- name: R
  role: 'UMAP clustering Rscript settings (n_neighbors: 5, n_components: 300) and
    visualisation of sensitivity trends'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:ad99f790d0c5aad6e3d1e8b320a2c6c67bb75c45f5c64677ccc3ee14332bfc9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genid/de-goulash.git
  tools:
  - De-goulash
  - Docker
  - Python
  - R
  - Snakemake
  - Yleaf
  - freebayes
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-barcode-downsampling

## Summary

Subsamples 10X scRNA-seq barcodes (10–500 cells per cluster) to assess how the number of cells sampled from a deconvoluted mixture cluster affects downstream SNP matching, identification stability, and likelihood-ratio thresholds in the de-goulash pipeline.

## When to use

Use when you need to determine the minimum number of cells from a deconvoluted single-cell cluster that still supports reliable genetic identification, e.g. testing identification sensitivity on 10X barcodes_reduced.txt files.

## When NOT to use

- The mixture sample has fewer cells per cluster than the lowest tested point (article notes deconvolution from fewer than 150 cells with deeper sequencing was not empirically confirmed).
- Input is bulk DNA/VCF data rather than 10X single-cell barcodes/BAM — the downsampling operates on cell-level barcode files.
- Cells are fixed or non-living — the 10X Genomics scRNA-seq platform used here requires live cells for successful genetic separation.

## Inputs

- barcodes_reduced.txt — barcodes file from the deconvoluted M4 cluster (10X output)
- possorted_genome_bam.bam — 10X possorted genome BAM for the mixture sample
- genome.fasta (hg38) plus MT.fasta and generated regions.txt / MT_regions.txt (via python process_reference.py)
- config.yaml with de-goulash step-1 and step-2 parameters
- exome_96_remmapedto38.vcf.gz WES reference database
- 1000G release 20130502 variant data and 1000G_populations.txt

## Outputs

- Per-subset merged-cluster VCF (e.g. output/iter2/cells_merge_clusters.vcf) from de-goulash step 1
- Per-subset likelihood ratios, matching-SNP percentages, and ancestry/sex calls from step 2
- Aggregated sensitivity curves: proportion of matching SNPs, identification stability, and LR vs. number of cells (10–500, 10 replicates per point)

## How to apply

Read the config.yaml parameterisation of the de-goulash pipeline (dp: 50, qual: 60, thr_cell_1: 10, thr_cell_2: 20, threshold_coverage: 10, threshold_coverage_pos: 5, threshold_base_calling: 90). From the barcodes_reduced.txt file of the deconvoluted M4 cluster, randomly subsample 10–500 barcodes per cell-count point, generating 10 replicate subsets per point to capture sampling variability. For each subset, run de-goulash Snakemake step 1 (deconvolution with freebayes variant calling and cell merging) and step 2 (Snakefile_analysis with dp_2: 50, qual_2: 60, Yleaf parameters read_depth: 1, quality: 20, base_calling: 90, WGS_hg38 positions, and 1000G population data) inside the geniderasmusmc/de-goulash:1 Docker container. Aggregate across replicates and cell counts the proportion of matching SNPs against the WES reference (exome_96_remmapedto38.vcf.gz), identification stability, and likelihood ratio, and visualise trends. Validate against the reported criteria: >90% matching SNPs for subsets with more than 20 cells, identification stabilising above 50 cells, and LR exceeding the 10E+9 threshold at 100–200 cells.

## Related tools

- **De-goulash** (Snakemake pipeline whose step 1 (deconvolution/genotyping/clustering) and step 2 (statistical analysis, LR computation) are executed on each barcode subset) — https://github.com/genid/de-goulash.git
- **Snakemake** (Orchestrates deconvolution (Snakefile) and statistical analysis (Snakefile_analysis) per subset)
- **Docker** (Runs the de-goulash pipeline via the geniderasmusmc/de-goulash:1 image with the input directory mounted at /single-cell)
- **freebayes** (Haplotype-based variant caller invoked in parallel regions during de-goulash step 1 on each subsampled cell set) — https://github.com/nh13/freebayes
- **Yleaf** (Used in step 2 with read_depth: 1, quality: 20, base_calling: 90 and WGS_hg38 position file for paternal haplogroup determination on subsampled clusters)
- **Python** (Subsampling barcodes from barcodes_reduced.txt and generating reference region files; step-2 analysis)
- **R** (UMAP clustering Rscript settings (n_neighbors: 5, n_components: 300) and visualisation of sensitivity trends)

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml --cores 1
```

## Evaluation signals

- Percentage of matching SNPs vs. the WES reference stays above 90% for all subsampled data points consisting of more than 20 cells.
- All M4 cluster subsets exceed the conservative 10E+9 likelihood-ratio threshold when 100–200 cells are sampled.
- Identification (sex, bio-geographic ancestry) remains stable across the 10 replicate barcode subsets at cell counts above 50.
- Subsets below thr_cell thresholds (10 and 20 minimal SNPs per cell in iterations 1 and 2) should drop out, confirming filtering behaviour of config parameters.
- Reproduced trends match the article's reported sensitivity profile across the 10–500 cell range.

## Limitations

- Success depends on the number of SNPs obtained per individual cell cluster; very low cell counts may not pass the per-cell SNP and coverage thresholds.
- The 10X Genomics scRNA-seq platform requires live cells, limiting application to mixtures containing living cells.
- Reference datasets must contain as many SNPs as possible (WES was used here); no universal identity SNPs are used.
- Deeper sequencing/coverage for mixtures with very low cell numbers has not been empirically tested; results from <150 cells need future empirical confirmation.

## Evidence

- [results] The percentage of matching SNPs stayed above 90% for all sampled data points consisting of more than 20 cells: [evidence span withheld — anchor status: not_found]
- [readme] Subsample 10–500 barcodes: "* barcodes: /single-cell/input/1/barcodes_reduced.txt"
- [readme] Docker invocation: "> docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml --cores 1"
- [methods] SNP filtering parameters: "* dp: 50 # SNP filtering depth
* qual: 60 # SNP filtering quality"
- [discussion] Live-cell limitation: "requires live cells for successful genetic separation, which consequently limits the application to biological mixtures containing living cells"
