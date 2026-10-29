---
name: genotyping-array-data-processing
description: Use when raw genotyping array CEL files need conversion to a filtered
  VCF, or when RNA-seq variant calls must be validated against an independent genotyping
  panel by sample-level concordance.
when_to_use_negative:
- The goal is ASE detection from RNA-seq VCFs rather than genotype validation — use
  VADT's binomial-test ASE pipeline instead of concordance comparison
- The input is already an annotated/filtered feature table rather than CEL files or
  VCFs — no Axiom or VCF comparison step applies
- Genotyping and RNA-seq data come from non-overlapping samples — the comparison requires
  a matching key file and overlapping sample IDs
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: Axiom Analysis Suite Software
  role: Processes raw 600K Axiom genotyping array CEL files into a VCF using the Best
    Practices Workflow for agricultural animals
- name: Compare_VCF_Files
  role: Filters and compares genotyping and RNA-seq VCFs, matches samples and overlapping
    SNPs, and computes per-sample genotype concordance
  repo: https://github.com/mjtiv/Compare_VCF_Files
- name: GATK
  role: Produces the RNA-seq variant call VCF via the best practices pipeline (Split'N'Trim,
    Base Recalibration, HaplotypeCaller) and provides the hard-filter criteria (Fisher
    Strand > 30.0, Quality Depth < 2.0, Depth < 100)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:da61be5429616e02b85256334254926b079227fd0e71ba7318f4ab699e26ddd6
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genotyping-array-data-processing

## Summary

Workflow for processing 600K Axiom Chicken Genotyping Array CEL data (GSE131764) into a filtered VCF and validating RNA-seq variant calls by per-sample genotype concordance (reported ~99.46% self-by-self vs 54.30% random-pairing baseline).

## When to use

Use when raw genotyping array CEL files need conversion to a filtered VCF, or when RNA-seq variant calls must be validated against an independent genotyping panel by sample-level concordance.

## When NOT to use

- The goal is ASE detection from RNA-seq VCFs rather than genotype validation — use VADT's binomial-test ASE pipeline instead of concordance comparison
- The input is already an annotated/filtered feature table rather than CEL files or VCFs — no Axiom or VCF comparison step applies
- Genotyping and RNA-seq data come from non-overlapping samples — the comparison requires a matching key file and overlapping sample IDs

## Inputs

- Raw genotyping CEL files from GSE131764 (600K ThermoFisher Axiom Chicken Genotyping Array)
- Genotyping VCF exported from Axiom Analysis Suite Software
- Probeset_id txt file listing passing probe set IDs from Axiom software
- RNA-seq multi-sample VCF produced by GATK's best practices variant calling pipeline (n = 68 samples)
- Matching key txt file relating genotyping panel sample names to RNA-seq sample names

## Outputs

- Filtered genotyping VCF (SNPs ≥ 97% call rate, overlapping samples only)
- Filtered RNA-seq VCF (GATK PASS variants, quality ≥ 20, read count ≥ 20, ≥ 75 bp from INDELs)
- genotype_concordance_report.csv with per-sample and average self-by-self concordance values
- Random-pairing baseline concordance matrix from shuffled sample pairings

## How to apply

Process raw genotyping CEL files with Axiom Analysis Suite Software (version 3.0.1, 64-bit) using the Best Practices Workflow for agricultural animals on the Gallus gallus 5.0 genome build, exporting results as a VCF. Filter the genotyping VCF to remove all SNPs with call rates below 97% and all non-overlapping samples. Filter the RNA-seq VCF (GATK pipeline) by removing variants failing GATK hard filters (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100), SNPs within 75 bp of INDELs, quality score < 20, and SNPs with fewer than 20 read counts. Compare the two filtered VCFs with Compare_VCF_Files, which matches samples and overlapping SNPs (~13,997 on average) and computes per-sample concordance; additionally shuffle sample pairings to build a random-baseline concordance matrix.

## Related tools

- **Axiom Analysis Suite Software** (Processes raw 600K Axiom genotyping array CEL files into a VCF using the Best Practices Workflow for agricultural animals)
- **Compare_VCF_Files** (Filters and compares genotyping and RNA-seq VCFs, matches samples and overlapping SNPs, and computes per-sample genotype concordance) — https://github.com/mjtiv/Compare_VCF_Files
- **GATK** (Produces the RNA-seq variant call VCF via the best practices pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller) and provides the hard-filter criteria (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100))

## Examples

```
python Compare_VCF_Files.py --Genotyping_VCF axiom_600k.vcf --Probeset_ID passing_probesets.txt --RNAseq_VCF rnaseq_gatk.vcf --Matching_Key sample_key.txt --Exclusion_Region_Length 75 --Minimum_Quality_Score 20 --Minimum_Read_Count 20
```

## Evaluation signals

- Average self-by-self concordance approximates the reported 99.46% (self-by-self 99.50%)
- Random-pairing baseline concordance from shuffled sample pairings is near the reported 54.30%, and self-by-self concordance clearly exceeds it
- Number of overlapping SNPs matched between the two filtered VCFs is in the reported range (~13,997 on average)
- Filtered genotyping VCF retains only SNPs with ≥ 97% call rates and samples overlapping the RNA-seq cohort (n = 68)
- genotype_concordance_report.csv is populated with per-sample concordance values for all matched sample pairs

## Limitations

- A large number of SNPs are excluded due to low overlap between RNA-seq variant calls and the 600K commercial genotyping panel, attributed to limitations of commercial genotyping panels in general
- Only ~13,997 SNPs on average overlap between the two VCFs, limiting the marker set available for concordance assessment
- Comparison requires a manually created sample matching key file (e.g. in Excel saved as txt) relating genotyping panel IDs to RNA-seq IDs

## Evidence

- [methods] The 600K genotyping data was filtered to remove all SNPs with < 97% call rates and all non-overlapping samples: "The 600K genotyping data was filtered to remove all SNPs with < 97% call rates and all non-overlapping samples"
- [readme] Compare_VCF_Files description: [evidence span withheld — anchor status: external_doc]
- [abstract] RNA-seq variants from subset of samples: "using variants called from a large sub-set of the samples (n = 68)"
- [methods] GATK hard filters: "using GATK to remove variants with “Fisher Strand > 30.0”, “Quality Depth < 2.0” and “Depth < 100”"
- [discussion] Low overlap with genotyping panel: "This low overlap between the RNA-seq variant calls and genotyping panel is most likely due to limitations of commercial genotyping panels in general"
