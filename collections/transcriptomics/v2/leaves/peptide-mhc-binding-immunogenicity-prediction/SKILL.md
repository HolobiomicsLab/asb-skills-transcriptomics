---
name: peptide-mhc-binding-immunogenicity-prediction
description: Use when mutant peptides of length 8–11 aa derived from called SNVs,
  INDELs, or gene fusions must be scored for peptide-MHC binding and immunogenicity
  against patient HLA alleles to shortlist candidate neoantigens.
when_to_use_negative:
- The task is TCR-pMHC binding prediction or TCR sequencing analysis — the article
  explicitly states this is not yet part of the pipeline and is planned for future
  updates.
- Peptides are shorter than 8 or longer than 11 amino acids — only peptides of length
  8–11 covering the mutations are extracted and predicted.
- The user wants generic MHC binding affinity without immunogenicity — TSNAD v2.0
  deliberately replaced NetMHCpan with DeepHLApan, which considers both binding and
  immunogenicity.
tools:
- name: DeepHLApan
  role: Predicts peptide-MHC binding and immunogenicity of the extracted mutant peptides
    against OptiType-detected HLA alleles
- name: TSNAD v2.0
  role: One-stop pipeline orchestrating peptide extraction, DeepHLApan scoring, expression
    filtering, and top-20-per-length ranking
  repo: https://github.com/jiujiezz/tsnad
- name: OptiType
  role: Identifies patient HLA alleles from cleaned fastq files, which define the
    MHC alleles used in prediction
- name: VEP
  role: Annotates called SNVs/INDELs at the amino-acid level to generate the mutant
    peptides fed to DeepHLApan
  repo: https://github.com/Ensembl/ensembl-vep
- name: Docker
  role: Provides the pre-configured biopharm/tsnad image bundling DeepHLApan and all
    pipeline dependencies
  repo: https://hub.docker.com/r/biopharm/tsnad
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:ea2430d1d0e13df347540ab5423693dbec5c4c058d17d854a2197f7888f903c6
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
  - Arriba
  - BWA
  - DeepHLApan
  - Docker
  - GATK
  - HISAT2
  - OptiType
  - SAMtools
  - STAR
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  - VEP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# peptide-mhc-binding-immunogenicity-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts peptide-MHC binding and immunogenicity of candidate mutant peptides using DeepHLApan v1.1, which jointly scores binding and immunogenicity, and then applies score and rank thresholds to designate potential and high-confidence neoantigens. This is the neoantigen-selection core of the TSNAD v2.0 pipeline, replacing NetMHCpan.

## When to use

Use when mutant peptides of length 8–11 aa derived from called SNVs, INDELs, or gene fusions must be scored for peptide-MHC binding and immunogenicity against patient HLA alleles to shortlist candidate neoantigens.

## When NOT to use

- The task is TCR-pMHC binding prediction or TCR sequencing analysis — the article explicitly states this is not yet part of the pipeline and is planned for future updates.
- Peptides are shorter than 8 or longer than 11 amino acids — only peptides of length 8–11 covering the mutations are extracted and predicted.
- The user wants generic MHC binding affinity without immunogenicity — TSNAD v2.0 deliberately replaced NetMHCpan with DeepHLApan, which considers both binding and immunogenicity.

## Inputs

- Mutant peptides of length 8–11 amino acids covering SNV/INDEL-annotated mutations
- Fusion-derived peptide sequences from RNA-Seq (STAR + Arriba)
- Patient HLA alleles identified by OptiType v1.3.5 from cleaned fastq files
- Gene expression quantification (TPM) from HISAT2/StringTie for the TPM < 1 expression filter

## Outputs

- Peptide-MHC binding and immunogenicity scores from DeepHLApan
- List of potential neoantigens (binding score >0.5 and immunogenic score >0.5)
- List of high-confidence neoantigens (immunogenic score >0.5, binding score ranked top 20 per peptide length)
- results/deephlapan_results/ folder with final neoantigen predictions (Docker run)

## How to apply

Annotate called SNVs/INDELs at the amino-acid level with VEP 104 and extract all mutant peptides covering the mutations with lengths 8 to 11 amino acids; include fusion-derived peptides detected by STAR 2.7 plus Arriba 1.1.0. Run DeepHLApan 1.1 on these peptides against the patient HLA alleles identified by OptiType 1.3.5. Retain peptide-MHC pairs with binding score >0.5 AND immunogenic score >0.5 as potential neoantigens, and remove neoantigens from genes with TPM < 1 using the gene-expression module (HISAT2 2.2.1, SAMtools, StringTie 2.1.6). Rank the binding scores per peptide length and designate pairs with immunogenic score >0.5 and binding score ranked top 20 across peptide lengths as high-confidence neoantigens — in the TESLA benchmark this top-20-per-length rule yields 80 high-confidence neoantigens per sample (400 across five samples).

## Related tools

- **DeepHLApan** (Predicts peptide-MHC binding and immunogenicity of the extracted mutant peptides against OptiType-detected HLA alleles)
- **TSNAD v2.0** (One-stop pipeline orchestrating peptide extraction, DeepHLApan scoring, expression filtering, and top-20-per-length ranking) — https://github.com/jiujiezz/tsnad
- **OptiType** (Identifies patient HLA alleles from cleaned fastq files, which define the MHC alleles used in prediction)
- **VEP** (Annotates called SNVs/INDELs at the amino-acid level to generate the mutant peptides fed to DeepHLApan) — https://github.com/Ensembl/ensembl-vep
- **Docker** (Provides the pre-configured biopharm/tsnad image bundling DeepHLApan and all pipeline dependencies) — https://hub.docker.com/r/biopharm/tsnad

## Examples

```
python TSNAD.py -I samples/ -R RNA-seq/ -V grch37 -O results/
```

## Evaluation signals

- Every retained potential neoantigen has binding score >0.5 and immunogenic score >0.5.
- Every high-confidence neoantigen has immunogenic score >0.5 and its binding score ranked top 20 across the different lengths of peptides.
- Running TSNAD v2.0 end-to-end on each of the five TESLA samples yields exactly 80 high-confidence neoantigens (400 total across five samples).
- All neoantigens in the final list come from genes with TPM ≥ 1 (unexpressed-gene neoantigens removed).
- All scored peptides have lengths between 8 and 11 amino acids, and final neoantigen predictions appear in results/deephlapan_results/.

## Limitations

- The pipeline has no TCR-pMHC binding prediction model and does not include TCR sequencing analysis; the article notes more effort on TCR-pMHC interaction is needed and this is planned for a future update.
- Performance depends on sequencing depth: minimum 15x for WGS and 50x for WES, with 30x WGS and 100x WES recommended.
- Even top-ranked TSNAD v2.0 selections only reach ~11.1% accuracy at top-10 against the TESLA baseline in which only 37 of 608 (6.1%) tested neoantigens were immunogenic, so predictions remain a shortlist, not validated immunogens.

## Evidence
