---
name: snp-based-mixture-deconvolution
description: Use when 10x scRNA-seq BAM + barcodes.tsv from a multi-person mixture
  must be separated into per-individual cell clusters and each cluster genotyped/identified
  (sex, ancestry, LR-based ID) for forensic purposes.
when_to_use_negative:
- The mixture contains non-living or fixed cells — the 10X Genomics scRNA-seq platform
  used here requires live cells for successful genetic separation.
- Mixtures of more than nine individuals or fewer than ~150 cells with deeper sequencing
  were only hypothesized, not empirically confirmed — treat results as speculative
  beyond the validated 2–9 person blood mixtures.
- Input is bulk (not single-cell) DNA profiling data — this skill depends on per-cell
  SNP extraction from 10x scRNA-seq BAM/barcodes files.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: De-goulash
  role: Two-step Snakemake pipeline (deconvolution + statistical analysis) for SNP-based
    cell clustering and forensic identification
  repo: https://github.com/genid/de-goulash.git
- name: freebayes
  role: Haplotype-based SNP variant calling from the 10x BAM over parallel genome
    regions
  repo: https://github.com/nh13/freebayes
- name: Snakemake
  role: Workflow manager executing both pipeline steps inside the container
- name: Docker
  role: Container (geniderasmusmc/de-goulash:1) that runs the pipeline without other
    dependencies
- name: Yleaf
  role: Y-haplogroup inference from cluster variant calls in step 2 (read_depth 1,
    quality 20, base_calling 90)
- name: R
  role: 'UMAP-based cell clustering Rscript (n_neighbors: 5, n_components: 300); R
    3.6.1 tested'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:a48c94a8f43e98d4651856df8a6b465610c92f1b9af7b18f70927b4897bf2c02
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# snp-based-mixture-deconvolution

## Summary

De-goulash deconvolves multi-person biological mixtures (2–9 blood contributors, ratios up to 1:60) by calling SNPs per cell from 10x scRNA-seq data, clustering cells by contributing individual, and performing forensic characterization and individual identification against a WES/1000G reference. Rationale: separating the mixture by contributor first converts downstream DNA analysis to a single-source problem.

## When to use

Use when 10x scRNA-seq BAM + barcodes.tsv from a multi-person mixture must be separated into per-individual cell clusters and each cluster genotyped/identified (sex, ancestry, LR-based ID) for forensic purposes.

## When NOT to use

- The mixture contains non-living or fixed cells — the 10X Genomics scRNA-seq platform used here requires live cells for successful genetic separation.
- Mixtures of more than nine individuals or fewer than ~150 cells with deeper sequencing were only hypothesized, not empirically confirmed — treat results as speculative beyond the validated 2–9 person blood mixtures.
- Input is bulk (not single-cell) DNA profiling data — this skill depends on per-cell SNP extraction from 10x scRNA-seq BAM/barcodes files.

## Inputs

- Possorted_genome_bam.bam from 10x (possorted_genome_trimmed.bam)
- barcodes.tsv from 10x (or barcodes_reduced.txt for sensitivity testing)
- Human reference genome FASTA (hg19/hg38) and MT.fasta (same build)
- regions.txt and MT_regions.txt for parallel freebayes
- config.yaml with all pipeline parameters
- exome_96_remmapedto38.vcf.gz (de-novo WES reference)
- 1000G release 20130502 chromosome VCFs and 1000G_populations.txt
- Yleaf WGS_hg38 positions file

## Outputs

- cells_merge_clusters.vcf — per-cluster variant calls (output/iter2/cells_merge_clusters.vcf)
- Per-cell-cluster assignments (clusters of cells separated by contributing individual)
- Likelihood ratios for individual identification against WES reference data
- Biogeographic ancestry assignment (maternal, paternal, bi-parental) and sex determination
- mtDNA haplogroups (via haplogrep) and Y-haplogroups (via Yleaf)

## How to apply

