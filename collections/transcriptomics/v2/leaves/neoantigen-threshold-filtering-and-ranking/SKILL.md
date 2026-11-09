---
name: neoantigen-threshold-filtering-and-ranking
description: Use when candidate mutant peptides (8–11 aa) already have DeepHLApan
  binding and immunogenicity scores and need to be filtered into potential vs high-confidence
  neoantigens.
when_to_use_negative:
- Input is raw FASTQ/WES data with no called mutations — the pipeline must first run
  Mutect2 SNV/INDEL calling, VEP annotation, and OptiType HLA typing before filtering
  applies
- You need TCR-pMHC interaction prediction or TCR sequencing analysis — the article
  explicitly lists these as future work, not part of TSNAD v2.0
- Scores come from NetMHCpan or other binding-only predictors — the >0.5 immunogenic
  threshold is defined for DeepHLApan's joint binding/immunogenicity scoring
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DeepHLApan
  role: Scores candidate pMHC pairs for both binding and immunogenicity; its scores
    are the inputs to the >0.5 thresholds and top-20 rank rule
  repo: https://github.com/jiujiezz/tsnad
- name: TSNAD v2.0
  role: One-stop pipeline that extracts 8–11 aa mutant peptides, runs DeepHLApan,
    and applies the threshold/rank filters to produce the final neoantigen table
  repo: https://github.com/jiujiezz/tsnad
- name: VEP
  role: Annotates Mutect2 SNV/INDEL calls at the amino-acid level so mutant peptides
    can be extracted for scoring
  repo: https://github.com/Ensembl/ensembl-vep
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:af6449162c8629a2693df7fcc8968f2e3da58d3ffb0350316ad4bfdc763f98fc
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
  - TSNAD v2.0
  - VEP
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Neoantigen threshold filtering and ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters DeepHLApan peptide-MHC (pMHC) predictions into potential and high-confidence neoantigens using binding and immunogenicity score thresholds (>0.5 each) plus a top-20 binding-rank rule per peptide length. Replaces NetMHCpan-style binding-only scoring to cut false positives.

## When to use

Use when candidate mutant peptides (8–11 aa) already have DeepHLApan binding and immunogenicity scores and need to be filtered into potential vs high-confidence neoantigens.

## When NOT to use

- Input is raw FASTQ/WES data with no called mutations — the pipeline must first run Mutect2 SNV/INDEL calling, VEP annotation, and OptiType HLA typing before filtering applies
- You need TCR-pMHC interaction prediction or TCR sequencing analysis — the article explicitly lists these as future work, not part of TSNAD v2.0
- Scores come from NetMHCpan or other binding-only predictors — the >0.5 immunogenic threshold is defined for DeepHLApan's joint binding/immunogenicity scoring

## Inputs

- DeepHLApan output with binding and immunogenicity scores for mutant peptide-HLA pairs
- Mutant peptide sequences of 8–11 aa covering each SNV/INDEL mutation
- Patient HLA alleles identified by OptiType 1.3.5
- VEP 104 amino-acid mutation annotations (from GATK Mutect2 SNV/INDEL calls)
- Optional: gene expression (TPM) values from the RNA-Seq module for expression filtering

## Outputs

- Table of potential neoantigens (pMHC pairs with binding score > 0.5 and immunogenic score > 0.5) with binding and immunogenicity scores
- High-confidence neoantigen flags (immunogenic score > 0.5 and binding score ranked top 20 per peptide-length class, up to 80 per sample)
- neoantigen_predictions.tsv written by TSNAD (final neoantigen results under results/deephlapan_results/ in the Docker distribution)

## How to apply

Run DeepHLApan 1.1 on all mutant peptides (lengths 8–11 aa, extracted with in-home scripts after VEP 104 amino-acid annotation) combined with the patient's OptiType-identified HLA alleles. Retain as potential neoantigens all pMHC pairs with binding score > 0.5 AND immunogenic score > 0.5; the dual criterion matters because DeepHLApan evaluates both peptide-MHC binding and potential immunogenicity, which significantly reduced false positives versus NetMHCpan. Among pairs passing the immunogenic score > 0.5 threshold, flag as high-confidence those whose binding score ranks in the top 20 within each peptide-length class, yielding up to 80 high-confidence neoantigens per sample (4 length classes × 20). Optionally apply an upstream expression filter removing neoantigens from genes with TPM < 1 detected in the RNA-Seq module. Write the ranked results, including binding and immunogenicity scores and high-confidence flags, to the neoantigen output table.

## Related tools

- **DeepHLApan** (Scores candidate pMHC pairs for both binding and immunogenicity; its scores are the inputs to the >0.5 thresholds and top-20 rank rule) — https://github.com/jiujiezz/tsnad
- **TSNAD v2.0** (One-stop pipeline that extracts 8–11 aa mutant peptides, runs DeepHLApan, and applies the threshold/rank filters to produce the final neoantigen table) — https://github.com/jiujiezz/tsnad
- **VEP** (Annotates Mutect2 SNV/INDEL calls at the amino-acid level so mutant peptides can be extracted for scoring) — https://github.com/Ensembl/ensembl-vep

## Examples

```
python TSNAD.py -I samples/ -R RNA-seq/ -V grch37 -O results/
```

## Evaluation signals

- Every retained potential neoantigen has binding score > 0.5 AND immunogenic score > 0.5; pairs failing either threshold are excluded
- High-confidence set has immunogenic score > 0.5 and binding score within the top 20 of its peptide-length class, capped at ~80 per sample (20 per length class for 8–11 aa)
- All scored peptides have lengths in the 8–11 aa range
- Output table contains binding score, immunogenicity score, and high-confidence flag columns for each pMHC pair
- Results are reproducible via the Docker run producing final files under results/deephlapan_results/

## Limitations

- The binding top-20 rank rule caps high-confidence calls per length class and is relative within each sample, not an absolute-quality guarantee
- No TCR-pMHC binding prediction model is included; the authors note more effort on TCR-pMHC interaction is needed to predict high-confidence neoantigens
- TPM < 1 expression filtering only applies when RNA-Seq data are provided; without module 4, unexpressed-gene neoantigens are not removed

## Evidence
