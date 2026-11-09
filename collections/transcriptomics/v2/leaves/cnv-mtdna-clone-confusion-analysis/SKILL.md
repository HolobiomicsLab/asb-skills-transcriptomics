---
name: cnv-mtdna-clone-confusion-analysis
description: Use when single-cell data have both mtDNA-variant clone labels (e.g.,
  from vireoSNP on MQuad SNPs) and CNV-based clone labels (e.g., CellRanger/VireoSNP
  BAF clustering) and you need to quantify concordance or finer subclonal resolution.
when_to_use_negative:
- The dataset has no independent CNV or nuclear clone labels to compare against mtDNA
  clones, so a confusion matrix would be uninformative
- CNV subclones are expected to be the target resolution themselves — coverage-inferred
  CNV subclones may be less dynamic than subclones shown by propagation of true point
  mutations, so mtDNA clones should not be forced to match CNV clones
- Only a handful of cells carry both label types, making cross-tabulation counts too
  sparse to interpret
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3173
tools:
- name: vireoSNP
  role: Assigns cells to mitochondrial clones via variational inference and performs
    BAF clustering for CNV clone labels
  repo: https://github.com/single-cell-genetics/vireo
- name: MQuad
  role: Selects clonally informative mtDNA variants (ΔBIC with Kneedle knee-point
    cutoff) used as features for the mtDNA clone labels
  repo: https://github.com/single-cell-genetics/MQuad
- name: cellsnp-lite
  role: Piles up mtDNA reads from BAM into SNP-by-cell AD/DP matrices feeding MQuad
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: CellRanger
  role: Provides the CNV profile of the MKN-45 scDNA-seq data used as one source of
    CNV clone labels
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:64f92665806f5a5e24fae4d488b725ba35cd76f3ad8821554d58e29cf5332a3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/vireo
  tools:
  - Cardelino
  - CellRanger
  - MQuad
  - Picard
  - STAR
  - cellsnp-lite
  - kneed
  - vireoSNP
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cnv-mtdna-clone-confusion-analysis

## Summary

Cross-modality validation of mtDNA-inferred clones against nuclear copy-number-variation clones by building a confusion matrix and checking overlap (e.g., CNV clone 1 with mitochondrial clone 0 in MKN-45), demonstrating that mtDNA variants complement CNV-based clonal structure.

## When to use

Use when single-cell data have both mtDNA-variant clone labels (e.g., from vireoSNP on MQuad SNPs) and CNV-based clone labels (e.g., CellRanger/VireoSNP BAF clustering) and you need to quantify concordance or finer subclonal resolution.

## When NOT to use

- The dataset has no independent CNV or nuclear clone labels to compare against mtDNA clones, so a confusion matrix would be uninformative
- CNV subclones are expected to be the target resolution themselves — coverage-inferred CNV subclones may be less dynamic than subclones shown by propagation of true point mutations, so mtDNA clones should not be forced to match CNV clones
- Only a handful of cells carry both label types, making cross-tabulation counts too sparse to interpret

## Inputs

- Per-cell mitochondrial clone assignments from vireoSNP on MQuad-selected mtDNA SNPs
- Per-cell CNV clone labels from CellRanger CNV profile and/or VireoSNP BAF clustering on specific chromosomes (chr2, chr4 for MKN-45)
- SNP-by-cell AD/DP matrices from cellsnp-lite for computing clone-level allele frequencies of informative variants

## Outputs

- Confusion matrix (cross-tabulation) between CNV clones and mitochondrial clones (Fig. 4b equivalent)
- Allele-frequency heatmap of clone-discriminative mtDNA variants across inferred clones (Fig. 4c equivalent)
- Variant table reporting clone-level allele frequencies per informative variant

## How to apply

After vireoSNP assigns cells to mitochondrial clones using MQuad-selected informative mtDNA SNPs, import independently derived CNV clone labels — for MKN-45 these come from CellRanger CNV profiling and VireoSNP-based B-allele-frequency (BAF) clustering on chr2 and chr4. Cross-tabulate the two label sets per cell to build a confusion matrix, and compute the allele-frequency heatmap of the clone-discriminative mtDNA variants per clone (e.g., the 24 informative variants × 5 clones in MKN-45, Fig. 4c). Interpret high-diagonal blocks as concordant structure (CNV clone 1 highly overlapping mitochondrial clone 0) and off-diagonal splits as evidence that mtDNA resolves subclones CNVs miss. Validate against reported counts: 24 informative variants, 5 mtDNA clones, and high overlap between CNV clone 1 and mtDNA clone 0.

## Related tools

- **vireoSNP** (Assigns cells to mitochondrial clones via variational inference and performs BAF clustering for CNV clone labels) — https://github.com/single-cell-genetics/vireo
- **MQuad** (Selects clonally informative mtDNA variants (ΔBIC with Kneedle knee-point cutoff) used as features for the mtDNA clone labels) — https://github.com/single-cell-genetics/MQuad
- **cellsnp-lite** (Piles up mtDNA reads from BAM into SNP-by-cell AD/DP matrices feeding MQuad) — https://github.com/single-cell-genetics/cellsnp-lite
- **CellRanger** (Provides the CNV profile of the MKN-45 scDNA-seq data used as one source of CNV clone labels)

## Examples

```
vireo -c cellsnp_out/Cell.barcode.txt -N cellsnp_out --NM 4 --out vireo_out; python -c "import pandas as pd; mt=pd.read_csv('vireo_out/clone_labels.tsv',sep='\t'); cnv=pd.read_csv('cnv_clone_labels.tsv',sep='\t'); print(pd.crosstab(cnv.cnv_clone,mt.donorID))"
```

## Evaluation signals

- Pipeline recovers the expected counts of informative variants and mtDNA clones (e.g., 24 variants and 5 clones for MKN-45)
- Confusion matrix shows high-degree overlap between CNV clone 1 and mitochondrial clone 0, matching Fig. 4b
- Variant table confirms MT0 carries 2393C>T and 8368G>A, largely absent in all other mitochondrial clones
- Allele-frequency heatmap shows clone-specific variants separating the inferred mitochondrial clones (Fig. 4c pattern)

## Limitations

- Subclonal structures inferred from read-coverage-inferred CNVs might not be as dynamic as subclones shown by the propagation of true point mutations, so discordance can reflect method differences rather than error
- MKN-45 scDNA-seq is low-coverage, limiting the number of detectable informative mtDNA variants
- CNV clone labels depend on the chromosomes chosen for BAF clustering (chr2 and chr4 in MKN-45); different chromosome choices may yield different clone assignments

## Evidence

- [intro] mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution: "our analysis reveals that mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution."
- [discussion] MKN-45 CNV profile via CellRanger: "For the MKN-45 scDNA-seq dataset, we first explored the copy number variation profile by CellRanger"
- [discussion] vireoSNP reconstructs clonal populations: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [discussion] MQuad selects informative variants with binomial mixture model: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] cellsnp-lite piles up raw reads from BAM files: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
