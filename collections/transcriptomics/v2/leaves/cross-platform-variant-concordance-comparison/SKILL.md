---
name: cross-platform-variant-concordance-comparison
description: Use when RNA-seq-derived variant calls (multi-sample VCF from GATK) must
  be quality-checked against an independent genotyping platform (e.g., Axiom array
  VCF) to confirm sample identity and call accuracy.
when_to_use_negative:
- VADT is the wrong tool here — use Compare_VCF_Files instead when the goal is genotype
  concordance between platforms, not allele-specific expression detection
- The two VCFs share no overlapping samples or the genotyping panel has negligible
  SNP overlap with the RNA-seq call set (the paper notes low overlap due to commercial
  panel limitations)
- Variant calls come from a caller other than GATK HaplotypeCaller in a non-standard
  VCF format, which the README warns could trigger parsing bugs
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_0622
tools:
- name: Compare_VCF_Files
  role: Filters the genotyping and RNA-seq VCFs, matches samples and overlapping SNPs,
    and computes per-sample genotype concordance
  repo: https://github.com/mjtiv/Compare_VCF_Files
- name: Axiom Analysis Suite Software
  role: Processes raw genotyping CEL files from the 600K Axiom Chicken Genotyping
    Array into a genotyping VCF
- name: GATK 3.7
  role: Produces the multi-sample RNA-seq VCF via the Best Practices pipeline (Split'N'Trim,
    Base Recalibration, HaplotypeCaller)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:9b8e79a072ffeb3b2abd366a1ab768b6bb5a903f021c4c574865b06e4d6e651e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/Compare_VCF_Files
  tools:
  - Axiom Analysis Suite Software
  - Compare_VCF_Files
  - GATK 3.7
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-platform-variant-concordance-comparison

## Summary

Validates RNA-seq variant calls against orthogonal genotyping array data by filtering both VCFs, matching samples and overlapping SNPs with Compare_VCF_Files, and computing per-sample genotype concordance (99.46% self-by-self vs 54.30% randomly-paired baseline in the broiler chicken ASE study).

## When to use

Use when RNA-seq-derived variant calls (multi-sample VCF from GATK) must be quality-checked against an independent genotyping platform (e.g., Axiom array VCF) to confirm sample identity and call accuracy.

## When NOT to use

- VADT is the wrong tool here — use Compare_VCF_Files instead when the goal is genotype concordance between platforms, not allele-specific expression detection
- The two VCFs share no overlapping samples or the genotyping panel has negligible SNP overlap with the RNA-seq call set (the paper notes low overlap due to commercial panel limitations)
- Variant calls come from a caller other than GATK HaplotypeCaller in a non-standard VCF format, which the README warns could trigger parsing bugs

## Inputs

- Genotyping VCF exported from Axiom Analysis Suite Software (e.g., from GSE131764 600K chicken array CEL files)
- Multi-sample RNA-seq VCF from GATK HaplotypeCaller (one tissue per VCF, n=68 samples)
- Probeset_id txt file listing passing probe set IDs from the Axiom software
- Sample matching key txt file mapping genotyping panel sample names to RNA-seq sample names (header 'Key','Value')
- User-defined filter parameters (indel exclusion length, minimum quality score, minimum read count)

## Outputs

- genotype_concordance_report.csv with per-sample and average self-by-self concordance values
- Report files of matching variants and matching samples
- Random-pairing baseline concordance matrix for contrast

## How to apply

Process raw genotyping CEL files (e.g., GSE131764 600K Axiom Chicken Genotyping Array) with Axiom Analysis Suite Software v3.0.1 using the Best Practices Workflow on Gallus gallus 5.0, exporting a VCF, then drop SNPs with <97% call rates and non-overlapping samples. Filter the RNA-seq VCF produced by GATK 3.7 by removing variants flagged as failing GATK filters (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100), SNPs within 75 bp of INDELs (the read-length exclusion distance), quality score < 20, and SNPs with < 20 read counts per sample. Supply a sample-matching key file that maps genotyping panel sample IDs to RNA-seq sample IDs, and run Compare_VCF_Files, which intersects the two VCFs (~13,997 overlapping SNPs on average), compares genotypes per matching sample, and writes per-sample and average concordance to genotype_concordance_report.csv. As a negative control, shuffle sample pairings and recompute concordance to build a random-pairing baseline matrix; genuine self-by-self concordance should far exceed it.

## Related tools

- **Compare_VCF_Files** (Filters the genotyping and RNA-seq VCFs, matches samples and overlapping SNPs, and computes per-sample genotype concordance) — https://github.com/mjtiv/Compare_VCF_Files
- **Axiom Analysis Suite Software** (Processes raw genotyping CEL files from the 600K Axiom Chicken Genotyping Array into a genotyping VCF)
- **GATK 3.7** (Produces the multi-sample RNA-seq VCF via the Best Practices pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller))

## Examples

```
python Compare_VCF_Files.py --Exclusion_Region_Length 75 --Minimum_Quality_Score 20 --Minimum_Read_Count 20
```

## Evaluation signals

- Average self-by-self concordance approximates the reported 99.46% (self-by-self 99.50%)
- Self-by-self concordance clearly exceeds the randomly-paired baseline of 54.30%
- On average ~13,997 overlapping SNPs are matched between the two filtered VCFs per sample comparison
- All genotyping SNPs with <97% call rates and non-overlapping samples are removed before comparison
- genotype_concordance_report.csv is produced with per-sample and average concordance values populated

## Limitations

- Low overlap between RNA-seq variant calls and the 600K commercial genotyping panel excludes a large number of SNPs, attributed to general limitations of commercial genotyping panels
- Only ~13,997 SNPs on average overlap, limiting the resolution of concordance estimates
- Compare_VCF_Files expects a GATK-HaplotypeCaller-formatted VCF; other callers' VCFs may need reformatting
- Requires a manually curated sample matching key (e.g., built in Excel) linking genotyping panel IDs to RNA-seq IDs

## Evidence

- [methods] Filter the genotyping VCF to remove all SNPs with < 97% call rates and all non-overlapping samples.: "The 600K genotyping data was filtered to remove all SNPs with < 97% call rates and all non-overlapping samples"
- [methods] GATK filter thresholds applied to RNA-seq variants.: "using GATK to remove variants with “Fisher Strand > 30.0”, “Quality Depth < 2.0” and “Depth < 100”"
- [methods] Indel-distance and quality thresholds used in the comparison.: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [readme] Compare_VCF_Files purpose.: [evidence span withheld — anchor status: external_doc]
- [readme] Sample matching key requirement.: [evidence span withheld — anchor status: external_doc]
- [abstract] RNA-seq variants called from 68-sample subset used for the validation.: "using variants called from a large sub-set of the samples (n = 68)"
