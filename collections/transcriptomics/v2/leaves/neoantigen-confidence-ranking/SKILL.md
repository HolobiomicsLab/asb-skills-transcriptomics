---
name: neoantigen-confidence-ranking
description: Use when you have candidate peptide-MHC predictions with DeepHLApan binding
  and immunogenicity scores plus gene expression (TPM) values and need to select high-confidence
  neoantigens per sample.
when_to_use_negative:
- No RNA-Seq data are available, so the TPM < 1 expression filter cannot be applied
  to remove neoantigens from unexpressed genes
- Peptide-MHC predictions come from a tool other than DeepHLApan whose scores are
  not on the >0.5 binding/immunogenicity scale, making the thresholds inapplicable
- The goal is TCR-pMHC interaction prediction, which TSNAD v2.0 explicitly does not
  include and defers to future work
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_3534
- http://edamontology.org/topic_2830
tools:
- name: TSNAD v2.0
  role: Integrated pipeline that runs the scoring, expression filtering, and per-length
    ranking to produce high-confidence neoantigens
  repo: https://github.com/jiujiezz/tsnad
- name: DeepHLApan
  role: Predicts peptide-MHC binding and immunogenicity scores used for the >0.5 thresholds
    and top-20 ranking
  repo: https://github.com/jiujiezz/tsnad
- name: OptiType
  role: Identifies patient HLA alleles against which peptides are scored
- name: StringTie
  role: Quantifies gene expression (TPM) used for the expression filter
- name: VEP
  role: Annotates SNVs/INDELs at the amino-acid level to extract mutant peptides of
    length 8 to 11
  repo: https://github.com/Ensembl/ensembl-vep
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:a475d371cf32c98ff1ee7c6a4c79085e7715003da11a446c669b05943b8f40f7
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
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_3534
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# neoantigen-confidence-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank and filter predicted peptide-MHC pairs into high-confidence neoantigens using DeepHLApan binding and immunogenicity scores combined with gene-expression evidence, as implemented in the TSNAD v2.0 pipeline.

## When to use

Use when you have candidate peptide-MHC predictions with DeepHLApan binding and immunogenicity scores plus gene expression (TPM) values and need to select high-confidence neoantigens per sample.

## When NOT to use

- No RNA-Seq data are available, so the TPM < 1 expression filter cannot be applied to remove neoantigens from unexpressed genes
- Peptide-MHC predictions come from a tool other than DeepHLApan whose scores are not on the >0.5 binding/immunogenicity scale, making the thresholds inapplicable
- The goal is TCR-pMHC interaction prediction, which TSNAD v2.0 explicitly does not include and defers to future work

## Inputs

- DeepHLApan output table of peptide-MHC binding scores and immunogenicity scores for 8-11 mer mutant peptides
- Gene expression quantification (TPM) from RNA-Seq via HISAT2/StringTie
- OptiType-detected patient HLA alleles
- VEP-annotated SNV/INDEL-derived and Arriba-detected fusion-derived mutant peptides

## Outputs

- List of potential neoantigens (binding score >0.5 and immunogenic score >0.5)
- Ranked list of high-confidence neoantigens (immunogenic score >0.5, binding score top 20 per peptide length)
- Final neoantigen results in results/deephlapan_results/ when run through TSNAD

## How to apply

After DeepHLApan predicts peptide-MHC binding and immunogenicity for mutant and fusion-derived peptides (lengths 8-11) against OptiType-detected HLA alleles, retain all pairs with binding score >0.5 and immunogenic score >0.5 as potential neoantigens. Apply the expression filter to remove neoantigens from unexpressed genes (TPM < 1), using gene expression quantified in the RNA-Seq module with HISAT2, SAMtools, and StringTie. For high-confidence selection, keep peptide-MHC pairs with immunogenic score >0.5 whose binding score ranks in the top 20 across the different peptide lengths (8, 9, 10, 11-mers), ranking binding scores per peptide length. Applying this to each of the five TESLA samples yields 80 high-confidence neoantigens per sample, 400 in total, which serves as the validation benchmark.

## Related tools

- **TSNAD v2.0** (Integrated pipeline that runs the scoring, expression filtering, and per-length ranking to produce high-confidence neoantigens) — https://github.com/jiujiezz/tsnad
- **DeepHLApan** (Predicts peptide-MHC binding and immunogenicity scores used for the >0.5 thresholds and top-20 ranking) — https://github.com/jiujiezz/tsnad
- **OptiType** (Identifies patient HLA alleles against which peptides are scored)
- **StringTie** (Quantifies gene expression (TPM) used for the expression filter)
- **VEP** (Annotates SNVs/INDELs at the amino-acid level to extract mutant peptides of length 8 to 11) — https://github.com/Ensembl/ensembl-vep

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [dir of RNA-seq]:/home/tsnad/RNA-seq -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash; cd /home/tsnad; python TSNAD.py -I samples/ -R RNA-seq/ -V grch37 -O results/
```

## Evaluation signals

- Running the five TESLA samples (Synapse syn21048999) end-to-end reproduces 80 high-confidence neoantigens per sample and 400 in total, matching the paper's reported output
- Every retained potential neoantigen has binding score >0.5 AND immunogenic score >0.5
- Every high-confidence neoantigen has immunogenic score >0.5 and a binding score ranked within the top 20 for its peptide length (8-11)
- No neoantigens from genes with TPM < 1 remain in the final list
- Final results are present in results/deephlapan_results/ after the TSNAD.py run

## Limitations

- TSNAD v2.0 lacks a TCR-pMHC binding prediction model; understanding pMHC-induced T-cell response is left for future work
- TCR sequencing analysis is not part of the pipeline and is planned for a future update
- Minimum sequencing depth of 15x for WGS and 50x for WES (recommended 30x and 100x) is needed for reliable input data

## Evidence
