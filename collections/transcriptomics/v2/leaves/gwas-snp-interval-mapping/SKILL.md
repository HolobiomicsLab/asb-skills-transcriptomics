---
name: gwas-snp-interval-mapping
description: Use when GWAS SNP results for a trait (e.g., soybean resistance to phytopathogenic
  fungi) need to be converted to a per-pathogen high-confidence candidate gene set
  by scanning a fixed genomic window around significant SNPs.
when_to_use_negative:
- Your GWAS study uses an organism or reference assemblies other than soybean Gmax_a1.v1/Gmax_a2.v1,
  so Soybase VCF-based coordinate conversion and the SoyFungiGCN coexpression modules
  do not apply
- You need de novo SNP association testing rather than post-GWAS candidate gene prioritization
  around already-significant SNPs
- You lack coexpression data or defense-related guide gene annotations, since cageminer
  scoring depends on both the 2 Mb SNP neighborhoods and rpb-based coexpression evidence
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SoyFungiGCN
  role: R package supplying the coexpression network (genes_modules) and the resource
    in which the SNP-to-candidate gene prioritization is performed
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Language used to parse Soybase VCF files for SNP coordinate conversion, define
    2 Mb candidate windows, and run cageminer prioritization
- name: remotes
  role: Used to install SoyFungiGCN from GitHub before running the prioritization
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:bd535fa0da7ff14380822d312d378329827fa5685868f7b79bc03fbe1721f70d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gwas-snp-interval-mapping

## Summary

Maps resistance-associated GWAS SNPs onto the soybean genome and enumerates/prioritizes candidate resistance genes within a fixed physical interval (2 Mb) around each significant SNP, integrating defense-related guide genes and coexpression evidence via the cageminer algorithm in the SoyFungiGCN framework. This bridges GWAS loci to a concrete, rankable candidate-gene list per pathogen.

## When to use

Use when GWAS SNP results for a trait (e.g., soybean resistance to phytopathogenic fungi) need to be converted to a per-pathogen high-confidence candidate gene set by scanning a fixed genomic window around significant SNPs.

## When NOT to use

- Your GWAS study uses an organism or reference assemblies other than soybean Gmax_a1.v1/Gmax_a2.v1, so Soybase VCF-based coordinate conversion and the SoyFungiGCN coexpression modules do not apply
- You need de novo SNP association testing rather than post-GWAS candidate gene prioritization around already-significant SNPs
- You lack coexpression data or defense-related guide gene annotations, since cageminer scoring depends on both the 2 Mb SNP neighborhoods and rpb-based coexpression evidence

## Inputs

- Curated list of resistance SNPs (chromosome, position, allele) reported on the Gmax_a1.v1 assembly
- Gmax_a1.v1 and Gmax_a2.v1 .vcf files from Soybase for coordinate conversion
- Coexpression module assignments (SoyFungiGCN genes_modules data object)
- MapMan defense gene annotations from PLAZA 3.0 Dicots (guide genes)
- Per-pathogen GWAS significance calls for the five fungal pathogens

## Outputs

- converted_snp_coordinates.tsv — SNP table with original Gmax_a1.v1 coordinates alongside mapped Gmax_a2.v1 chromosome/position and flags for unmappable SNPs
- candidate_counts.csv — number of high-confidence candidate resistance genes per pathogen (expected 188/56/11/8/3)
- Ranked list of high-confidence candidate resistance genes per fungal species

## How to apply

First harmonize SNP coordinates onto a single reference assembly: load curated SNPs (chromosome, position, allele) and map Gmax_a1.v1 positions to Gmax_a2.v1 using the .vcf files for both assemblies downloaded from Soybase, matching by allele concordance and flagging any SNP that cannot be uniquely mapped. Define the candidate search space as all genes lying within a 2 Mb window around each significant resistance SNP. Assign defense-related guide genes using MapMan annotations from PLAZA 3.0 Dicots, then run the cageminer prioritization separately for each of the five pathogens (F. virguliforme, F. graminearum, C. gregata, M. phaseolina, P. pachyrhizi) with an rpb threshold of 0.2, using RNA-seq-derived coexpression modules from SoyFungiGCN (e.g., genes_modules) to weight candidates by defense relevance. Tabulate high-confidence candidates per pathogen and compare against the expected counts of 188, 56, 11, 8, and 3.

## Related tools

- **SoyFungiGCN** (R package supplying the coexpression network (genes_modules) and the resource in which the SNP-to-candidate gene prioritization is performed) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Language used to parse Soybase VCF files for SNP coordinate conversion, define 2 Mb candidate windows, and run cageminer prioritization)
- **remotes** (Used to install SoyFungiGCN from GitHub before running the prioritization)

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN")
```

## Evaluation signals

- Reproduced high-confidence candidate counts per pathogen equal the reported 188, 56, 11, 8, and 3 for F. virguliforme, F. graminearum, C. gregata, M. phaseolina, and P. pachyrhizi, respectively
- converted_snp_coordinates.tsv exists, is a valid TSV with expected coordinate columns, and mapped rows show consistent REF/ALT alleles across the two assemblies
- SNPs that cannot be uniquely mapped between Gmax_a1.v1 and Gmax_a2.v1 are explicitly flagged rather than silently dropped
- Every candidate gene in the search space lies within the 2 Mb window of at least one significant resistance SNP
- Prioritization is run per pathogen with the specified rpb threshold of 0.2 (not a default or different cutoff)

## Limitations

- The SoyFungiGCN package is marked experimental in its README, so APIs and data objects may change without notice
- No changelog is available for the resource, complicating reproducibility checks across package versions
- Supplementary figures and tables referenced by the article are not included in the provided text, so some parameter provenance cannot be verified
- Coordinate conversion depends on allele concordance between Gmax_a1.v1 and Gmax_a2.v1 VCFs; SNPs that cannot be uniquely mapped must be flagged and excluded from downstream windows

## Evidence

- [abstract] Reported candidate counts validate the 2 Mb window + rpb 0.2 prioritization: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi, respectively"
- [abstract] Integration of GWAS and transcriptomic data is the core premise: "we integrated publicly available genome-wide association studies and transcriptomic data to prioritize candidate genes associated with resistance"
- [methods] Grounds the example invocation for obtaining the tool: "You can install the released version of SoyFungiGCN from GitHub with:"
- [readme] README confirms package purpose and experimental lifecycle status: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
