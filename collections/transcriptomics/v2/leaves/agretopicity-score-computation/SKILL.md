---
name: agretopicity-score-computation
description: Use when you have ScanNeo2 variant_effects.tsv entries plus typed HLA
  allele lists (mhc-I.tsv / mhc-II.tsv) and need to rank mutant-vs-wildtype MHC binding
  predictions into agretopicity-scored neoepitope tables.
when_to_use_negative:
- The prioritization MHC class is not a subset of the detected hlatyping.class (e.g.
  class I prioritization requested but only HLA-HD class II alleles were typed) —
  the (allele, length) prediction grid cannot be built.
- Variants come from outside ScanNeo2's variant_effects.tsv outputs or lack a matched
  wildtype peptide context, so the mt/wt IC50 pair required for the agretopicity ratio
  is unavailable.
- The goal is HLA genotyping itself (yara + Optitype for class I, HLA-HD for class
  II) rather than peptide-level neoepitope prioritization.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: netMHCpan
  role: Predicts MHC class I binding affinity (IC50) for wildtype and mutant k-mers
    per (allele, length) job; its outputs feed the mt/wt agretopicity ratio
- name: netMHCIIpan
  role: Predicts MHC class II binding affinity for the 13–15 aa wildtype/mutant k-mer
    pairs used in class II agretopicity computation
- name: ScanNeo2
  role: Snakemake workflow whose prioritization rules orchestrate k-mer extraction,
    one prediction job per (allele, length, wt|mt) cell, agretopicity computation
    and neoepitope table output
  repo: https://github.com/ylab-hi/ScanNeo2/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:010879f9d58cdd5110ea033519e9d055992896e1fb814610e1a578e280504547
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - ScanNeo2
  - netMHCIIpan
  - netMHCpan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# agretopicity-score-computation

## Summary

Computes the agretopicity score (mutant affinity / wildtype affinity, i.e. mt_epitope_seq_ic50 / wt_epitope_seq_ic50) for each candidate neoepitope in ScanNeo2's prioritization stage, using netMHCpan and netMHCIIpan binding predictions against the sample's typed HLA alleles.

## When to use

Use when you have ScanNeo2 variant_effects.tsv entries plus typed HLA allele lists (mhc-I.tsv / mhc-II.tsv) and need to rank mutant-vs-wildtype MHC binding predictions into agretopicity-scored neoepitope tables.

## When NOT to use

- The prioritization MHC class is not a subset of the detected hlatyping.class (e.g. class I prioritization requested but only HLA-HD class II alleles were typed) — the (allele, length) prediction grid cannot be built.
- Variants come from outside ScanNeo2's variant_effects.tsv outputs or lack a matched wildtype peptide context, so the mt/wt IC50 pair required for the agretopicity ratio is unavailable.
- The goal is HLA genotyping itself (yara + Optitype for class I, HLA-HD for class II) rather than peptide-level neoepitope prioritization.

## Inputs

- Per-variant-type variant_effects.tsv files (ScanNeo2 variant-effect tables)
- Typed HLA allele lists for the sample: mhc-I.tsv (Optitype) and mhc-II.tsv (HLA-HD)
- Wildtype and mutant epitope k-mer sequences (8–11 aa for MHC-I, 13–15 aa for MHC-II)
- netMHCpan / netMHCIIpan IC50 binding-affinity outputs for each (allele, length, wt|mt) prediction job

## Outputs

- <variant_type>_<mhc_class>_neoepitopes.txt per variant type and MHC class
- <variant_type>_variant_effects.tsv per variant type
- mhc-I_neoepitopes_all.txt aggregating all activated MHC-I variant types
- mhc-II_neoepitopes_all.txt aggregating all activated MHC-II variant types
- All written under results/<sample>/prioritization/

## How to apply

Read the per-variant-type variant_effects.tsv files and the typed HLA allele lists for the sample, and verify that the configured prioritization MHC class is a subset of hlatyping.class before predicting. Extract wildtype and mutant epitope k-mers at the configured lengths (8, 9, 10, 11 for MHC-I; 13, 14, 15 for MHC-II) and submit each k-mer in one prediction job per (allele, length, wt|mt) cell — netMHCpan for class I and netMHCIIpan for class II, with the IEDB analysis resource supplying T-cell epitope and immunogenicity prediction. For each mutant/wildtype pair, compute agretopicity as mt_affinity/wt_affinity from the predicted IC50 values, alongside the immunogenicity, self-similarity and NMD fields collected per variant. Write the per-variant-type <variant_type>_<mhc_class>_neoepitopes.txt and <variant_type>_variant_effects.tsv files plus the combined mhc-I_neoepitopes_all.txt / mhc-II_neoepitopes_all.txt to results/<sample>/prioritization/.

## Related tools

- **netMHCpan** (Predicts MHC class I binding affinity (IC50) for wildtype and mutant k-mers per (allele, length) job; its outputs feed the mt/wt agretopicity ratio)
- **netMHCIIpan** (Predicts MHC class II binding affinity for the 13–15 aa wildtype/mutant k-mer pairs used in class II agretopicity computation)
- **ScanNeo2** (Snakemake workflow whose prioritization rules orchestrate k-mer extraction, one prediction job per (allele, length, wt|mt) cell, agretopicity computation and neoepitope table output) — https://github.com/ylab-hi/ScanNeo2/

## Evaluation signals

- Each output file exists in results/<sample>/prioritization/ and contains the required columns: allele, mt_epitope_seq_ic50, wt_epitope_seq_ic50, agretopicity.
- For every row, agretopicity reproduces mt_epitope_seq_ic50 / wt_epitope_seq_ic50 within rounding tolerance; values < 1 indicate the mutant peptide binds more strongly (lower IC50) than its wildtype counterpart.
- mhc-I_neoepitopes_all.txt (and the class II equivalent) aggregates neoepitopes from every activated variant type, matching the union of the per-variant-type files.
- All configured k-mer lengths are represented (8, 9, 10, 11 for MHC-I; 13, 14, 15 for MHC-II) and exactly one prediction job was run per (allele, length, wt|mt) cell.
- Immunogenicity, self-similarity and NMD fields are populated alongside agretopicity, enabling downstream stringency filters (immunogenicity score >= 0.5, TPM >= 2, ranking score >= 1000, VAF >= 0.02).

## Limitations

- The score depends entirely on predicted (not measured) IC50 values from netMHCpan/netMHCIIpan via the IEDB resource; unfiltered output retains many false positives — on the TESLA benchmark ScanNeo2 detected 35 of 37 validated immunogenic peptides but also 476 non-immunogenic ones, reduced to 391 only under stringent parameters.
- Frameshift-derived candidates are restricted to transcripts escaping nonsense-mediated decay, so NMD-targeted variants yield no agretopicity-scored neoepitopes.
- The article does not pin a Snakemake version, and the ScanNeo2 README badge requires snakemake >= 9.0.0; results may vary across workflow/tool versions.

## Evidence

- [results] the Immune Epitope Database and Analysis Resource (IEDB) (Dhanda et al. 2019) is used to predict the T-cell epitope for MHC class I and II as well as the immunogenicity: "the Immune Epitope Database and Analysis Resource (IEDB) (Dhanda et al. 2019) is used to predict the T-cell epitope for MHC class I and II as well as the immunogenicity"
- [results] peptides of length 8–11 and 13–25 amino acids, corresponding to MHC class I and MHC class II peptides, respectively: "peptides of length 8–11 and 13–25 amino acids, corresponding to MHC class I and MHC class II peptides, respectively"
- [methods] git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2: "git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2"
