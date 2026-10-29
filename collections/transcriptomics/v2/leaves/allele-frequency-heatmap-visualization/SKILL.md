---
name: allele-frequency-heatmap-visualization
description: Use when you have vireoSNP clone labels and a set of MQuad ΔBIC-selected
  mtDNA variants and need to visually confirm clonal discrimination, e.g. reproducing
  Fig. 4c for MKN-45 scDNA-seq data.
when_to_use_negative:
- Input is raw per-cell genotype matrices without prior MQuad variant selection —
  noisy, non-clonal mtDNA SNVs will dominate the heatmap and obscure clonal structure
- The task is donor demultiplexing or allele-specific CNV analysis (e.g., with vireo
  alone, Numbat, XClone); the heatmap is specifically for validating clonal substructure
  via mitochondrial variants
- No clone labels exist yet — heatmap requires completed vireoSNP clone assignment;
  run cellsnp-lite → MQuad → vireoSNP first
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: cellsnp-lite
  role: Supplies the SNP-by-cell AD/DP pileup matrices whose allele frequencies are
    rendered in the heatmap
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Defines which of the piled-up variants are informative and appear as heatmap
    rows (ΔBIC/Kneedle selection)
  repo: https://github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: Provides the per-cell clone assignment that defines the heatmap columns and
    confusion-matrix grouping
  repo: https://github.com/single-cell-genetics/vireo
- name: CellRanger
  role: Source of CNV clone labels (with VireoSNP BAF clustering on chr2/chr4) used
    in the companion confusion matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:adae98498aab4e7f1c553286100cd44b3a40316ebea651929f180a7ca86ff4f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - CellRanger
  - MQuad
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# allele-frequency-heatmap-visualization

## Summary

Visualization step that renders the SNP-by-cell AD/DP pileup from cellsnp-lite, restricted to MQuad-selected clone-discriminative mtDNA variants, as an allele-frequency heatmap across vireoSNP-inferred mitochondrial clones (Fig. 4c), often paired with a CNV-clone vs mtDNA-clone confusion matrix (Fig. 4b).

## When to use

Use when you have vireoSNP clone labels and a set of MQuad ΔBIC-selected mtDNA variants and need to visually confirm clonal discrimination, e.g. reproducing Fig. 4c for MKN-45 scDNA-seq data.

## When NOT to use

- Input is raw per-cell genotype matrices without prior MQuad variant selection — noisy, non-clonal mtDNA SNVs will dominate the heatmap and obscure clonal structure
- The task is donor demultiplexing or allele-specific CNV analysis (e.g., with vireo alone, Numbat, XClone); the heatmap is specifically for validating clonal substructure via mitochondrial variants
- No clone labels exist yet — heatmap requires completed vireoSNP clone assignment; run cellsnp-lite → MQuad → vireoSNP first

## Inputs

- SNP-by-cell alternate-allele depth (AD) and total depth (DP) matrices from cellsnp-lite pileup of the hg38-aligned MKN-45 10x scDNA-seq BAM
- List of MQuad-selected clone-discriminative mtDNA variants (24 for MKN-45) with ΔBIC scores
- vireoSNP clone assignment per cell (5 mitochondrial clones; MT0..MT4)
- CNV clone labels per cell (CellRanger + VireoSNP BAF clustering on chr2 and chr4)

## Outputs

- Allele-frequency heatmap of clone-discriminative mtDNA variants across inferred mitochondrial clones (Fig. 4c analogue)
- Confusion/overlap matrix between CNV clones and mitochondrial clones (Fig. 4b analogue)
- Variant table reporting each informative variant's clone-level allele frequencies

## How to apply

After cellsnp-lite (v1.2.1) produces the SNP-by-cell AD/DP matrices and MQuad (v0.1.6) selects informative SNPs via a ΔBIC knee-point cutoff (Kneedle, kneed v0.7.0, excluding rCRS regions 302–315, 513–525, 3105–3109), restrict the AD and DP matrices to those variants. Compute per-variant allele frequencies either per cell or aggregated per vireoSNP (v0.5.3)-assigned clone, then plot a heatmap with variants as rows and mitochondrial clones as columns so clone-marking mutations stand out. For MKN-45, the finished figure should show clone MT0 highlighted by 2393C>T and 8368G>A, which are largely absent in the other 4 clones. Complement the heatmap with a confusion matrix crossing the two CellRanger/VireoSNP BAF-derived CNV clone labels (chr2, chr4) against the 5 mitochondrial clones to display overlap between CNV clone 1 and mitochondrial clone 0.

## Related tools

- **cellsnp-lite** (Supplies the SNP-by-cell AD/DP pileup matrices whose allele frequencies are rendered in the heatmap) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Defines which of the piled-up variants are informative and appear as heatmap rows (ΔBIC/Kneedle selection)) — https://github.com/single-cell-genetics/MQuad
- **vireoSNP** (Provides the per-cell clone assignment that defines the heatmap columns and confusion-matrix grouping) — https://github.com/single-cell-genetics/vireo
- **CellRanger** (Source of CNV clone labels (with VireoSNP BAF clustering on chr2/chr4) used in the companion confusion matrix)

## Examples

```
import pandas as pd, seaborn as sns; af = ad_df.loc[variants].groupby(assignment.clone).sum() / dp_df.loc[variants].groupby(assignment.clone).sum(); sns.heatmap(af, cmap='viridis')
```

## Evaluation signals

- Heatmap restricted to exactly the MQuad-selected variant set (24 variants for the MKN-45 reproduction) and 5 clone columns
- Allele frequencies lie in [0, 1] and equal per-clone AD sums divided by per-clone DP sums
- MT0 row entries for 2393C>T and 8368G>A show high allele frequency while the other 4 clones show near-zero frequency
- Confusion matrix shows high-degree overlap between CNV clone 1 and mitochondrial clone 0, matching Fig. 4b
- Variants in rCRS error-prone regions 302–315, 513–525, 3105–3109 are absent from the plotted set

## Limitations

- Interpretation depends on prior ΔBIC/Kneedle variant selection; including noisy or error-prone region variants (e.g., rCRS 302–315) creates false clone-marking signals in the heatmap
- CellRanger BAMs may not flag PCR duplicates reliably, so pileup depths (and thus displayed allele frequencies) can be affected; cellSNP/cellsnp-lite default to keeping duplicates
- CNV-derived subclones may be less dynamic than point-mutation-defined subclones, so heatmap-defined mtDNA clones and CNV clones need not correspond one-to-one

## Evidence

- [other] Plot the allele-frequency heatmap of the 24 clone-discriminative mtDNA variants across the inferred clones (Fig. 4c) and build the confusion matrix between CNV clones and mitochondrial clones (Fig. 4b).: [evidence span withheld — anchor status: not_found]
- [abstract] MQuad enables clonal substructure discovery using single cell mitochondrial variants: "MQuad enables clonal substructure discovery using single cell mitochondrial variants"
- [discussion] vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [readme] README (MQuad): use MQuad to differentiate informative mtDNA variants from noisy backbground; use vireoSNP to assign cells to clones based on mtDNA variant profile: "use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
- [intro] mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution: "our analysis reveals that mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution"
