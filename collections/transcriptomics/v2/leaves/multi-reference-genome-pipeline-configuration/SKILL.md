---
name: multi-reference-genome-pipeline-configuration
description: Use when predicting tumor neoantigens from WES/WGS plus optional RNA-Seq
  of matched tumor-normal pairs and you need to test or control for reference genome
  build (GRCh37 vs GRCh38) effects on variant calling and downstream pMHC predictions.
when_to_use_negative:
- The analysis requires TCR sequencing integration or TCR-pMHC binding prediction
  — the article states this is future work and not yet part of the TSNAD pipeline
- Input data is not paired WES/WGS (e.g., already-variant-called VCFs only) and you
  need the full raw-read-to-neoantigen one-stop workflow with its specific depth requirements
  (minimum 15x WGS / 50x WES)
- A single reference genome is fixed and re-running both builds is unnecessary — TSNAD
  v1.0 users, who support only GRCh37, cannot benefit from the GRCh38 branch
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TSNAD v2.0
  role: One-stop pipeline whose -V flag selects GRCh37 or GRCh38 for all embedded
    variant-calling and neoantigen-prediction modules
  repo: https://github.com/jiujiezz/tsnad
- name: Docker
  role: Deployment mechanism providing the biopharm/tsnad image preloaded with all
    tools and both reference genome files
  repo: https://hub.docker.com/r/biopharm/tsnad
- name: BWA
  role: Maps trimmed short reads to the user-selected reference genome (GRCh37 or
    GRCh38)
- name: GATK
  role: Recalibrates base quality and calls SNVs/INDELs with Mutect2 against the chosen
    reference build
  repo: https://github.com/broadinstitute/gatk
- name: VEP
  role: Annotates called variants at the amino-acid level using annotations consistent
    with the selected genome build
  repo: https://github.com/Ensembl/ensembl-vep
- name: DeepHLApan
  role: Predicts peptide-MHC binding and immunogenicity scores used to retain potential
    neoantigens (both >0.5) and rank top-20-per-length high-confidence candidates
- name: OptiType
  role: Identifies HLA alleles needed as input to DeepHLApan prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:a44e01486265ef3c140d839b8aaf4be98121e62a3a29944c267fc083728d8112
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/jiujiezz/tsnad
  tools:
  - BWA
  - DeepHLApan
  - Docker
  - GATK
  - HISAT2
  - OptiType
  - SAMtools
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  - VEP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# multi-reference-genome-pipeline-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the TSNAD v2.0 one-stop neoantigen prediction pipeline twice, once against GRCh37 and once against GRCh38, using the -V genome-build flag of the Dockerized pipeline to assess how reference genome choice affects neoantigen calls on the TESLA dataset.

## When to use

Use when predicting tumor neoantigens from WES/WGS plus optional RNA-Seq of matched tumor-normal pairs and you need to test or control for reference genome build (GRCh37 vs GRCh38) effects on variant calling and downstream pMHC predictions.

## When NOT to use

- The analysis requires TCR sequencing integration or TCR-pMHC binding prediction — the article states this is future work and not yet part of the TSNAD pipeline
- Input data is not paired WES/WGS (e.g., already-variant-called VCFs only) and you need the full raw-read-to-neoantigen one-stop workflow with its specific depth requirements (minimum 15x WGS / 50x WES)
- A single reference genome is fixed and re-running both builds is unnecessary — TSNAD v1.0 users, who support only GRCh37, cannot benefit from the GRCh38 branch

## Inputs

- Raw WES or WGS FASTQ files for matched tumor-normal samples
- Tumor RNA-Seq FASTQ files (optional, for gene expression TPM filtering and fusion detection)
- Clinical-grade HLA typing data or OptiType-inferred HLA alleles
- Reference genome build selection: GRCh37 or GRCh38

## Outputs