Run de-goulash step 1 (deconvolution) as a Snakemake pipeline in Docker with config.yaml: filter SNPs at dp: 50 and qual: 60, call variants with freebayes over parallel regions (generated with fasta_generate_regions.py), apply iteration thresholds thr_cell_1: 10 and thr_cell_2: 20 (minimal SNPs per cell), and merge cells using threshold_coverage: 10, threshold_coverage_pos: 5, threshold_base_calling: 90; cluster cells via UMAP Rscript (n_neighbors: 5, n_components: 300). Then run step 2 (Snakefile_analysis) with dp_2: 50, qual_2: 60, exome reference exome_96_remmapedto38.vcf.gz, 1000G release 20130502 population data for likelihood-ratio computation and ancestry assignment, haplogrep for mtDNA, and Yleaf v2.2 (read_depth: 1, quality: 20, base_calling: 90, WGS_hg38 positions file) for Y-haplogroup inference. For cell-number sensitivity, subsample 10–500 barcodes from barcodes_reduced.txt per cluster in 10 replicates, rerun both steps, and check matching-SNP percentage, identification stability, and LR trends against thresholds.

## Related tools

- **De-goulash** (Two-step Snakemake pipeline (deconvolution + statistical analysis) for SNP-based cell clustering and forensic identification) — https://github.com/genid/de-goulash.git
- **freebayes** (Haplotype-based SNP variant calling from the 10x BAM over parallel genome regions) — https://github.com/nh13/freebayes
- **Snakemake** (Workflow manager executing both pipeline steps inside the container)
- **Docker** (Container (geniderasmusmc/de-goulash:1) that runs the pipeline without other dependencies)
- **Yleaf** (Y-haplogroup inference from cluster variant calls in step 2 (read_depth 1, quality 20, base_calling 90))
- **R** (UMAP-based cell clustering Rscript (n_neighbors: 5, n_components: 300); R 3.6.1 tested)

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml
```

## Evaluation signals

- Clustering shows clear single-cell separation by contributing individual across 2- to 9-person mixtures up to ratio 1:60.
- For cell-count sensitivity, matching-SNP percentage stays above 90% for all subsets of more than 20 cells.
- Identification stabilises above 50 cells and all M4 clusters exceed the conservative 10E+9 likelihood-ratio threshold with 100–200 cells.
- Sex and biogeographic ancestry (maternal, paternal, bi-parental) are correctly determined for all separated contributors.
- Pipeline completes without other dependencies inside the de-goulash:1 container, writing cells_merge_clusters.vcf under output/iter2/.

## Limitations

- The 10X Genomics scRNA-seq platform requires live cells, limiting application to biological mixtures containing living cells; platforms compatible with fixed cells are untested.
- Deconvolution of mixtures with more than nine individuals was not experimentally tested, only hypothesized.
- No universal identity SNPs are used, so reference datasets must contain as many SNPs as possible; WES data was used, whole transcriptome sequencing of references proposed but not done.
- Appearance prediction was not performed and would require specific SNPs not obtained here; transferability to mixtures other than blood is not yet demonstrated.

## Evidence

- [readme] readme-pipeline-definition: "de-goulash is a bioinformatics pipeline build in [Snakemake](https://snakemake.readthedocs.io/en/stable/) which allows clustering mixed individuals using 10x single-cell RNA-seq."
- [readme] readme-config-thresholds: "* thr_cell_1: 10 #Minimal number of SNPs per cell
#### threshold for iteration 2
* thr_cell_2: 20 #Minimal number of SNPs per cell"
- [readme] readme-docker-usage: "We provided a docker image where you can run the pipeline without having to install any other dependency than docker."
- [abstract] abstract-proof-of-concept: "For all 2 up to 9-person balanced and imbalanced blood mixtures with ratios up to 1:60, we achieved a clear single-cell separation according to the contributing individuals."
- [methods] methods-docker: "You can execute de-goulash Snakemake pipeline throught docker image-container"
- [discussion] discussion-live-cells: "requires live cells for successful genetic separation, which consequently limits the application to biological mixtures containing living cells"
