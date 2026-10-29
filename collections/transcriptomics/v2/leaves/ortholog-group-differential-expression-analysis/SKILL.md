---
name: ortholog-group-differential-expression-analysis
description: Use when differential expression has been called on ortholog-group IDs
  (e.g. via Seq2Fun/ExpressAnalyst for species lacking a reference transcriptome)
  and you must validate whether the significant DEG list is inflated by reads from
  one gene mapping to different ortholog IDs in different species.
when_to_use_negative:
- Input is already gene-level (not ortholog-level) DE results from a species with
  a well-annotated reference transcriptome — cross-species ortholog duplication is
  not the relevant concern
- You intend to use the default stricter cutoff (FDR < 0.05 and |log fold change|
  > 1.5) when reproducing the salamander study, which explicitly used FDR-adjusted
  p < 0.1
- The taxonomic subgroup database was chosen improperly (e.g. reads mapped to the
  wrong phylum's database), since key functional groups may be missed and duplication
  audit would be meaningless
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalyst
  role: Web platform that performs the ortholog-group-level differential expression
    and functional analysis whose significant results are audited for duplication
  repo: github.com/xia-lab/ExpressAnalystR
- name: ExpressAnalystR
  role: R package synchronized with the web server; supplies DE analysis functions
    (annotation, normalization, PerformDEAnal) used to produce ortholog-level results
    reproducibly
  repo: github.com/xia-lab/ExpressAnalystR
- name: Seq2Fun
  role: Maps raw RNA-seq reads from non-model species to KEGG ortholog (KO) databases
    via translated search, generating the ortholog-level counts analyzed here
- name: EcoOmicsDB
  role: Ortholog database backing the KO-level quantification across taxonomic subgroups
    for species without reference transcriptomes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:1afe8fdecc87c4d3d29a2c22fafab02764782ea8e6bcaff3182707e879a6ec30
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - ExpressAnalystR
  - Seq2Fun
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ortholog-group-differential-expression-analysis

## Summary

Differential expression analysis performed at KEGG ortholog (KO) group level for non-model species, followed by an audit of how many significant hits are duplicated ortholog pairs (reads from the same gene mapping to similar but distinct orthologs across species). This skill tests whether ortholog-level DEG counts are inflated, e.g. confirming that duplicated DEGs account for only ~3% of significant salamander results.

## When to use

Use when differential expression has been called on ortholog-group IDs (e.g. via Seq2Fun/ExpressAnalyst for species lacking a reference transcriptome) and you must validate whether the significant DEG list is inflated by reads from one gene mapping to different ortholog IDs in different species.

## When NOT to use

- Input is already gene-level (not ortholog-level) DE results from a species with a well-annotated reference transcriptome — cross-species ortholog duplication is not the relevant concern
- You intend to use the default stricter cutoff (FDR < 0.05 and |log fold change| > 1.5) when reproducing the salamander study, which explicitly used FDR-adjusted p < 0.1
- The taxonomic subgroup database was chosen improperly (e.g. reads mapped to the wrong phylum's database), since key functional groups may be missed and duplication audit would be meaningless

## Inputs

- Ortholog-level differential expression results table with per-species FDR-adjusted p-values and expression direction per ortholog group (e.g. KO IDs from Seq2Fun/Kallisto quantification in ExpressAnalyst)
- Salient metadata of the case study species set (e.g. salamander RNA-seq reads from NCBI SRA SRR7499348–SRR7499365 mapped to EcoOmicsDB ortholog databases)
- DE significance threshold definition from the original publication (FDR-adjusted p < 0.1 for salamanders)

## Outputs

- duplicated_ortholog_pairs.csv — table of similar ortholog pairs where both members are DE at FDR < 0.1 with distinct species-specific patterns
- Single scalar: fraction of significant DE results attributable to duplicated ortholog pairs (~0.03 for the salamander case study)
- Filtered significant-DEG ortholog list used as the denominator

## How to apply

Start from the ortholog-level differential expression results table containing per-species FDR-adjusted p-values and expression directions, generated with ExpressAnalyst after quantification (Seq2Fun or Kallisto v0.46.1) against the appropriate EcoOmicsDB taxonomic subgroup. Filter ortholog groups to significant DEGs using the threshold from the original publication — FDR-adjusted p-value < 0.1 for the salamander case study (stricter studies may use FDR < 0.05 with |log2 fold change| > 1.5, excluding QC outliers such as sample SRR7499350). Among significant groups, detect pairs of similar ortholog groups where BOTH members pass the DE threshold but show distinct species-specific patterns (one member significant in one species, the other in a different species); flag these as duplicated DEGs. Write the flagged pairs to duplicated_ortholog_pairs.csv, then compute the single fraction of significant results accounted for by duplicated pairs and report it as one number. The rationale is that ExpressAnalyst's read-to-ortholog alignment can legitimately map reads from the same gene to similar but different orthologs per species, so duplication must be quantified rather than assumed negligible.

## Related tools

- **ExpressAnalyst** (Web platform that performs the ortholog-group-level differential expression and functional analysis whose significant results are audited for duplication) — github.com/xia-lab/ExpressAnalystR
- **ExpressAnalystR** (R package synchronized with the web server; supplies DE analysis functions (annotation, normalization, PerformDEAnal) used to produce ortholog-level results reproducibly) — github.com/xia-lab/ExpressAnalystR
- **Seq2Fun** (Maps raw RNA-seq reads from non-model species to KEGG ortholog (KO) databases via translated search, generating the ortholog-level counts analyzed here)
- **EcoOmicsDB** (Ortholog database backing the KO-level quantification across taxonomic subgroups for species without reference transcriptomes)

## Examples

```
dataSets <- PerformDEAnal("estrogen.txt", "custom", "absent vs. present", "NA", "intonly"); dataSet <- readDataset("estrogen.txt"); print(head(dataSet$comp.res));
```

## Evaluation signals

- The computed duplicated-DEG fraction approximates the reported ~3% of significant results in the salamander case study
- Every row of duplicated_ortholog_pairs.csv has both members significant at FDR-adjusted p < 0.1 in at least one species each
- Flagged pairs show distinct species-specific patterns: one member significant in one species, the other in a different species (not co-significant in the same species with the same direction)
- The denominator of the fraction equals the count of ortholog groups passing FDR < 0.1 after QA/QC exclusions (e.g. outlier sample SRR7499350 removed per the original publication)
- ExpressAnalystR DE results reproduce web-server output when the saved R command history is replayed locally

## Limitations

- Reads from the same gene may be mapped to similar but different orthologs for different species, requiring care when interpreting results for taxonomically diverse comparisons
- KO-based annotation has limited transcriptome coverage (e.g. only 76.16% of human protein-coding genes have KOs; vitellogenin is absent from the KO system), so the ortholog-level DE table is incomplete
- Choosing k for ortholog clustering involves an inherent tradeoff between grouping distinct functional proteins and grouping across species, which affects which ortholog pairs can appear duplicated

## Evidence

- [discussion] salamander DEG threshold and original-publication cutoff: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [discussion] cross-species ortholog mapping limitation motivating the duplication audit: [evidence span withheld — anchor status: not_found]
- [readme] ExpressAnalystR provides the DE machinery behind the ortholog-level results: "The R package is composed of R functions necessary for the web-server to perform data annotation, normalization, differential expression and meta-analysis."
