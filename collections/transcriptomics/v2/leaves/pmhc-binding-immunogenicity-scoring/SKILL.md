---
name: pmhc-binding-immunogenicity-scoring
description: Use when 8–11-aa mutant peptides extracted from annotated tumor SNV/INDEL
  calls must be paired with OptiType-derived HLA alleles and ranked by pMHC binding
  plus immunogenicity to prioritize neoantigens.
when_to_use_negative:
- No patient HLA typing is available (DeepHLApan in TSNAD requires OptiType-derived
  HLA alleles as input)
- The task requires TCR-pMHC interaction prediction — the article states no TCR-pMHC
  binding prediction model is included and this is future work
- Peptides outside the 8–11 amino acid range or non-MHC-I binding scoring is needed,
  since the extraction step covers only lengths 8 to 11
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3306
tools:
- name: DeepHLApan 1.1
  role: Scores each candidate peptide-HLA pair for both peptide-MHC binding and immunogenicity,
    replacing NetMHCpan in the TSNAD v2.0 prediction module
- name: VEP 104
  role: Annotates Mutect2 SNV/INDEL calls to provide the amino-acid-level mutation
    information from which mutant peptides are extracted
  repo: https://github.com/Ensembl/ensembl-vep
- name: OptiType 1.3.5
  role: Supplies the patient HLA alleles that are combined with the extracted mutant
    peptides as DeepHLApan input
- name: TSNAD v2.0
  role: One-stop pipeline that orchestrates peptide extraction, DeepHLApan scoring,
    and threshold-based neoantigen filtering
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:2b4a0ea474a2bc232a9b6c38761d7deb9c26225ef0ca5237c9babf34e57e942b
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
  - DeepHLApan 1.1
  - OptiType 1.3.5
  - TSNAD v2.0
  - VEP
  - VEP 104
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# pmhc-binding-immunogenicity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores candidate peptide-MHC (pMHC) pairs for both binding affinity and immunogenicity using DeepHLApan 1.1 inside TSNAD v2.0, replacing the binding-only NetMHCpan step; this dual scoring significantly reduced false positives in neoantigen prediction.

## When to use

Use when 8–11-aa mutant peptides extracted from annotated tumor SNV/INDEL calls must be paired with OptiType-derived HLA alleles and ranked by pMHC binding plus immunogenicity to prioritize neoantigens.

## When NOT to use

- No patient HLA typing is available (DeepHLApan in TSNAD requires OptiType-derived HLA alleles as input)
- The task requires TCR-pMHC interaction prediction — the article states no TCR-pMHC binding prediction model is included and this is future work
- Peptides outside the 8–11 amino acid range or non-MHC-I binding scoring is needed, since the extraction step covers only lengths 8 to 11

## Inputs

- SNV/INDEL calls from GATK Mutect2 (tumor-normal WES/WGS VCF)
- VEP 104 annotations providing amino-acid-level mutation information
- Mutant peptides of length 8–11 amino acids covering each mutation (extracted by in-home scripts)
- Patient HLA alleles typed by OptiType 1.3.5
- Optional gene expression (TPM) values from TSNAD Module 4 for expression filtering

## Outputs

- Ranked pMHC neoantigen results with binding and immunogenicity scores (neoantigen_predictions.tsv)
- Potential neoantigen set (binding score > 0.5 and immunogenic score > 0.5)
- High-confidence neoantigen flags (immunogenic score > 0.5 and binding score ranked top 20 per peptide-length class)
- Files written to results/deephlapan_results/ in the TSNAD output directory

## How to apply

Take SNV/INDEL calls from GATK Mutect2 and annotate them with VEP 104 to obtain amino-acid-level mutation information, then use in-home scripts to extract all mutant peptides covering each mutation with lengths 8–11 amino acids. Combine these mutant peptides with the HLA alleles identified by OptiType (Module 2) and run DeepHLApan 1.1, which — unlike NetMHCpan — outputs both a peptide-MHC binding score and an immunogenicity score for each presented pMHC pair. Retain as potential neoantigens every pair with binding score > 0.5 AND immunogenic score > 0.5. Flag high-confidence neoantigens as pairs with immunogenic score > 0.5 whose binding score ranks in the top 20 within each peptide-length class, yielding up to 80 high-confidence neoantigens per sample. Optionally intersect with the gene-expression filter (drop mutations in genes with TPM < 1 from the Module 4 RNA-Seq analysis) before final ranking.

## Related tools

- **DeepHLApan 1.1** (Scores each candidate peptide-HLA pair for both peptide-MHC binding and immunogenicity, replacing NetMHCpan in the TSNAD v2.0 prediction module)
- **VEP 104** (Annotates Mutect2 SNV/INDEL calls to provide the amino-acid-level mutation information from which mutant peptides are extracted) — https://github.com/Ensembl/ensembl-vep
- **OptiType 1.3.5** (Supplies the patient HLA alleles that are combined with the extracted mutant peptides as DeepHLApan input)
- **TSNAD v2.0** (One-stop pipeline that orchestrates peptide extraction, DeepHLApan scoring, and threshold-based neoantigen filtering) — https://github.com/jiujiezz/tsnad

## Examples

```
python TSNAD.py -I samples/ -R RNA-seq/ -V grch37 -O results/
```

## Evaluation signals

- Every retained potential neoantigen pair has both binding score > 0.5 and immunogenic score > 0.5
- All scored peptides are within the 8–11 amino acid length range extracted from VEP-annotated mutations
- High-confidence pairs have immunogenic score > 0.5 and binding score ranked top 20 within each peptide-length class (maximum ~80 high-confidence neoantigens per sample)
- Scored results with binding and immunogenicity values and high-confidence flags appear in neoantigen_predictions.tsv under results/deephlapan_results/
- Predicted high-confidence neoantigens can be benchmarked against immunogenicity-validated sets such as the TESLA dataset (608 tested neoantigens, 37 / 6.1% found immunogenic)

## Limitations

- Thresholds (score > 0.5, rank top 20 per length) are empirical cutoffs from the TSNAD v2.0 methods, not externally validated probabilities
- No TCR-pMHC binding prediction model is included; the authors state more effort is needed to understand how pMHC induces T-cell response
- TSNAD v2.0 is limited to MHC-I neoantigen prediction from WES/WGS/RNA-Seq with HLA alleles from OptiType; TCR sequencing analysis is planned only for a future update

## Evidence
