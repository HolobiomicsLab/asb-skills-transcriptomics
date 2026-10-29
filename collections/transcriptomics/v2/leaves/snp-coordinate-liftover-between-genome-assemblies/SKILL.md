---
name: snp-coordinate-liftover-between-genome-assemblies
description: Use when curated SNP positions are given on the Gmax_a1.v1 assembly but
  downstream analyses (e.g., gene proximity or coexpression module assignment) require
  Gmax_a2.v1 coordinates, and Soybase VCFs exist for both assemblies.
when_to_use_negative:
- The input SNP coordinates are already reported on the target assembly (Gmax_a2.v1)
  and no conversion is needed.
- VCF files are not available for both assemblies (a1.v1 and a2.v1), so allele-concordance
  anchoring cannot be performed.
- The goal is exploring the soybean-fungal-pathogen coexpression network via SoyFungiGCN
  rather than harmonizing SNP coordinates.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Scripting environment used to load the curated SNP data frame, parse both
    Soybase VCFs, match coordinates, and write converted_snp_coordinates.tsv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:ad4c223ca1260bf5e6a1f9a163a6c663b1ba0c39321e635ac0075a2163373e97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# snp-coordinate-liftover-between-genome-assemblies

## Summary

A procedure for converting manually curated resistance-associated SNP coordinates from an older soybean genome assembly (Gmax_a1.v1) to a newer one (Gmax_a2.v1), using the paired Soybase .vcf files for both assemblies to anchor each site by chromosome, position, and REF/ALT allele concordance, yielding a unified SNP coordinate set for downstream candidate gene analyses.

## When to use

Use when curated SNP positions are given on the Gmax_a1.v1 assembly but downstream analyses (e.g., gene proximity or coexpression module assignment) require Gmax_a2.v1 coordinates, and Soybase VCFs exist for both assemblies.

## When NOT to use

- The input SNP coordinates are already reported on the target assembly (Gmax_a2.v1) and no conversion is needed.
- VCF files are not available for both assemblies (a1.v1 and a2.v1), so allele-concordance anchoring cannot be performed.
- The goal is exploring the soybean-fungal-pathogen coexpression network via SoyFungiGCN rather than harmonizing SNP coordinates.

## Inputs

- Curated list of resistance-associated SNPs with positions on the Gmax_a1.v1 assembly (chromosome, position, allele fields, e.g., a data frame in R)
- Gmax_a1.v1 .vcf file from Soybase
- Gmax_a2.v1 .vcf file from Soybase

## Outputs

- converted_snp_coordinates.tsv: a TSV with one row per SNP, original Gmax_a1.v1 chromosome/position plus mapped Gmax_a2.v1 chromosome/position
- Explicit flags marking SNPs that could not be uniquely mapped between assemblies

## How to apply

Load the curated resistance SNP list into R as a data frame with chromosome, position, and allele fields. Download and parse the Gmax_a1.v1 and Gmax_a2.v1 .vcf files from Soybase, extracting chromosome names, variant positions, and REF/ALT alleles from each. For each curated SNP, match its a1.v1 chromosome and position against the a1.v1 VCF, then anchor the corresponding a2.v1 site by allele concordance in the a2.v1 VCF. Build the converted coordinate table with one row per SNP, retaining original a1.v1 coordinates alongside the mapped a2.v1 chromosome and position, and explicitly flag any SNP that cannot be uniquely mapped rather than silently dropping it. Write the result to converted_snp_coordinates.tsv and validate allele consistency across assemblies.

## Related tools

- **R** (Scripting environment used to load the curated SNP data frame, parse both Soybase VCFs, match coordinates, and write converted_snp_coordinates.tsv)

## Examples

```
write.table(converted_snps, "converted_snp_coordinates.tsv", sep = "\t", quote = FALSE, row.names = FALSE)
```

## Evaluation signals

- The output file exists and is a valid TSV containing the expected coordinate columns (a1.v1 chromosome/position and mapped a2.v1 chromosome/position).
- Mapped rows show consistent REF/ALT alleles across the two assemblies' VCFs.
- Each output row corresponds to exactly one input SNP; no SNPs are silently dropped.
- Any SNP that cannot be uniquely mapped is explicitly flagged in the output rather than omitted.
- All curated SNPs were matched against the a1.v1 VCF before anchoring their a2.v1 counterparts by allele concordance.

## Limitations

- SNPs that cannot be uniquely mapped between assemblies are flagged rather than converted, so the converted set may be a subset of the curated set.
- Anchoring relies on allele concordance between the two Soybase VCFs; discordant or missing variants prevent reliable mapping.
- The procedure is specific to converting Gmax_a1.v1 coordinates to Gmax_a2.v1; it does not generalize to arbitrary assembly pairs without corresponding VCFs.

## Evidence

- [other] VCFs from Soybase as anchors: [evidence span withheld — anchor status: not_found]
- [other] Allele-concordance anchoring: [evidence span withheld — anchor status: not_found]
- [other] Validation and flagging behavior: [evidence span withheld — anchor status: not_found]
- [intro] Downstream purpose of the converted coordinates: "we integrated multiple publicly available RNA-seq and GWAS datasets to identify high-confidence candidate genes for resistance to five phytopathogenic fungi"
