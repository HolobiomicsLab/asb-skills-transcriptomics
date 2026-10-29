---
name: hla-typed-peptide-prediction
description: Use when tumor SNV/INDEL calls (e.g., from GATK Mutect2) and patient
  WES/RNA-Seq data are available and HLA-typed peptide-MHC immunogenicity prediction
  is needed.
when_to_use_negative:
- Input lacks patient HLA typing — OptiType alleles are required to pair peptides
  with MHC
- Goal is TCR-pMHC interaction prediction — the article notes TCR-pMHC binding prediction
  is a planned future feature, not included
- Variants lie in unexpressed genes — the expression filter removes neoantigens from
  genes with TPM < 1, so RNA-Seq expression data should be available
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_0080
tools:
- name: DeepHLApan
  role: Scores peptide-MHC binding and immunogenicity of presented pMHC pairs, replacing
    NetMHCpan
- name: VEP 104
  role: Annotates Mutect2 SNV/INDEL calls to obtain amino-acid-level mutation information
  repo: https://github.com/Ensembl/ensembl-vep
- name: OptiType 1.3.5
  role: Identifies patient HLA alleles used as prediction targets
- name: TSNAD v2.0
  role: One-stop pipeline orchestrating annotation, peptide extraction, and DeepHLApan
    scoring
  repo: https://github.com/jiujiezz/tsnad
- name: GATK 4.2.0.0
  role: Mutect2 module calls SNVs/INDELs that feed the neoantigen prediction step
  repo: https://github.com/broadinstitute/gatk
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:6f0b2c137efa3d3645f3cf6951ceda898ca7b90b2e724a930b45be08861cb81d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/Ensembl/ensembl-vep
  tools:
  - DeepHLApan
  - GATK 4.2.0.0
  - OptiType 1.3.5
  - TSNAD v2.0
  - VEP
  - VEP 104
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# hla-typed-peptide-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict candidate neoantigenic peptide-MHC pairs by combining patient HLA alleles (OptiType) with mutant peptides extracted after VEP annotation, scored by DeepHLApan for both binding and immunogenicity within TSNAD v2.0.

## When to use

Use when tumor SNV/INDEL calls (e.g., from GATK Mutect2) and patient WES/RNA-Seq data are available and HLA-typed peptide-MHC immunogenicity prediction is needed.

## When NOT to use

- Input lacks patient HLA typing — OptiType alleles are required to pair peptides with MHC
- Goal is TCR-pMHC interaction prediction — the article notes TCR-pMHC binding prediction is a planned future feature, not included
- Variants lie in unexpressed genes — the expression filter removes neoantigens from genes with TPM < 1, so RNA-Seq expression data should be available

## Inputs

- SNV/INDEL variant calls from GATK Mutect2 (VCF)
- VEP 104 amino-acid-level mutation annotations
- HLA alleles typed by OptiType 1.3.5 from WES/tumor RNA-Seq
- Reference genome build (GRCh37 or GRCh38)

## Outputs

- neoantigen_predictions.tsv with ranked pMHC pairs including binding and immunogenicity scores and high-confidence flags
- Potential neoantigen list (binding score > 0.5 and immunogenic score > 0.5)
- High-confidence neoantigen set (immunogenic score > 0.5, binding rank top 20 per length class)

## How to apply

Annotate Mutect2 SNV/INDEL calls with VEP 104 to obtain amino-acid-level mutation information, then use in-home scripts to extract all mutant peptides covering each mutation with lengths 8 to 11 amino acids. Combine these peptides with the HLA alleles identified by OptiType 1.3.5 and run DeepHLApan 1.1, which replaced NetMHCpan in TSNAD v2.0 because it scores both peptide-MHC binding and potential immunogenicity of presented pMHC pairs, significantly reducing false positives. Retain pMHC pairs with binding score > 0.5 and immunogenic score > 0.5 as potential neoantigens, and flag pairs with immunogenic score > 0.5 whose binding score ranks in the top 20 within each peptide-length class as high-confidence neoantigens (up to 80 per sample across the four length classes). Write ranked results with binding and immunogenicity scores and high-confidence flags to neoantigen_predictions.tsv.

## Related tools

- **DeepHLApan** (Scores peptide-MHC binding and immunogenicity of presented pMHC pairs, replacing NetMHCpan)
- **VEP 104** (Annotates Mutect2 SNV/INDEL calls to obtain amino-acid-level mutation information) — https://github.com/Ensembl/ensembl-vep
- **OptiType 1.3.5** (Identifies patient HLA alleles used as prediction targets)
- **TSNAD v2.0** (One-stop pipeline orchestrating annotation, peptide extraction, and DeepHLApan scoring) — https://github.com/jiujiezz/tsnad
- **GATK 4.2.0.0** (Mutect2 module calls SNVs/INDELs that feed the neoantigen prediction step) — https://github.com/broadinstitute/gatk

## Examples

```
cd /home/tsnad; bash uncompress.sh; python TSNAD.py -I samples/ -R RNA-seq/ -V grch38 -O results/
```

## Evaluation signals

- All retained pMHC pairs have binding score > 0.5 and immunogenic score > 0.5
- High-confidence set is bounded: up to 20 binding-ranked pairs per peptide length class (8-11 aa), i.e., up to 80 per sample
- Output file neoantigen_predictions.tsv exists in results/deephlapan_results/ and contains binding and immunogenicity score columns
- Mutant peptides span lengths 8 to 11 amino acids covering every VEP-annotated mutation
- Predictions are anchored to the OptiType-typed HLA alleles for the sample

## Limitations

- No TCR-pMHC binding prediction model included; the discussion notes more effort should be put into T-cell receptor-pMHC interaction to predict high-confidence neoantigens
- TCR sequencing analysis is not yet part of the pipeline (planned for a future update)
- TSNAD v1.0 only supports GRCh37; v2.0 adds GRCh38 selection, so genome build must match annotation resources
- Gene expression filter removes neoantigens from unexpressed genes (TPM < 1), requiring RNA-Seq data for that filtering

## Evidence
