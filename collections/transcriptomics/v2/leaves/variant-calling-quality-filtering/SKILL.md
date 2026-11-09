---
name: variant-calling-quality-filtering
description: Use when possorted_genome_bam and barcodes files from 10x scRNA-seq of
  a cell mixture must be genotyped and quality-filtered to yield per-cell SNP counts
  for contributor clustering.
when_to_use_negative:
- The mixture contains fixed or non-living cells — the 10x genomics scRNA-seq platform
  requires live cells for successful genetic separation
- The goal is downstream individual identification and biogeographical ancestry assignment
  rather than separation — use de-goulash Snakemake step 2 (Snakefile_analysis) instead
- Per-cell genotype data or cluster VCFs already exist — re-running variant calling
  adds no information
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: freebayes
  role: Haplotype-based SNP/variant caller run in parallel over generated region files
    against genome.fasta and MT.fasta
  repo: https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py
- name: de-goulash
  role: Snakemake pipeline encoding the variant calling, SNP/coverage filtering (merging-cells
    Python rule) and clustering rules for both deconvolution iterations
  repo: https://github.com/genid/de-goulash.git
- name: Snakemake
  role: Workflow engine executing the deconvolution rules (variant calling, merging
    cells, clustering Rscript) defined in Snakefile with config.yaml
  repo: https://snakemake.readthedocs.io/en/stable/
- name: Docker
  role: Runs the pipeline via the geniderasmusmc/de-goulash:1 image with the working
    directory mounted to /single-cell, requiring no other dependencies
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:4de331c02eed325b1d2f8cdd81519c089338ead8c7083895755e72ccb9d5c74c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - De-goulash
  - Docker
  - Python
  - R
  - Snakemake
  - de-goulash
  - freebayes
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# variant-calling-quality-filtering

## Summary

Haplotype-based SNP calling on 10x scRNA-seq alignments with freebayes, followed by depth/quality/base-calling filtering, to obtain per-cell SNP genotypes sufficient to genetically separate contributors of multi-person biological mixtures within the de-goulash two-iteration deconvolution pipeline.

## When to use

Use when possorted_genome_bam and barcodes files from 10x scRNA-seq of a cell mixture must be genotyped and quality-filtered to yield per-cell SNP counts for contributor clustering.

## When NOT to use

- The mixture contains fixed or non-living cells — the 10x genomics scRNA-seq platform requires live cells for successful genetic separation
- The goal is downstream individual identification and biogeographical ancestry assignment rather than separation — use de-goulash Snakemake step 2 (Snakefile_analysis) instead
- Per-cell genotype data or cluster VCFs already exist — re-running variant calling adds no information

## Inputs

- possorted_genome_bam.bam (10x scRNA-seq alignment, optionally trimmed)
- barcodes.tsv / barcodes_reduced.txt listing cells to use onwards
- genome.fasta (human reference, e.g. hg19 or hg38)
- MT.fasta (mitochondrial DNA sequence, same build as genome)
- regions.txt and MT_regions.txt (region files for parallel freebayes)
- config.yaml with filtering parameters (dp, qual, thr_cell_1, thr_cell_2, threshold_coverage, threshold_coverage_pos, threshold_base_calling)

## Outputs

- Per-cell SNP genotype/VCF calls passing dp=50, qual=60 filters (output/iter1 and output/iter2/cells_merge_clusters.vcf)
- Merged per-cell SNP matrix retaining only cells meeting coverage and base-calling thresholds
- Fraction of cells separated per iteration (e.g. M2: 21.3% at iteration 1, ~97% at iteration 2)
- Contributor cluster assignments from UMAP-based clustering

## How to apply

Configure config.yaml with sample_bam, barcodes, genome.fasta, MT.fasta and region files (regions.txt and MT_regions.txt generated via python process_reference.py or fasta_generate_regions.py), then run de-goulash Snakemake step 1 in the geniderasmusmc/de-goulash:1 Docker container, letting freebayes call variants in parallel over region files. Filter SNPs with depth dp=50 and quality qual=60. In the merging-cells Python rule, retain only cells passing threshold_coverage=10 (total coverage of selected SNPs per cell), threshold_coverage_pos=5 (coverage per selected SNP per cell), and threshold_base_calling=90 (base-calling quality). For iteration 1 (mtDNA-SNP based) require thr_cell_1=10 minimal SNPs per cell; for iteration 2 (genome-wide-SNP based, targeting output/iter2/cells_merge_clusters.vcf) require thr_cell_2=20. Feed the filtered per-cell SNP matrix to the clustering Rscript (UMAP, n_neighbors=5, n_components=300) to assign cells to contributor clusters.

## Related tools

- **freebayes** (Haplotype-based SNP/variant caller run in parallel over generated region files against genome.fasta and MT.fasta) — https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py
- **de-goulash** (Snakemake pipeline encoding the variant calling, SNP/coverage filtering (merging-cells Python rule) and clustering rules for both deconvolution iterations) — https://github.com/genid/de-goulash.git
- **Snakemake** (Workflow engine executing the deconvolution rules (variant calling, merging cells, clustering Rscript) defined in Snakefile with config.yaml) — https://snakemake.readthedocs.io/en/stable/
- **Docker** (Runs the pipeline via the geniderasmusmc/de-goulash:1 image with the working directory mounted to /single-cell, requiring no other dependencies)

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml --cores 1
```

## Evaluation signals

- Filtered variants respect the configured SNP thresholds: depth dp=50 and quality qual=60
- Retained cells meet threshold_coverage=10 total coverage, threshold_coverage_pos=5 coverage per SNP, and threshold_base_calling=90
- Iteration 1 cells carry at least thr_cell_1=10 SNPs per cell; iteration 2 at least thr_cell_2=20 SNPs per cell
- Separated fractions match reported values: M2 ~21.3% (iteration 1) rising to ~97% (iteration 2); M3 ~96.5% and M4 ~98% after iteration 2
- Clustering Rscript runs with n_neighbors=5 and n_components=300, and nBclust is only executed when clusters > 1 is configured

## Limitations

- The 10x genomics scRNA-seq platform requires live cells, limiting application to biological mixtures containing living cells; alternative platforms for fixed cells have not been tested
- Separation from fewer than 150 cells or with deeper sequencing/coverage has not been empirically confirmed
- Deconvolution was only validated for 2- to 9-person mixtures (ratios up to 1:60); more than nine individuals is hypothesized but untested
- Cells failing the coverage/base-calling thresholds (e.g. low-SNP cells below thr_cell) are dropped, reducing the fraction of separated cells

## Evidence

- [readme] SNP filtering thresholds dp/qual and per-cell thresholds: "* dp: 50 # SNP filtering depth"
- [readme] Per-cell coverage and base-calling filters in merging rule: "* threshold_coverage: 10 #treshhold total coverage of selected SNPs per cell"
- [methods] Parallel freebayes over generated region files: "regions: /single-cell/input/reference/regions.txt #for parallel freebayes"
- [readme] Generation of region files required by parallel freebayes: "region file can be generated with https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py"
