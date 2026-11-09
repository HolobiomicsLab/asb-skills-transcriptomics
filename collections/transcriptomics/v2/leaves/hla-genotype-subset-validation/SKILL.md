---
name: hla-genotype-subset-validation
description: 'Use when running ScanNeo2''s prioritization stage: before submitting
  peptide-binding prediction jobs, confirm the requested MHC class(es) and allele
  lists match the HLA genotyping results (mhc-I.tsv / mhc-II.tsv).'
when_to_use_negative:
- No HLA genotyping results exist yet for the sample — run Optitype/HLA-HD typing
  first; this skill only validates an already-typed allele set.
- You intend to predict binding for HLA alleles outside the patient's typed genotype
  (e.g. a fixed allele panel) — ScanNeo2's prioritization explicitly restricts prediction
  to hlatyping.class outputs.
- You need MHC-II epitope lengths beyond the implemented 13–15-mers (the article describes
  13–25-mers for MHC class II, but the workflow restricts extraction to 13,14,15).
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ScanNeo2
  role: Workflow whose prioritization rules implement the subset check between configured
    prediction classes and hlatyping.class before prediction
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: Optitype
  role: Source of the typed HLA class I allele list that defines the allowed allele
    subset for netMHCpan jobs
- name: HLA-HD
  role: Source of the typed HLA class II allele list that defines the allowed allele
    subset for netMHCIIpan jobs
- name: netMHCpan
  role: MHC class I binding-affinity predictor executed only for alleles passing the
    subset validation
- name: netMHCIIpan
  role: MHC class II binding-affinity predictor executed only for alleles passing
    the subset validation
- name: Snakemake
  role: Encodes the prioritization rules and the class-subset configuration check
    (snakemake >= 9.0.0 per README badge)
  repo: https://github.com/ylab-hi/ScanNeo2/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:91501c83e8d458b5bdfe7c59e774d537bae06549c8062d5d508d6aee232d8325
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - HLA-HD
  - Optitype
  - ScanNeo2
  - Snakemake
  - netMHCIIpan
  - netMHCpan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hla-genotype-subset-validation

## Summary

A guard step in ScanNeo2's prioritization stage that verifies the configured prediction MHC classes (and every submitted HLA allele) are a strict subset of the alleles actually typed by Optitype (class I) and HLA-HD (class II), before any netMHCpan/netMHCIIpan jobs are launched.

## When to use

Use when running ScanNeo2's prioritization stage: before submitting peptide-binding prediction jobs, confirm the requested MHC class(es) and allele lists match the HLA genotyping results (mhc-I.tsv / mhc-II.tsv).

## When NOT to use

- No HLA genotyping results exist yet for the sample — run Optitype/HLA-HD typing first; this skill only validates an already-typed allele set.
- You intend to predict binding for HLA alleles outside the patient's typed genotype (e.g. a fixed allele panel) — ScanNeo2's prioritization explicitly restricts prediction to hlatyping.class outputs.
- You need MHC-II epitope lengths beyond the implemented 13–15-mers (the article describes 13–25-mers for MHC class II, but the workflow restricts extraction to 13,14,15).

## Inputs

- Typed HLA class I allele list per sample (mhc-I.tsv, from Optitype)
- Typed HLA class II allele list per sample (mhc-II.tsv, from HLA-HD)
- ScanNeo2 Snakemake config specifying hlatyping.class and prioritization MHC class(es)
- Per-variant-type variant_effects.tsv files (SNVs/indels, alternative splicing, exitron-splicing, gene fusions)

## Outputs

- Validated (MHC class, allele, k-mer length, wt|mt) prediction job set, restricted to typed alleles
- Per-variant-type neoepitope tables (<variant_type>_<mhc_class>_neoepitopes.txt) in results/<sample>/prioritization/
- Aggregated mhc-I_neoepitopes_all.txt and mhc-II_neoepitopes_all.txt

## How to apply

