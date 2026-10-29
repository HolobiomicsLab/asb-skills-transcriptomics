---
name: vcf-variant-comparison-against-wes-reference
description: Use when a deconvoluted single-cell cluster VCF (e.g. cells_merge_clusters.vcf)
  must be matched against a WES reference VCF to compute matching-SNP fractions and
  identification likelihood ratios for each contributor.
when_to_use_negative:
- The query is sex or bio-geographic ancestry assignment rather than individual identification
  — those use the 1000G population analysis and Yleaf/haplogrep branches of step 2,
  not WES-reference matching
- The reference sample has not been WES-sequenced (or similarly densely genotyped)
  — the paper notes that because no universal identity SNPs are used, reference datasets
  must contain as many SNPs as possible
- Cells are not living (the 10X Genomics scRNA-seq platform requires live cells for
  successful genetic separation, so no cluster VCF can be produced)
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2885
tools:
- name: De-goulash
  role: Dedicated Snakemake pipeline whose Snakemake step 2 (Snakefile_analysis) performs
    the VCF-to-WES-reference matching, forensic likelihood calculation, and population
    assignment
  repo: https://github.com/genid/de-goulash
- name: Snakemake
  role: Workflow engine executing the statistical analysis step with config.yaml parameters
- name: Docker
  role: Container (geniderasmusmc/de-goulash:1) that packages the pipeline so only
    Docker needs to be installed; the analysis step is run via docker run with the
    current directory mounted at /single-cell
- name: freebayes
  role: Haplotype-based variant caller producing the per-cell and cluster VCFs that
    feed the comparison (via parallel region files from fasta_generate_regions.py)
  repo: https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:5ad85405ce5531a2a2a4d30a3f85b97353b93beed0b57b969955ecea3b997518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genid/de-goulash
  tools:
  - De-goulash
  - Docker
  - Python
  - R
  - Snakemake
  - freebayes
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# vcf-variant-comparison-against-wes-reference

## Summary

Matches identity SNPs from deconvoluted scRNA-seq cell-cluster VCFs against de-novo whole-exome sequencing (WES) reference VCFs to individually identify mixture contributors with likelihood ratios far above the court-acceptable 10E+9 threshold.

## When to use

Use when a deconvoluted single-cell cluster VCF (e.g. cells_merge_clusters.vcf) must be matched against a WES reference VCF to compute matching-SNP fractions and identification likelihood ratios for each contributor.

## When NOT to use

- The query is sex or bio-geographic ancestry assignment rather than individual identification — those use the 1000G population analysis and Yleaf/haplogrep branches of step 2, not WES-reference matching
- The reference sample has not been WES-sequenced (or similarly densely genotyped) — the paper notes that because no universal identity SNPs are used, reference datasets must contain as many SNPs as possible
- Cells are not living (the 10X Genomics scRNA-seq platform requires live cells for successful genetic separation, so no cluster VCF can be produced)

## Inputs

- Merged cell-cluster VCF from de-goulash step 1 (output/iter2/cells_merge_clusters.vcf) for mixtures M2-M5 and M9
- De-novo WES reference VCF (exome_96_remmapedto38.vcf.gz, hg38-remapped)
- 1000 Genomes population file (1000G_populations.txt, release 20130502)
- config.yaml with analysis-step filter parameters (dp_2: 50, qual_2: 60)

## Outputs

- Per cluster-contributor matching-SNP fraction (expected >90% shared SNPs with the WES reference)
- Likelihood ratios of individual identification (expected range 1.71E+14 to 5.00E+65, all above 10E+9)

## How to apply

Run the de-goulash statistical analysis step (Snakefile_analysis) via the geniderasmusmc/de-goulash:1 Docker container, loading the merged cell-cluster VCF (output/iter2/cells_merge_clusters.vcf) from Snakemake step 1 as the query SNP set for each cluster of mixtures M2-M5 and M9. Provide the de-novo WES reference (ref_exome: exome_96_remmapedto38.vcf.gz) and the 1000 Genomes population file (1000G_populations.txt) as references. Apply the analysis-step SNP filters from config.yaml (dp_2: 50 depth, qual_2: 60 quality) to cluster SNPs before comparison, since the WES reference contains no universal identity SNPs and matching relies on as many shared SNPs as possible. For each separated cluster-contributor pair, compute the fraction of matching SNPs against the WES reference profile and the likelihood ratio of identification; expect >90% shared SNPs and LR values of 1.71E+14 to 5.00E+65, including in mixtures with up to 9 individuals. Validate that every identification exceeds the conservative 10E+9 court-acceptable threshold.

## Related tools

- **De-goulash** (Dedicated Snakemake pipeline whose Snakemake step 2 (Snakefile_analysis) performs the VCF-to-WES-reference matching, forensic likelihood calculation, and population assignment) — https://github.com/genid/de-goulash
- **Snakemake** (Workflow engine executing the statistical analysis step with config.yaml parameters)
- **Docker** (Container (geniderasmusmc/de-goulash:1) that packages the pipeline so only Docker needs to be installed; the analysis step is run via docker run with the current directory mounted at /single-cell)
- **freebayes** (Haplotype-based variant caller producing the per-cell and cluster VCFs that feed the comparison (via parallel region files from fasta_generate_regions.py)) — https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 --snakefile Snakefile_analysis --configfile config.yaml --cores 1
```

## Evaluation signals

- Every separated contributor matches its WES reference sample with >90% shared SNPs
- All likelihood ratios exceed the court-acceptable identification threshold of 10E+9 (expected range 1.71E+14 to 5.00E+65)
- Identification succeeds for every cluster of mixtures M2 through M9, including mixtures with up to 9 individuals
- Cluster SNPs were filtered with dp_2: 50 (depth) and qual_2: 60 (quality) before comparison, matching config.yaml analysis-step settings
- Correct contributor-to-cluster pairing: each identified reference sample corresponds to the same individual assigned during de-goulash deconvolution/clustering in step 1

## Limitations

- Mixtures with more than nine individuals were not experimentally tested; >9-person deconvolution is only hypothesized to be feasible depending on SNPs obtained per cell cluster
- Because no universal identity SNPs are used, the reference dataset must contain as many SNPs as possible; whole transcriptome sequencing of reference samples is proposed to increase identity SNPs but was not done
- The 10X Genomics scRNA-seq platform requires live cells, restricting application to mixtures containing living cells
- Transferability to biological mixtures other than blood has not yet been demonstrated

## Evidence

- [abstract] All separated contributors were correctly individually identified with court-acceptable statistical certainty using de-novo generated whole exome sequencing reference data.: "All separated contributors were correctly individually identified with court-acceptable statistical certainty using de-novo generated whole exome sequencing reference data."
- [methods] ref_exome: /single-cell/input/exome_96_remmapedto38.vcf.gz: "ref_exome: /single-cell/input/exome_96_remmapedto38.vcf.gz"
- [methods] dp_2: 50 #SNP filtering depth; qual_2: 60 #SNP filtering quality: "* dp_2: 50 #SNP filtering depth
* qual_2: 60 #SNP filtering quality"
- [readme] README: de-goulash statistical analysis invocation: "> docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 --snakefile Snakefile_analysis --configfile config.yaml --cores 1"
- [discussion] Whole transcriptome sequencing could also be used on the reference samples, which would increase the number of identity SNPs available for genetic matching.: "Whole transcriptome sequencing could also be used on the reference samples, which would increase the number of identity SNPs available for genetic matching."
