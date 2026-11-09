---
name: forensic-mixture-separation-validation
description: Use when validating or stress-testing a single-cell mixture deconvolution
  pipeline against in silico mixtures with known contributor ratios (e.g. 1:10–1:99)
  and ground-truth cell-to-individual labels.
when_to_use_negative:
- Mixtures of more than nine contributors or fewer than ~150 cells — these were not
  empirically tested and success is only hypothesized.
- Biological material without live cells — the 10x Genomics scRNA-seq platform used
  by de-goulash requires live cells, so fixed or non-living mixtures are out of scope.
- Single-source samples with no mixture component to separate — there is no deconvolution
  ground truth to validate against.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: de-goulash
  role: 'Snakemake pipeline for genotyping and clustering mixed single-cell samples
    (step 1: deconvolution)'
  repo: https://github.com/genid/de-goulash.git
- name: freebayes
  role: haplotype-based SNP variant calling on per-cell BAMs across the regions file
  repo: https://github.com/nh13/freebayes
- name: Snakemake
  role: orchestrates the two-step de-goulash pipeline from config.yaml parameters
- name: Docker
  role: runs the de-goulash pipeline via the geniderasmusmc/de-goulash:1 image without
    other dependencies
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:55d59132f6c2e384e6651fc7256432f8243c479f55d809f5f72c2d15bf3572dd
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
  - de-goulash
  - freebayes
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# forensic-mixture-separation-validation

## Summary

Validation skill for benchmarking single-cell genetic deconvolution of forensic biological mixtures against known ground-truth contributor ratios, using the de-goulash Snakemake/Docker pipeline on 10x scRNA-seq data. It establishes the sensitivity limit of correct 2-person separation (clean to 1:40, minor bleed-through at 1:60, failure at 1:99).

## When to use

Use when validating or stress-testing a single-cell mixture deconvolution pipeline against in silico mixtures with known contributor ratios (e.g. 1:10–1:99) and ground-truth cell-to-individual labels.

## When NOT to use

- Mixtures of more than nine contributors or fewer than ~150 cells — these were not empirically tested and success is only hypothesized.
- Biological material without live cells — the 10x Genomics scRNA-seq platform used by de-goulash requires live cells, so fixed or non-living mixtures are out of scope.
- Single-source samples with no mixture component to separate — there is no deconvolution ground truth to validate against.

## Inputs

- possorted_genome_bam.bam (10x single-cell alignment file) per mixture
- barcodes.tsv from 10x listing the cells to include
- genome.fasta (human reference, e.g. hg19/hg38) plus *MT.fasta mitochondrial sequence
- regions.txt and MT_regions.txt for parallel freebayes (generated with process_reference.py / fasta_generate_regions.py)
- config.yaml holding sample_bam, barcodes, reference paths, and all threshold parameters

## Outputs

- output/iter2/cells_merge_clusters.vcf — genotyped, clustered cells per mixture ratio
- per-ratio separation performance assessment (correct vs. misassigned cells per contributor cluster)

## How to apply

Assemble in silico two-person mixtures from single-individual 10x scRNA-seq datasets at imbalanced ratios spanning 1:10, 1:20, 1:40, 1:60, 1:80, and 1:99. Point config.yaml at each mixture's possorted_genome_bam.bam, barcodes.tsv, genome.fasta (plus *MT.fasta), and the freebayes regions file generated with fasta_generate_regions.py, keeping default SNP filtering (dp: 50, qual: 60). Run de-goulash Snakemake step 1 inside the geniderasmusmc/de-goulash:1 docker container with two threshold iterations (thr_cell_1: 10, thr_cell_2: 20 minimal SNPs per cell), apply the cell-merging rule (threshold_coverage: 10, threshold_coverage_pos: 5, threshold_base_calling: 90), and cluster merged cells with the Rscript rule using UMAP settings n_neighbors: 5, n_components: 300, clusters: 0. Write the genotyped, clustered cells per ratio to output/iter2/cells_merge_clusters.vcf and judge separation by comparing clustering assignments against the known contributor labels, expecting clean separation at 1:10–1:40, minor bleed-through at 1:60 (about five minor-component cells misassigned), degraded assignment at 1:80, and no separation at 1:99.

## Related tools

- **de-goulash** (Snakemake pipeline for genotyping and clustering mixed single-cell samples (step 1: deconvolution)) — https://github.com/genid/de-goulash.git
- **freebayes** (haplotype-based SNP variant calling on per-cell BAMs across the regions file) — https://github.com/nh13/freebayes
- **Snakemake** (orchestrates the two-step de-goulash pipeline from config.yaml parameters)
- **Docker** (runs the de-goulash pipeline via the geniderasmusmc/de-goulash:1 image without other dependencies)

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml --cores 1
```

## Evaluation signals

- Expected degradation curve reproduced: clean contributor separation at 1:10–1:40, five minor-component cells misassigned at 1:60, degraded assignment at 1:80, no separation at 1:99
- Config.yaml contains the paper-default thresholds: dp: 50, qual: 60, thr_cell_1: 10, thr_cell_2: 20
- Merging thresholds honored: threshold_coverage: 10, threshold_coverage_pos: 5, threshold_base_calling: 90
- UMAP clustering used n_neighbors: 5 and n_components: 300 with clusters: 0 (no nbclust prediction)
- Final output per ratio written to output/iter2/cells_merge_clusters.vcf as the Snakemake target

## Limitations

- Correct deconvolution fails entirely at the most extreme 1:99 imbalance; degraded assignment already occurs at 1:80 with minor bleed-through at 1:60
- The 10x scRNA-seq platform requires live cells, restricting validation to mixtures containing living cells
- Deeper sequencing or fewer than ~150 cells for successful deconvolution is proposed but requires future empirical confirmation

## Evidence

- [abstract] clear single-cell separation up to 1:60: "For all 2 up to 9-person balanced and imbalanced blood mixtures with ratios up to 1:60, we achieved a clear single-cell separation according to the contributing individuals."
- [readme] de-goulash is a Snakemake pipeline for 10x scRNA-seq clustering: [evidence span withheld — anchor status: not_found]
- [readme] threshold iterations 1 and 2: "thr_cell_1: 10 #Minimal number of SNPs per cell"
- [readme] cell merging thresholds: "threshold_coverage: 10 #treshhold total coverage of selected SNPs per cell"
- [readme] docker invocation for deconvolution step: "docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf"
