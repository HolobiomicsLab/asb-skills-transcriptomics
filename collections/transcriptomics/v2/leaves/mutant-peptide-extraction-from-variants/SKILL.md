---
name: mutant-peptide-extraction-from-variants
description: Use when VEP-annotated somatic SNV/INDEL calls (e.g. from GATK Mutect2)
  must be converted into amino-acid-level mutant peptide sequences for downstream
  neoantigen prediction.
when_to_use_negative:
- Mutations come from gene fusions or RNA-Seq-only analyses — TSNAD v2.0 handles fusion-derived
  neoantigens via STAR/Arriba in a separate module, not via this SNV/INDEL peptide
  extraction.
- Genes carrying the mutations are not expressed (TPM < 1) — the expression filter
  from module 4 removes neoantigens from unexpressed genes before prediction is useful.
- A standalone MHC-I binding predictor is needed without immunogenicity scoring —
  DeepHLApan replaced NetMHCpan specifically to add immunogenicity; NetMHCpan is no
  longer used in this pipeline.
edam_operation: http://edamontology.org/operation_0480
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_2830
tools:
- name: TSNAD v2.0
  role: Host pipeline whose in-home scripts perform the 8–11 aa mutant peptide extraction
    from VEP-annotated variants
  repo: https://github.com/jiujiezz/tsnad
- name: VEP
  role: Annotates Mutect2 SNV/INDEL calls to obtain mutation information at the amino
    acid level prior to peptide extraction
  repo: https://github.com/Ensembl/ensembl-vep
- name: DeepHLApan
  role: Scores the extracted mutant peptides with HLA alleles for peptide-MHC binding
    and immunogenicity (replaced NetMHCpan)
- name: GATK
  role: Mutect2 module produces the SNV/INDEL calls that are the source variants for
    peptide extraction
  repo: https://github.com/broadinstitute/gatk
- name: OptiType
  role: Identifies patient HLA alleles (v1.3.5) paired with extracted peptides for
    pMHC prediction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:70c47747a5210cf3d2131c00bd8ee54725c1e78b8a18f66c70355b3a91f2c835
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
  - DeepHLApan
  - GATK
  - OptiType
  - TSNAD v2.0
  - VEP
  edam_operation: http://edamontology.org/operation_0480
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# Mutant peptide extraction from variants

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts VEP-annotated SNV/INDEL calls into a list of mutant peptides spanning each mutation, with peptide lengths from 8 to 11 amino acids, to feed pMHC binding/immunogenicity prediction (DeepHLApan in TSNAD v2.0).

## When to use

Use when VEP-annotated somatic SNV/INDEL calls (e.g. from GATK Mutect2) must be converted into amino-acid-level mutant peptide sequences for downstream neoantigen prediction.

## When NOT to use

- Mutations come from gene fusions or RNA-Seq-only analyses — TSNAD v2.0 handles fusion-derived neoantigens via STAR/Arriba in a separate module, not via this SNV/INDEL peptide extraction.
- Genes carrying the mutations are not expressed (TPM < 1) — the expression filter from module 4 removes neoantigens from unexpressed genes before prediction is useful.
- A standalone MHC-I binding predictor is needed without immunogenicity scoring — DeepHLApan replaced NetMHCpan specifically to add immunogenicity; NetMHCpan is no longer used in this pipeline.

## Inputs

- VCF of somatic SNV/INDEL calls from GATK Mutect2
- VEP 104 annotation output with amino-acid-level mutation information
- Patient HLA typing from OptiType 1.3.5 (combined with extracted peptides at scoring stage)

## Outputs

- Mutant peptide sequences covering each mutation with lengths 8–11 amino acids
- neoantigen_predictions.tsv with ranked pMHC pairs including binding and immunogenicity scores and high-confidence flags
- High-confidence neoantigen set (up to 80 per sample: immunogenic score >0.5 and binding score rank top 20 per peptide length)

## How to apply

First run VEP 104 on the Mutect2 SNV/INDEL VCF to obtain mutation information at the amino acid level. Then, using in-home scripts, extract all mutant peptides that cover each mutation with lengths in the range 8 to 11 amino acids — a window that spans the canonical MHC-I presented lengths. Combine these peptides with the patient HLA alleles identified by OptiType 1.3.5 and score them with DeepHLApan 1.1 for peptide-MHC binding and immunogenicity. Retain pairs with binding score >0.5 and immunogenic score >0.5 as potential neoantigens, and flag as high-confidence those with immunogenic score >0.5 whose binding score ranks in the top 20 within each peptide-length class. Write the ranked pMHC results, including binding and immunogenicity scores and high-confidence flags, to neoantigen_predictions.tsv.

## Related tools

- **TSNAD v2.0** (Host pipeline whose in-home scripts perform the 8–11 aa mutant peptide extraction from VEP-annotated variants) — https://github.com/jiujiezz/tsnad
- **VEP** (Annotates Mutect2 SNV/INDEL calls to obtain mutation information at the amino acid level prior to peptide extraction) — https://github.com/Ensembl/ensembl-vep
- **DeepHLApan** (Scores the extracted mutant peptides with HLA alleles for peptide-MHC binding and immunogenicity (replaced NetMHCpan))
- **GATK** (Mutect2 module produces the SNV/INDEL calls that are the source variants for peptide extraction) — https://github.com/broadinstitute/gatk
- **OptiType** (Identifies patient HLA alleles (v1.3.5) paired with extracted peptides for pMHC prediction)

## Examples

```
python TSNAD.py -I samples/ -R RNA-seq/ -V grch37 -O results/  # final neoantigen results in results/deephlapan_results/
```

## Evaluation signals

- Every extracted peptide has length between 8 and 11 amino acids inclusive.
- Every VEP-annotated coding mutation is covered by at least one extracted peptide spanning the mutation site.
- All retained potential neoantigens have binding score >0.5 and immunogenic score >0.5.
- High-confidence set equals pairs with immunogenic score >0.5 and binding score rank top 20 across peptide lengths, capped at up to 80 per sample.
- Output neoantigen_predictions.tsv contains ranked pMHC rows with binding and immunogenicity scores and high-confidence flags.

## Limitations

- Binding/immunogenicity thresholds (0.5) and top-20-per-length ranking are empirical; TESLA benchmark showed only 37 of 608 predicted neoantigens (6.1%) were immunogenic, so false positives remain.
- No TCR-pMHC binding prediction model is included; the authors state more effort should be put into TCR-pMHC interaction for high-confidence neoantigen prediction.
- TSNAD v1.0 only supported GRCh37; v2.0 adds GRCh38, so reference genome must match the annotation resources (e.g. dbSNP b37 bundle for GRCh37 workflows).
- Expression filter (TPM < 1) requires RNA-Seq/gene expression data from module 4; without it, neoantigens from unexpressed genes are not removed.

## Evidence

- [methods] Take the SNV/INDEL calls produced by GATK Mutect2 in Module 1 and annotate them with VEP 104 to obtain mutation information at the amino acid level.: [evidence span withheld — anchor status: not_found]