- Per-sample high-confidence neoantigen lists in results/deephlapan_results/ (pMHC pairs with binding and immunogenic scores)
- Per-k (top-10/20/30/40/50) accuracy metrics for GRCh37 and GRCh38 runs written to tesla_benchmark_results.csv
- Comparison against the TESLA baseline positive rate of 37 (6.1%) of 608 immunogenic neoantigens

## How to apply

Pull the all-in-one image with 'docker pull biopharm/tsnad:latest' and launch a container mounting the WES/WGS directory, the RNA-Seq directory, and an output directory. Run 'python TSNAD.py -I samples/ -R RNA-seq/ -V [grch37/grch38] -O results/' once with each build so Trimmomatic trimming, BWA 0.7.17 mapping, SAMtools/GATK 4.2.0.0 processing, Mutect2 SNV/INDEL calling, OptiType 1.3.5 HLA typing, VEP 104 annotation, and DeepHLApan 1.1 binding/immunogenicity prediction all use the same genome build. Retain pMHC pairs with binding score >0.5 and immunogenic score >0.5, drop neoantigens from genes with TPM < 1, and rank candidates per peptide length (8-11 mers) with immunogenic score >0.5 and binding score ranked top 20 to define high-confidence neoantigens. Build top-10/20/30/40/50 candidate lists per patient under each reference genome and score them against TESLA-validated immunogenic pMHC labels, comparing achieved positive rates against the TESLA baseline of 37/608 (6.1%) immunogenic neoantigens. Rationale: TSNAD v1.0 supported only GRCh37, so dual-build runs quantify whether GRCh38 support in v2.0 changes prediction quality.

## Related tools

- **TSNAD v2.0** (One-stop pipeline whose -V flag selects GRCh37 or GRCh38 for all embedded variant-calling and neoantigen-prediction modules) — https://github.com/jiujiezz/tsnad
- **Docker** (Deployment mechanism providing the biopharm/tsnad image preloaded with all tools and both reference genome files) — https://hub.docker.com/r/biopharm/tsnad
- **BWA** (Maps trimmed short reads to the user-selected reference genome (GRCh37 or GRCh38))
- **GATK** (Recalibrates base quality and calls SNVs/INDELs with Mutect2 against the chosen reference build) — https://github.com/broadinstitute/gatk
- **VEP** (Annotates called variants at the amino-acid level using annotations consistent with the selected genome build) — https://github.com/Ensembl/ensembl-vep
- **DeepHLApan** (Predicts peptide-MHC binding and immunogenicity scores used to retain potential neoantigens (both >0.5) and rank top-20-per-length high-confidence candidates)
- **OptiType** (Identifies HLA alleles needed as input to DeepHLApan prediction)

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash -c "cd /home/tsnad && python TSNAD.py -I samples/ -R RNA-seq/ -V grch38 -O results/"
```

## Evaluation signals

- The pipeline completes under both -V grch37 and -V grch38 and writes final neoantigen results to results/deephlapan_results/ in each run
- Predicted neoantigen coordinates/gene annotations are consistent with the selected genome build (no cross-build coordinate mixing between the two runs)
- Retained pMHC pairs satisfy binding score >0.5 and immunogenic score >0.5, and high-confidence neoantigens are immunogenic score >0.5 with binding score ranked top 20 per peptide length
- On the TESLA benchmark, the poorest top-10 selection on GRCh38 achieves 11.1% accuracy, exceeding the 6.1% (37/608) TESLA baseline positive rate
- Expression filter removes neoantigens from genes with TPM < 1 before ranking

## Limitations

- TSNAD v1.0 only supports GRCh37; dual-build comparison requires TSNAD v2.0 or later (README version 2.0.1)
- TCR sequencing analysis and a TCR-pMHC binding prediction model are not yet part of the pipeline and are planned for a future update
- Sequencing depth matters: minimum 15x for WGS and 50x for WES, recommended 30x WGS and 100x WES; lower depth undermines variant calling
- The Docker image is large, so 'docker pull biopharm/tsnad:latest' may take several hours

## Evidence

- [results] Dual-build benchmark outcome on TESLA: [evidence span withheld — anchor status: not_found]
