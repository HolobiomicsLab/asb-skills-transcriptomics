---
name: forensic-mixture-contributor-characterization
description: Use when scRNA-seq reads from a multi-person mixture have already been
  deconvoluted into per-individual cell clusters and each cluster must be assigned
  sex, maternal/paternal haplogroups, and ancestry fractions.
when_to_use_negative:
- 'Cells are not living: the 10X Genomics scRNA-seq platform the pipeline builds on
  requires live cells for successful genetic separation'
- 'Clusters have not yet been deconvoluted: this skill consumes de-goulash step 1
  cluster VCF output and cannot separate mixture contributors itself'
- 'The biological mixture is not blood: transferability to mixtures other than blood
  was not demonstrated in this study'
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0102
tools:
- name: De-goulash
  role: Snakemake pipeline whose step 2 (Snakefile_analysis) performs forensic statistical
    analysis, haplogroup assignment, and ancestry estimation on cluster variant calls
  repo: https://github.com/genid/de-goulash
- name: Snakemake
  role: Workflow engine executing the analysis Snakefile_analysis with config.yaml
    inside the Docker container
- name: Docker
  role: Container (geniderasmusmc/de-goulash:1) in which the analysis runs without
    additional dependency installation
- name: Yleaf
  role: Y-chromosome haplogroup prediction (v2.2) using read_depth 1, quality 20,
    base_calling 90, and WGS_hg38.txt positions file
- name: R 3.6.1
  role: Runtime for clustering and analysis scripts within the pipeline
- name: Python 3.7.3
  role: Runtime for pipeline helper scripts (e.g., cell merging)
- name: Java Run time environment 8
  role: Runtime required for Haplogrep2 mtDNA haplogroup assignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:7071135aa31bf6b347f923f47857b33b6c089379c54bf6fa1759a1e7b68e6095
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genid/de-goulash
  tools:
  - De-goulash
  - Docker
  - Java Run time environment 8
  - Python
  - Python 3.7.3
  - R
  - R 3.6.1
  - Snakemake
  - Yleaf
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# forensic-mixture-contributor-characterization

## Summary

Genetically characterizes each deconvoluted single-cell cluster from a multi-person blood mixture for sex, mtDNA/Y-chromosome haplogroups, and bi-parental ancestry using the de-goulash statistical analysis step (Snakemake step 2) run in Docker, with Haplogrep2 and Yleaf v2.2.

## When to use

Use when scRNA-seq reads from a multi-person mixture have already been deconvoluted into per-individual cell clusters and each cluster must be assigned sex, maternal/paternal haplogroups, and ancestry fractions.

## When NOT to use

- Cells are not living: the 10X Genomics scRNA-seq platform the pipeline builds on requires live cells for successful genetic separation
- Clusters have not yet been deconvoluted: this skill consumes de-goulash step 1 cluster VCF output and cannot separate mixture contributors itself
- The biological mixture is not blood: transferability to mixtures other than blood was not demonstrated in this study

## Inputs

- Per-cluster variant calls from de-goulash deconvolution step 1 (e.g., output/iter2/cells_merge_clusters.vcf)
- X-chromosome (XIST) and Y-chromosome read counts per cluster
- Mitochondrial variant calls per cluster
- Whole exome reference VCF exome_96_remmapedto38.vcf.gz
- 1000 Genomes release 20130502 chromosomes in /single-cell/input/1000G/ with 1000G_populations.txt
- config.yaml with analysis parameters (dp_2: 50, qual_2: 60, Yleaf read_depth 1, quality 20, base_calling 90, positions_file WGS_hg38.txt)

## Outputs

- Sex assignment per cluster (male/female) from XIST and Y-chromosome evidence
- mtDNA haplogroup per cluster (e.g., L2a1j, U5b2b4a) from Haplogrep2
- Y-chromosome haplogroup per male cluster (e.g., E1b1b1b2a1a1) from Yleaf
- Bi-parental ancestry fractions per cluster from STRUCTURE (e.g., 53.6% African / 44.6% European; 99.2% European)

