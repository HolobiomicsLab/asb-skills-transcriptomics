---
name: allele-specific-peptide-prioritization
description: Use when patient-specific HLA allele lists (mhc-I.tsv / mhc-II.tsv) and
  per-variant-type variant_effects.tsv files are available and predicted neoantigens
  must be ranked per allele by MHC binding affinity.
when_to_use_negative:
- HLA genotypes have not yet been produced (mhc-i.tsv / mhc-II.tsv missing) — prioritization
  classes must be a subset of hlatyping.class
- Only a small set of candidate peptides needs direct affinity checking outside a
  workflow, with no variant-context wildtype counterparts available for agretopicity
  computation
- The goal is de novo peptide discovery from raw sequencing data — this skill only
  scores peptides already extracted from detected variants
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: netMHCpan
  role: Predicts MHC class I binding affinity for mutant and wildtype 8–11-mer epitopes
    per typed HLA allele
- name: netMHCIIpan
  role: Predicts MHC class II binding affinity for mutant and wildtype 13–15-mer epitopes
    per typed HLA allele
- name: IEDB resources
  role: Provides the T-cell epitope and immunogenicity prediction backends for MHC
    class I and II
- name: ScanNeo2
  role: Hosts the Snakemake prioritization rules that coordinate k-mer extraction,
    prediction jobs, scoring, and output writing
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: Snakemake
  role: Workflow engine executing the one-job-per-(allele, length, wt|mt) prioritization
    rules
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:d4a765d629a9cc769870f1ffe284a8706c22e8bc29692c3a5757944deb4df40f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - IEDB resources
  - ScanNeo2
  - Snakemake
  - netMHCIIpan
  - netMHCpan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Allele-specific peptide prioritization

## Summary

Prioritization stage of a neoantigen workflow in which variant-derived wildtype and mutant peptide k-mers are scored for binding affinity against the patient's typed HLA class I and II alleles using netMHCpan and netMHCIIpan, yielding a ranked, tab-delimited neoepitope list with agretopicity and immunogenicity metrics.

## When to use

Use when patient-specific HLA allele lists (mhc-I.tsv / mhc-II.tsv) and per-variant-type variant_effects.tsv files are available and predicted neoantigens must be ranked per allele by MHC binding affinity.

## When NOT to use

- HLA genotypes have not yet been produced (mhc-i.tsv / mhc-II.tsv missing) — prioritization classes must be a subset of hlatyping.class
- Only a small set of candidate peptides needs direct affinity checking outside a workflow, with no variant-context wildtype counterparts available for agretopicity computation
- The goal is de novo peptide discovery from raw sequencing data — this skill only scores peptides already extracted from detected variants

## Inputs

- Per-variant-type variant_effects.tsv files (SNV, indel, alt-splicing, exitron-splicing, gene-fusion sources)
- Typed HLA allele lists mhc-i.tsv and mhc-II.tsv from the hlatyping stage
- Variant amino-acid sequence context for wildtype and mutant transcripts
- Configuration specifying prioritization MHC classes, epitope lengths, and IEDB resources

## Outputs

- results/<sample>/prioritization/<variant_type>_<mhc_class>_neoepitopes.txt (tab-delimited per-variant-type neoepitope lists)
- results/<sample>/prioritization/<variant_type>_variant_effects.tsv
- Combined mhc-I_neoepitopes_all.txt and mhc-II_neoepitopes_all.txt aggregating all activated variant types
- Columns including allele, mt_epitope_seq_ic50, wt_epitope_seq_ic50, agretopicity, immunogenicity, self-similarity, NMD

## How to apply

Read the per-variant-type variant_effects.tsv files together with the HLA genotype calls and verify that the configured prioritization MHC class is a subset of hlatyping.class. Extract wildtype and mutant epitope k-mers at the configured lengths (8, 9, 10, 11 for MHC class I; 13, 14, 15 for MHC class II) from the variant amino-acid sequence context, then submit each k-mer in one prediction job per (allele, length, wt|mt) cell to netMHCpan for class I and netMHCIIpan for class II via the Snakemake prioritization rules. Compute the agretopicity score as mt_affinity/wt_affinity for each mutant/wildtype pair and collect the immunogenicity, self-similarity and NMD fields alongside. For frameshift-derived candidates, only transcripts that escape nonsense-mediated decay are carried through. Write per-variant-type <variant_type>_<mhc_class>_neoepitopes.txt plus <variant_type>_variant_effects.tsv, and aggregate across all activated variant types into mhc-I_neoepitopes_all.txt / mhc-II_neoepitopes_all.txt under results/<sample>/prioritization/. Optionally apply the paper's stringent post-hoc filters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, VAF ≥0.02) to reduce non-immunogenic calls while retaining validated epitopes.

## Related tools

- **netMHCpan** (Predicts MHC class I binding affinity for mutant and wildtype 8–11-mer epitopes per typed HLA allele)
- **netMHCIIpan** (Predicts MHC class II binding affinity for mutant and wildtype 13–15-mer epitopes per typed HLA allele)
- **IEDB resources** (Provides the T-cell epitope and immunogenicity prediction backends for MHC class I and II)
- **ScanNeo2** (Hosts the Snakemake prioritization rules that coordinate k-mer extraction, prediction jobs, scoring, and output writing) — https://github.com/ylab-hi/ScanNeo2/
- **Snakemake** (Workflow engine executing the one-job-per-(allele, length, wt|mt) prioritization rules)

## Evaluation signals

- Each output file exists in the prioritization directory and contains the required columns (allele, mt_epitope_seq_ic50, wt_epitope_seq_ic50, agretopicity)
- mhc-I_neoepitopes_all.txt / mhc-II_neoepitopes_all.txt aggregate neoepitopes from every activated variant type
- agretopicity equals mt_affinity/wt_affinity for each mutant/wildtype pair
- Exactly one prediction job exists per (allele, length, wt|mt) cell, with epitope lengths 8–11 for MHC-I and 13–15 for MHC-II
- On the TESLA benchmark, the pipeline recovers validated immunogenic peptides (ScanNeo2 detected 35 of 37 experimentally validated immunogenic peptides)

## Limitations

- Unfiltered prioritization captures many non-immunogenic peptides: ScanNeo2 detected 35 of 37 validated immunogenic peptides but also 476 non-immunogenic ones; stringent filters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, VAF ≥0.02) reduce non-immunogenic calls to 391 while retaining 34
- ScanNeo2 requires on average ~22.5% more CPU time than nextNEOpi
- Epitope-length conventions differ between sources (article: 13–25 aa for MHC-II; workflow implementation uses 13, 14, 15-mers), so configuration must be checked per run
- No specific Snakemake version is pinned in the article text (repository badge indicates snakemake ≥9.0.0)

## Evidence

- [results] IEDB epitope and immunogenicity prediction: "the Immune Epitope Database and Analysis Resource (IEDB) (Dhanda et al. 2019) is used to predict the T-cell epitope for MHC class I and II as well as the immunogenicity"
- [results] Peptide length conventions: "peptides of length 8–11 and 13–25 amino acids, corresponding to MHC class I and MHC class II peptides, respectively"
- [results] Stringent filtering parameters: "we applied ScanNeo2 with more stringent parameters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, and VAF ≥0.02)"
- [results] Benchmark sensitivity: "ScanNeo2 detects 35 of the 37 experimentally validated immunogenic peptides but also captures 476 non-immunogenic peptides"
- [methods] Workflow installation: "git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2"
