---
name: likelihood-ratio-forensic-identification
description: Use when separated cell clusters from a mixed scRNA-seq sample must be
  attributed to specific contributors by matching cluster SNP profiles against WES
  reference VCFs, yielding a likelihood ratio per contributor.
when_to_use_negative:
- When no reference dataset (WES or transcriptome sequencing) exists for candidate
  contributors — the approach does not use universal identity SNPs, so references
  must contain as many SNPs as possible
- 'When cell clusters contain too few cells: sensitivity analysis showed matching
  SNPs stay above 90% only for sampled points with more than 20 cells, and the 10E+9
  LR threshold is exceeded at 100-200 cells'
- When the goal is appearance-trait prediction — the required specific SNPs were not
  obtained and this extension was not performed
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0625
tools:
- name: De-goulash
  role: Snakemake statistical analysis step (Snakefile_analysis) computing likelihood
    of forensic parameters, population assignment, HaploGrep and Yleaf execution
  repo: https://github.com/genid/de-goulash
- name: Docker
  role: Container image geniderasmusmc/de-goulash:1 used to execute the analysis pipeline
    without other dependencies
- name: Yleaf
  role: 'Y-chromosome haplogroup/sex determination within the analysis step (read_depth:
    1, quality: 20, base_calling: 90)'
- name: freebayes
  role: Upstream haplotype-based variant calling producing the cluster SNP sets queried
    in the identification step
  repo: https://github.com/nh13/freebayes
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:c0b9d8fe26188a82462db51673c09cfea5e24933b9c5a4a9ec15c435908dec54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genid/de-goulash
  tools:
  - De-goulash
  - Docker
  - Python
  - R
  - Yleaf
  - freebayes
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# likelihood-ratio-forensic-identification

## Summary

Computes forensic likelihood ratios by matching identity SNPs from deconvoluted single-cell clusters against a de-novo whole-exome sequencing (WES) reference profile, using de-goulash's statistical analysis step with 1000G population data. Establishes individual identification of mixture contributors with court-acceptable statistical certainty (LR > 10E+9).

## When to use

Use when separated cell clusters from a mixed scRNA-seq sample must be attributed to specific contributors by matching cluster SNP profiles against WES reference VCFs, yielding a likelihood ratio per contributor.

## When NOT to use

- When no reference dataset (WES or transcriptome sequencing) exists for candidate contributors — the approach does not use universal identity SNPs, so references must contain as many SNPs as possible
- When cell clusters contain too few cells: sensitivity analysis showed matching SNPs stay above 90% only for sampled points with more than 20 cells, and the 10E+9 LR threshold is exceeded at 100-200 cells
- When the goal is appearance-trait prediction — the required specific SNPs were not obtained and this extension was not performed

## Inputs

- Merged cell-cluster VCF from de-goulash deconvolution step 1 (output/iter2/cells_merge_clusters.vcf)
- De-novo WES reference VCF (exome_96_remmapedto38.vcf.gz)
- 1000 Genomes population file (1000G_populations.txt, release 20130502)
- 1000G chromosome VCFs for variant calling (/single-cell/input/1000G/)
- config.yaml with analysis parameters (dp_2: 50, qual_2: 60, Yleaf settings)

## Outputs

- Per-cluster fraction of matching SNPs against the WES reference (>90% expected for true contributors)
- Likelihood ratios of individual identification against the WES reference profile
- Biogeographical ancestry assignment (maternal, paternal, bi-parental) per contributor
- Sex / Y-haplogroup determination via Yleaf and HaploGrep

## How to apply

Run de-goulash statistical analysis step 2 (Snakefile_analysis) inside the Docker container (geniderasmusmc/de-goulash:1), loading the merged cell-cluster VCF (output/iter2/cells_merge_clusters.vcf) from Snakemake deconvolution step 1 as the query SNP set for each cluster of mixtures M2-M5 and M9. Apply the analysis-step SNP filters from config.yaml (dp_2: 50 depth, qual_2: 60 quality) to cluster SNPs before comparison, and match each cluster against the de-novo WES reference (ref_exome: /single-cell/input/exome_96_remmapedto38.vcf.gz) plus the 1000 Genomes population file (1000G_populations.txt) for likelihood-ratio calculation. The same step assigns biogeographical ancestry, runs HaploGrep, and executes Yleaf v2.2 (read_depth: 1, quality: 20, base_calling: 90, WGS_hg38 positions file) for sex/haplogroup determination. Decision criterion: a cluster-contributor match is accepted when the fraction of matching SNPs exceeds 90% and the likelihood ratio exceeds the conservative court-acceptable 10E+9 threshold; reported LRs ranged from 1.71E+14 to 5.00E+65, including mixtures with up to 9 contributors.

## Related tools

- **De-goulash** (Snakemake statistical analysis step (Snakefile_analysis) computing likelihood of forensic parameters, population assignment, HaploGrep and Yleaf execution) — https://github.com/genid/de-goulash
- **Docker** (Container image geniderasmusmc/de-goulash:1 used to execute the analysis pipeline without other dependencies)
- **Yleaf** (Y-chromosome haplogroup/sex determination within the analysis step (read_depth: 1, quality: 20, base_calling: 90))
- **freebayes** (Upstream haplotype-based variant calling producing the cluster SNP sets queried in the identification step) — https://github.com/nh13/freebayes

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 --snakefile Snakefile_analysis --configfile config.yaml --cores 1
```

## Evaluation signals

- Every separated contributor matches a WES reference sample with over 90% shared SNPs
- All likelihood ratios exceed the conservative court-acceptable 10E+9 threshold; reproduced values should fall in or near the reported range 1.71E+14 to 5.00E+65
- Sex and biogeographical ancestry (maternal, paternal, bi-parental) are correctly determined for all separated contributors
- Input filters are correctly applied: dp_2 = 50 read depth and qual_2 = 60 quality on cluster SNPs before reference comparison
- In the M4 sensitivity setting, matching SNPs stay above 90% for subsets with more than 20 cells and the LR threshold is exceeded with 100-200 cells

## Limitations

- No universal identity SNPs are used, so reference datasets must contain as many SNPs as possible; whole transcriptome sequencing of reference samples was proposed to increase matching SNPs but not done
- The 10X Genomics scRNA-seq platform requires live cells, limiting application to mixtures containing living cells
- Deconvolution from fewer than 150 cells (even with deeper sequencing) was not empirically confirmed; deconvolution of more than nine individuals was only hypothesized
- Genetic imputation for IGG/FGG public-database upload was only envisioned, not performed; national forensic DNA databases are not expected to include transcriptome/genome data soon

## Evidence

- [abstract] All separated contributors were correctly individually identified with court-acceptable statistical certainty using de-novo generated whole exome sequencing reference data.: "All separated contributors were correctly individually identified with court-acceptable statistical certainty using de-novo generated whole exome sequencing reference data."
- [readme] exome_96_remmapedto38.vcf.gz: "* ref_exome: /single-cell/input/exome_96_remmapedto38.vcf.gz"
- [readme] Yleaf parameters: "#### Yleaf parameters"
- [readme] dp_2 filtering: "* dp_2: 50 #SNP filtering depth"
- [readme] de-goulash statistical analysis command: "> docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 --snakefile Snakefile_analysis --configfile config.yaml --cores 1"