## How to apply

Run the de-goulash statistical analysis pipeline (Snakefile_analysis with config.yaml, dirpath_analysis set to 'output') through the Docker image geniderasmusmc/de-goulash:1, mounting the working directory to /single-cell. Assign sex of each cluster from XIST and Y-chromosome read evidence; determine mtDNA haplogroups from cluster mitochondrial variants with Haplogrep2; determine the Y haplogroup with Yleaf v2.2 using read_depth 1, quality 20, base_calling 90, and the WGS_hg38.txt positions file. Estimate bi-parental ancestry fractions with STRUCTURE against the 1000 Genomes release 20130502 data in /single-cell/input/1000G/ using 1000G_populations.txt as the population reference. Accept results only when sex, mtDNA haplogroups, Y haplogroup, and ancestry fractions match known contributor characteristics (e.g., in M2: cluster 1 = male, mtDNA L2a1j, Y E1b1b1b2a1a1, 53.6% African / 44.6% European; cluster 2 = female, mtDNA U5b2b4a, 99.2% European).

## Related tools

- **De-goulash** (Snakemake pipeline whose step 2 (Snakefile_analysis) performs forensic statistical analysis, haplogroup assignment, and ancestry estimation on cluster variant calls) — https://github.com/genid/de-goulash
- **Snakemake** (Workflow engine executing the analysis Snakefile_analysis with config.yaml inside the Docker container)
- **Docker** (Container (geniderasmusmc/de-goulash:1) in which the analysis runs without additional dependency installation)
- **Yleaf** (Y-chromosome haplogroup prediction (v2.2) using read_depth 1, quality 20, base_calling 90, and WGS_hg38.txt positions file)
- **R 3.6.1** (Runtime for clustering and analysis scripts within the pipeline)
- **Python 3.7.3** (Runtime for pipeline helper scripts (e.g., cell merging))
- **Java Run time environment 8** (Runtime required for Haplogrep2 mtDNA haplogroup assignment)

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 --snakefile Snakefile_analysis --configfile config.yaml --cores 1
```

## Evaluation signals

- Sex assignment agrees with XIST and Y-chromosome read evidence for every cluster
- mtDNA haplogroups from Haplogrep2 reproduce expected values (M2: cluster 1 = L2a1j, cluster 2 = U5b2b4a)
- Yleaf haplogroup matches expectation for the male cluster (M2: E1b1b1b2a1a1)
- STRUCTURE ancestry fractions match reported values (M2: cluster 1 = 53.6% African / 44.6% European; cluster 2 = 99.2% European)
- Characterizations agree with a priori contributor knowledge of the mixture (African male and European female)

## Limitations

- The 10X Genomics scRNA-seq platform requires live cells, restricting application to mixtures containing living cells
- Appearance trait prediction was not performed and would require specific SNPs not obtained here
- Reference datasets must contain as many SNPs as possible because no universal identity SNPs are used; WES reference data was used to solve this
- Only validated on blood mixtures with 2 to 9 contributors; transferability to other biological mixtures not yet demonstrated

## Evidence

- [abstract] Sex and bio-geographic ancestry determination: "For all separated mixture contributors, sex and bio-geographic ancestry (maternal, paternal, and bi-parental) were correctly determined."
- [readme] Pipeline structure: "Individual genetic identification and biogeographical ancestry assigment. It requires the output variant calling for each assignated cluster from step 1 and it will calculate likelihood of forensic"
- [readme] Yleaf parameters: "* read_depth: 1
* quality: 20
* base_calling: 90
* positions_file: /single-cell/software/Yleaf/Position_files/WGS_hg38.txt"
- [readme] 1000G reference input: "Path where the chromosomes for 1000G variant calling: /single-cell/input/1000G/ [https://ftp-trace.ncbi.nih.gov/1000genomes/ftp/release/20130502/]"
- [readme] Analysis invocation: "> docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 --snakefile Snakefile_analysis --configfile config.yaml --cores 1"