Read the per-sample typed HLA allele lists (mhc-I.tsv / mhc-II.tsv) produced by the hlatyping stage (Optitype for class I via yara-filtered reads, HLA-HD for class II) together with the per-variant-type variant_effects.tsv files. Check the workflow config so that the MHC class(es) requested for prioritization are a subset of hlatyping.class; fail fast if a class was configured for prediction but never genotyped. Only then extract wildtype/mutant k-mers (8,9,10,11-mers for MHC-I; 13,14,15-mers for MHC-II in the implementation) and submit one prediction job per (allele, length, wt|mt) cell — netMHCpan for class I, netMHCIIpan for class II. The rationale is that IEDB-based prediction must be restricted to the patient's own HLA alleles, otherwise the neoepitope list contains irrelevant allele-peptide pairs and the downstream agretopicity (mt_affinity/wt_affinity) scoring is meaningless.

## Related tools

- **ScanNeo2** (Workflow whose prioritization rules implement the subset check between configured prediction classes and hlatyping.class before prediction) — https://github.com/ylab-hi/ScanNeo2/
- **Optitype** (Source of the typed HLA class I allele list that defines the allowed allele subset for netMHCpan jobs)
- **HLA-HD** (Source of the typed HLA class II allele list that defines the allowed allele subset for netMHCIIpan jobs)
- **netMHCpan** (MHC class I binding-affinity predictor executed only for alleles passing the subset validation)
- **netMHCIIpan** (MHC class II binding-affinity predictor executed only for alleles passing the subset validation)
- **Snakemake** (Encodes the prioritization rules and the class-subset configuration check (snakemake >= 9.0.0 per README badge)) — https://github.com/ylab-hi/ScanNeo2/

## Evaluation signals

- The prioritization MHC class configuration is a strict subset of hlatyping.class for every sample (no prediction class without a corresponding typed allele list).
- Every (allele, length, wt|mt) prediction job maps to an allele present in mhc-I.tsv or mhc-II.tsv — no jobs against untyped alleles.
- Each output file <variant_type>_<mhc_class>_neoepitopes.txt exists in results/<sample>/prioritization/ and contains the required columns: allele, mt_epitope_seq_ic50, wt_epitope_seq_ic50, agretopicity.
- mhc-I_neoepitopes_all.txt and mhc-II_neoepitopes_all.txt aggregate rows from every activated variant type (SNV/indel, alternative splicing, exitron, fusion).
- Agretopicity equals mt_epitope_seq_ic50 / wt_epitope_seq_ic50 for each mutant/wildtype pair (recomputable from output columns).

## Limitations

- Prediction is restricted to the HLA allele panel covered by Optitype (class I) and HLA-HD (class II); alleles outside these panels cannot be prioritized.
- MHC-II prediction lengths in the workflow are limited to 13, 14 and 15-mers, narrower than the article's stated 13–25 amino acid class II peptide range.
- If a prioritization MHC class is configured but the corresponding genotyping was skipped (not in hlatyping.class), no valid predictions can be produced for that class.

## Evidence

- [results] DNA-seq and RNA-seq data are filtered for HLA reads using yara (Siragusa et al. 2013) and subsequently aligned against a panel of HLA class I alleles using Optitype: "DNA-seq and RNA-seq data are filtered for HLA reads using yara (Siragusa et al. 2013) and subsequently aligned against a panel of HLA class I alleles using Optitype"
- [results] HLA-HD (Kawaguchi et al. 2017) is used for HLA class II genotyping: "HLA-HD (Kawaguchi et al. 2017) is used for HLA class II genotyping"
- [results] the Immune Epitope Database and Analysis Resource (IEDB) (Dhanda et al. 2019) is used to predict the T-cell epitope for MHC class I and II as well as the immunogenicity: "the Immune Epitope Database and Analysis Resource (IEDB) (Dhanda et al. 2019) is used to predict the T-cell epitope for MHC class I and II as well as the immunogenicity"
- [methods] git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2: "git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2"
