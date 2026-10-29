---
name: somatic-variant-effect-annotation
description: Use when you have somatic SNV/INDEL calls (e.g., GATK Mutect2 VCFs from
  tumor-normal WES/WGS) and HLA typing, and need amino-acid-level variant annotation
  plus peptide-MHC binding/immunogenicity scoring for neoantigen prediction.
when_to_use_negative:
- Input is raw reads without prior somatic mutation calling — Mutect2 SNV/INDEL calling
  must run first in Module 1
- No HLA typing is available — DeepHLApan scoring requires HLA alleles from OptiType
- The goal is TCR-pMHC binding prediction — the article explicitly notes no TCR-pMHC
  interaction model is included in TSNAD v2.0
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TSNAD v2.0
  role: One-stop pipeline that orchestrates VEP annotation, 8–11-mer peptide extraction,
    and DeepHLApan neoantigen scoring
  repo: https://github.com/jiujiezz/tsnad
- name: VEP 104
  role: Annotates Mutect2 SNV/INDEL calls to obtain mutation information at the amino
    acid level
  repo: https://github.com/Ensembl/ensembl-vep
- name: DeepHLApan 1.1
  role: Scores peptide-MHC binding and immunogenicity of presented pMHC pairs, replacing
    NetMHCpan
- name: GATK 4.2.0.0
  role: Mutect2 module produces the SNV/INDEL calls that feed the annotation step
  repo: https://github.com/broadinstitute/gatk
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:d9de416e2f7833bd6fd3719894bae7d89187dda3c1d35562e17667fbfedad780
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
  - DeepHLApan 1.1
  - GATK 4.2.0.0
  - TSNAD v2.0
  - VEP
  - VEP 104
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# somatic-variant-effect-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates tumor somatic SNV/INDEL calls with Variant Effect Predictor (VEP 104) to obtain amino-acid-level mutation information, then extracts 8–11-mer mutant peptides and scores peptide-MHC pairs with DeepHLApan 1.1 for neoantigen prediction as implemented in TSNAD v2.0.

## When to use

Use when you have somatic SNV/INDEL calls (e.g., GATK Mutect2 VCFs from tumor-normal WES/WGS) and HLA typing, and need amino-acid-level variant annotation plus peptide-MHC binding/immunogenicity scoring for neoantigen prediction.

## When NOT to use

- Input is raw reads without prior somatic mutation calling — Mutect2 SNV/INDEL calling must run first in Module 1
- No HLA typing is available — DeepHLApan scoring requires HLA alleles from OptiType
- The goal is TCR-pMHC binding prediction — the article explicitly notes no TCR-pMHC interaction model is included in TSNAD v2.0

## Inputs

- SNV/INDEL calls from GATK Mutect2 (VCF) for tumor-normal WES/WGS pairs
- HLA alleles typed by OptiType 1.3.5
- Reference genome GRCh37 or GRCh38 (with dbSNP dbsnp_138.b37.vcf resource)
- Optional tumor RNA-Seq data for gene expression (TPM) filtering

## Outputs

- Amino-acid-level mutation annotations from VEP 104
- Extracted mutant peptides of length 8 to 11 amino acids covering each mutation
- Ranked pMHC neoantigen results (neoantigen_predictions.tsv) with DeepHLApan binding and immunogenicity scores and high-confidence flags

## How to apply

Take SNV/INDEL calls produced by the Mutect2 module of GATK 4.2.0.0 and annotate them with VEP 104 to obtain mutation information at the amino acid level. Using TSNAD's in-home scripts, extract all mutant peptides covering each mutation with lengths in the range 8 to 11 amino acids. Combine these peptides with HLA alleles identified by OptiType 1.3.5 and run DeepHLApan 1.1, which replaces NetMHCpan and scores both peptide-MHC binding and the potential immunogenicity of the presented pMHC pairs. Retain peptide-MHC pairs with binding score >0.5 and immunogenic score >0.5 as potential neoantigens, and flag high-confidence neoantigens as those with immunogenic score >0.5 whose binding score ranks top 20 within each peptide-length class (up to 80 high-confidence neoantigens per sample). Optionally apply an expression filter removing neoantigens from genes with TPM < 1 using RNA-Seq-derived expression. The rationale is that considering immunogenicity alongside binding significantly reduces false positives.

## Related tools

- **TSNAD v2.0** (One-stop pipeline that orchestrates VEP annotation, 8–11-mer peptide extraction, and DeepHLApan neoantigen scoring) — https://github.com/jiujiezz/tsnad
- **VEP 104** (Annotates Mutect2 SNV/INDEL calls to obtain mutation information at the amino acid level) — https://github.com/Ensembl/ensembl-vep
- **DeepHLApan 1.1** (Scores peptide-MHC binding and immunogenicity of presented pMHC pairs, replacing NetMHCpan)
- **GATK 4.2.0.0** (Mutect2 module produces the SNV/INDEL calls that feed the annotation step) — https://github.com/broadinstitute/gatk

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash; cd /home/tsnad; python TSNAD.py -I samples/ -V grch38 -O results/
```

## Evaluation signals

- Only peptide-MHC pairs with binding score >0.5 and immunogenic score >0.5 appear in the potential neoantigen output
- High-confidence flags are restricted to pairs with immunogenic score >0.5 and binding score ranked top 20 per peptide-length class (maximum 80 per sample)
- All extracted mutant peptides have lengths within the 8–11 amino acid range and cover each called mutation
- Final DeepHLApan neoantigen results are written to results/deephlapan_results/ in the TSNAD output directory
- Output supports both GRCh37 and GRCh38 reference genome selections

## Limitations

- TSNAD v1.0 only supported GRCh37; GRCh38 support was added in v2.0, so verify genome version selection matches input data
- No TCR sequencing analysis or TCR-pMHC binding prediction model is included; planned for future updates
- Gene expression filter (TPM < 1) requires RNA-Seq data; without it, neoantigens from unexpressed genes are not removed

## Evidence

- [readme] VEP (Variant Effect Predictor) predicts the functional effects of genomic variants.: [evidence span withheld — anchor status: not_found]
