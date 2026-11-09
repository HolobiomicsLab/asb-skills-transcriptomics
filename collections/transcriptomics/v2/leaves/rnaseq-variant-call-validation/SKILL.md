---
name: rnaseq-variant-call-validation
description: Use when RNA-seq-derived VCFs need validation against genotype array
  data (e.g., Axiom SNP array VCF from GSE131764) to confirm variant-calling fidelity
  before ASE or downstream analyses.
when_to_use_negative:
- Input variant calls were not produced by GATK HaplotypeCaller — the README warns
  VCFs from other callers may have format incompatibilities with Compare_VCF_Files/VADT
- There is no independent genotyping dataset overlapping the RNA-seq samples, so no
  self-by-self vs random-pairing contrast can be computed
- The goal is ASE detection itself rather than variant-call QC — use VADT's binomial
  test pipeline instead
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_0622
tools:
- name: Compare_VCF_Files
  role: Filters both VCFs on user parameters, matches overlapping SNPs and samples
    via a matching key, and computes per-sample genotype concordance
  repo: https://github.com/mjtiv/Compare_VCF_Files
- name: Axiom Analysis Suite Software
  role: Processes raw genotyping CEL files (v3.0.1, Best Practices Workflow, Gallus
    gallus 5.0) to produce the genotyping VCF
- name: GATK
  role: 'RNA-seq variant calling pipeline (v3.7: Split''N''Trim, Base Recalibration,
    HaplotypeCaller with g.vcf setting) whose output is validated'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:292f1d9c122063784bc887cd87ccc50be4b6fb93fab83d0f0215f9440cac2458
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/Compare_VCF_Files
  tools:
  - Axiom Analysis Suite Software
  - Compare_VCF_Files
  - GATK
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rnaseq-variant-call-validation

## Summary

Validate RNA-seq variant calls (from GATK HaplotypeCaller, g.vcf workflow) by comparing per-sample genotypes against an independent genotyping platform (Axiom 600K chicken array VCF) using Compare_VCF_Files, reporting self-by-self vs randomly-paired concordance.

## When to use

Use when RNA-seq-derived VCFs need validation against genotype array data (e.g., Axiom SNP array VCF from GSE131764) to confirm variant-calling fidelity before ASE or downstream analyses.

## When NOT to use

- Input variant calls were not produced by GATK HaplotypeCaller — the README warns VCFs from other callers may have format incompatibilities with Compare_VCF_Files/VADT
- There is no independent genotyping dataset overlapping the RNA-seq samples, so no self-by-self vs random-pairing contrast can be computed
- The goal is ASE detection itself rather than variant-call QC — use VADT's binomial test pipeline instead

## Inputs

- Axiom genotyping VCF exported from Axiom Analysis Suite Software (GSE131764 CEL files, 600K Axiom Chicken Genotyping Array)
- Probeset_id txt file listing passing probe set IDs from the Axiom software
- Multi-sample RNA-seq VCF produced by GATK 3.7 HaplotypeCaller (g.vcf setting), same tissue per VCF
- Matching key txt file relating genotyping sample names (column 1) to RNA-seq sample names (column 2), header 'Key'/'Value'

## Outputs

- Per-sample concordance values in genotype_concordance_report.csv
- Average self-by-self concordance (expected ~99.46% average; 99.50% self-by-self)
- Random-pairing baseline concordance matrix (expected baseline ~54.30%)
- Report of matching variants and samples

## How to apply

Process the raw genotyping CEL files from GSE131764 with Axiom Analysis Suite Software (v3.0.1, Best Practices Workflow for agricultural animals on Gallus gallus 5.0) and export a VCF; filter it to remove SNPs with <97% call rates and non-overlapping samples. Independently, call RNA-seq variants with GATK 3.7 (Split'N'Trim, Base Recalibration, HaplotypeCaller with g.vcf setting) from the subset of samples (n=68), then filter: remove GATK-flag failures (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100), SNPs within 75 bp of INDELs, quality score < 20, and read count < 20 per sample. Run Compare_VCF_Files with a matching-key file linking genotyping sample IDs to RNA-seq sample IDs; it matches overlapping SNPs (~13,997 on average) and computes per-sample concordance. Shuffle sample pairings to build a random-pairing baseline concordance matrix, then contrast self-by-self concordance against this baseline.

## Related tools

- **Compare_VCF_Files** (Filters both VCFs on user parameters, matches overlapping SNPs and samples via a matching key, and computes per-sample genotype concordance) — https://github.com/mjtiv/Compare_VCF_Files
- **Axiom Analysis Suite Software** (Processes raw genotyping CEL files (v3.0.1, Best Practices Workflow, Gallus gallus 5.0) to produce the genotyping VCF)
- **GATK** (RNA-seq variant calling pipeline (v3.7: Split'N'Trim, Base Recalibration, HaplotypeCaller with g.vcf setting) whose output is validated)

## Examples

```
python Compare_VCF_Files.py --Genotype_VCF_File axiom.vcf --Probeset_ID_File probeset_ids.txt --RNA_Seq_VCF_File rnaseq.vcf --Matching_Key_File key.txt --Exclusion_Region_Length 75 --Minimum_Quality_Score 20 --Minimum_Read_Count 20
```

## Evaluation signals

- Average self-by-self concordance approximates the reported 99.46% (self-by-self 99.50%)
- Self-by-self concordance clearly exceeds the randomly-paired baseline of 54.30%
- Number of overlapping SNPs matched between the two filtered VCFs is on average ~13,997
- Genotyping VCF retains only SNPs with ≥97% call rates and overlapping samples
- Both VCFs pass the defined filters: quality score ≥ 20, read count ≥ 20, exclusion region of 75 bp around INDELs

## Limitations

- A large number of SNPs were excluded due to low overlap with the 600K chicken genotyping panel, attributed to limitations of commercial genotyping panels in general
- SNPs passing filters average ~174,000 per tissue, but only ~13,997 overlap the genotyping panel for concordance analysis
- Compare_VCF_Files requires a manually created matching-key file relating genotyping sample IDs to RNA-seq sample IDs

## Evidence

- [other] RNA-seq variants for the ASE analysis were called from a large subset of 68 samples: [evidence span withheld — anchor status: not_found]
- [methods] GATK filter thresholds applied to the RNA-seq VCF: [evidence span withheld — anchor status: not_found]
- [methods] Genotyping array filtering criterion: "The 600K genotyping data was filtered to remove all SNPs with < 97% call rates and all non-overlapping samples"
- [readme] Compare_VCF_Files program description from README: [evidence span withheld — anchor status: external_doc]
- [readme] Matching key file requirement: [evidence span withheld — anchor status: external_doc]
- [methods] Concordance comparison citation from article: "compares concordance of matching samples and SNPs (https://github.com/mjtiv/Compare_VCF_Files)"
