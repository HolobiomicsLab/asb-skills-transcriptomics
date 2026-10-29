---
name: neoepitope-kmer-extraction-from-variant-proteins
description: Use when HLA genotypes (mhc-I.tsv / mhc-II.tsv) and per-variant variant_effects.tsv
  files are available and predicted MHC-binding neoepitopes with mutant-vs-wildtype
  affinity scores must be produced.
when_to_use_negative:
- HLA genotypes have not yet been determined — run the yara/Optitype (class I) and
  HLA-HD (class II) typing modules first, since prediction requires typed alleles
- Variant effects (SNV/indel, alternative splicing, exitron-splicing, gene fusions)
  have not been detected and translated into variant protein sequences yet
- The desired output is raw variant calls rather than binding-predicted neoepitope
  candidates
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0160
tools:
- name: netMHCpan
  role: Predicts MHC class I binding affinity (IC50) for each (allele, length, wt|mt)
    k-mer cell
- name: netMHCIIpan
  role: Predicts MHC class II binding affinity (IC50) for each (allele, length, wt|mt)
    k-mer cell
- name: IEDB
  role: Underlying resource for T-cell epitope and immunogenicity prediction in the
    prioritization stage
- name: ScanNeo2
  role: Wraps k-mer extraction, prediction jobs, agretopicity scoring and output writing
    as Snakemake prioritization rules
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: Snakemake
  role: Workflow engine orchestrating the per-(allele, length, wt|mt) prediction jobs
    and aggregation into the combined output files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:4384a597cacda57827cd2da8418ad37c2a7f7abe43d08bbaaede782e0008f4c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - IEDB
  - ScanNeo2
  - Snakemake
  - netMHCIIpan
  - netMHCpan
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Neoepitope k-mer extraction from variant proteins

## Summary

Extracts wildtype and mutant epitope k-mers (8–11 aa for MHC-I; 13–15 aa for MHC-II in ScanNeo2's prioritization stage) from variant-derived protein sequences so they can be submitted to netMHCpan/netMHCIIpan binding-affinity prediction and scored into a final neoepitope list.

## When to use

Use when HLA genotypes (mhc-I.tsv / mhc-II.tsv) and per-variant variant_effects.tsv files are available and predicted MHC-binding neoepitopes with mutant-vs-wildtype affinity scores must be produced.

## When NOT to use

- HLA genotypes have not yet been determined — run the yara/Optitype (class I) and HLA-HD (class II) typing modules first, since prediction requires typed alleles
- Variant effects (SNV/indel, alternative splicing, exitron-splicing, gene fusions) have not been detected and translated into variant protein sequences yet
- The desired output is raw variant calls rather than binding-predicted neoepitope candidates

## Inputs

- Per-variant-type variant_effects.tsv files (variant effect tables from upstream variant calling modules)
- Typed HLA allele lists: mhc-I.tsv and mhc-II.tsv per sample
- Configuration: prioritization MHC classes (must be a subset of hlatyping.class) and epitope lengths (MHC-I: 8,9,10,11; MHC-II: 13,14,15)

## Outputs

- results/<sample>/prioritization/<variant_type>_<mhc_class>_neoepitopes.txt (per-variant-type neoepitope tables)
- results/<sample>/prioritization/<variant_type>_variant_effects.tsv
- Combined mhc-I_neoepitopes_all.txt and mhc-II_neoepitopes_all.txt aggregating all activated variant types
- Tab-delimited list of predicted neoantigens with columns including allele, mt_epitope_seq_ic50, wt_epitope_seq_ic50, agretopicity, immunogenicity, self-similarity, NMD

## How to apply

Read the per-variant-type variant_effects.tsv files together with the typed HLA allele lists (mhc-I.tsv / mhc-II.tsv), and verify that the configured prioritization MHC class is a subset of hlatyping.class. Extract wildtype and mutant epitope k-mers at the configured lengths — 8, 9, 10, 11 aa for MHC class I and 13, 14, 15 aa for MHC class II (ScanNeo2's article-level peptide windows are 8–11 and 13–25 amino acids for MHC-I and MHC-II respectively). Submit each k-mer in one prediction job per (allele, length, wt|mt) cell to netMHCpan for class I and netMHCIIpan for class II via the Snakemake prioritization rules; IEDB is the underlying resource for T-cell epitope and immunogenicity prediction. Compute the agretopicity score as mt_affinity/wt_affinity for each mutant/wildtype pair and collect immunogenicity, self-similarity and NMD fields. Rationale: predicting both mutant and wildtype affinity against the same typed alleles lets binding gains over the wildtype peptide (agretopicity) rank candidate neoantigens rather than raw IC50 alone.

## Related tools

- **netMHCpan** (Predicts MHC class I binding affinity (IC50) for each (allele, length, wt|mt) k-mer cell)
- **netMHCIIpan** (Predicts MHC class II binding affinity (IC50) for each (allele, length, wt|mt) k-mer cell)
- **IEDB** (Underlying resource for T-cell epitope and immunogenicity prediction in the prioritization stage)
- **ScanNeo2** (Wraps k-mer extraction, prediction jobs, agretopicity scoring and output writing as Snakemake prioritization rules) — https://github.com/ylab-hi/ScanNeo2/
- **Snakemake** (Workflow engine orchestrating the per-(allele, length, wt|mt) prediction jobs and aggregation into the combined output files)

## Examples

```
snakemake --use-conda results/<sample>/prioritization/mhc-I_neoepitopes_all.txt
```

## Evaluation signals

- Each expected output file exists in results/<sample>/prioritization/
- Neoepitope files contain the required columns: allele, mt_epitope_seq_ic50, wt_epitope_seq_ic50, agretopicity
- mhc-I_neoepitopes_all.txt (and mhc-II_neoepitopes_all.txt) aggregates neoepitopes from every activated variant type
- Agretopicity values equal mt_affinity/wt_affinity for each mutant/wildtype pair
- Prioritization MHC class is a subset of hlatyping.class (config consistency check passed)

## Limitations

- K-mer lengths are fixed per MHC class in the prioritization stage (8,9,10,11 for MHC-I; 13,14,15 for MHC-II), narrower than the article's general 13–25 aa MHC-II peptide window
- ScanNeo2 detects 35 of 37 TESLA-validated immunogenic peptides but also captures 476 non-immunogenic peptides without downstream filtering
- Stringent downstream filters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, VAF ≥0.02) retain only 34 of 37 validated peptides, so affinity prediction alone trades sensitivity for specificity

## Evidence

- [results] Peptide lengths per MHC class: "peptides of length 8–11 and 13–25 amino acids, corresponding to MHC class I and MHC class II peptides, respectively"
- [results] IEDB-based epitope and immunogenicity prediction: "the Immune Epitope Database and Analysis Resource (IEDB) (Dhanda et al. 2019) is used to predict the T-cell epitope for MHC class I and II as well as the immunogenicity"
- [results] Sensitivity trade-off of ScanNeo2 epitope detection: "ScanNeo2 detects 35 of the 37 experimentally validated immunogenic peptides but also captures 476 non-immunogenic peptides"
- [results] Stringent filter thresholds applied downstream: "we applied ScanNeo2 with more stringent parameters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, and VAF ≥0.02)"
- [results] Snakemake orchestration of the workflow: "We took advantage of those tools and developed a snakemake-based workflow for the detection of neoantigens from multiple sources, termed ScanNeo2"
