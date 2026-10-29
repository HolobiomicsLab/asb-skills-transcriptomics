---
name: clone-cnv-concordance-analysis
description: Use when you have mtDNA clone assignments (e.g., MQuad/vireoSNP output
  with cell-level clone labels) plus independent annotations such as CNV subclones
  or tumor/normal labels, and need to test whether the mitochondrial clones recapitulate
  or refine the known clonal structure.
when_to_use_negative:
- The goal is donor deconvolution of multiplexed scRNA-seq from common nuclear SNPs
  rather than within-sample mtDNA clonal substructure.
- No independent annotations (CNV subclones or tumor labels) exist, so the concordance/enrichment
  tests cannot be computed and only clonal assignment is needed.
- The data are not single-cell (e.g., bulk DNA/RNA), or the assay is scATAC-seq where
  mtDNA-specific callers like mgatk or EMBLEM were the primary design targets.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: cellsnp-lite
  role: Piles up mtDNA reads from the 10x BAM into SNP-by-cell AD/DP matrices (v1.2.1)
    as input to variant informativeness testing
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Selects clonally informative mtDNA variants via a binomial mixture model with
    ΔBIC cutoffs set at the Kneedle knee point (v0.1.6)
  repo: https://github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: Assigns individual cells to mitochondrial clones by variational inference
    on the MQuad-selected SNPs, with a probability cutoff of 0.8 (v0.5.3)
  repo: https://github.com/single-cell-genetics/vireo
- name: kneed
  role: Determines the ΔBIC cutoff for informative-SNP selection at the knee point
    (v0.7.0)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:99bb75de22509751e3405c9fa6f1de5d20b12d786fa91ed6b6fbc48cd105fef4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - MQuad
  - cellsnp-lite
  - kneed
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clone-cnv-concordance-analysis

## Summary

Assigns single cells to mitochondrial clones using the cellSNP-MQuad-VireoSNP pipeline on 10x scRNA-seq BAMs, then cross-validates the mtDNA clones against independent tumor cell labels and CNV-defined subclones via two-sided Fisher's exact tests and confusion matrices.

## When to use

Use when you have mtDNA clone assignments (e.g., MQuad/vireoSNP output with cell-level clone labels) plus independent annotations such as CNV subclones or tumor/normal labels, and need to test whether the mitochondrial clones recapitulate or refine the known clonal structure.

## When NOT to use

- The goal is donor deconvolution of multiplexed scRNA-seq from common nuclear SNPs rather than within-sample mtDNA clonal substructure.
- No independent annotations (CNV subclones or tumor labels) exist, so the concordance/enrichment tests cannot be computed and only clonal assignment is needed.
- The data are not single-cell (e.g., bulk DNA/RNA), or the assay is scATAC-seq where mtDNA-specific callers like mgatk or EMBLEM were the primary design targets.

## Inputs

- Analysis-ready single-cell BAM file (10x Genomics scRNA-seq, aligned to hg38) with cell barcodes
- Cell barcode list corresponding to the BAM
- CNV subclone labels per cell from an independent copy-number analysis (e.g., the original TNBC study)
- Tumor/normal cell-type labels per cell

## Outputs

- SNP-by-cell AD/DP pileup matrices from cellsnp-lite
- List of informative mtDNA variants with ΔBIC scores from MQuad
- Cell-to-clone assignments with probabilities from vireoSNP (filtered at >0.8)
- Two-sided Fisher's exact test p-value for clone-1 tumor enrichment
- Confusion/contingency matrix between mtDNA clones and CNV subclones

## How to apply

Pile up mitochondrial variants from the analysis-ready 10x BAM (aligned to hg38) with cellsnp-lite v1.2.1 to produce SNP-by-cell AD/DP matrices, then fit MQuad v0.1.6's binomial mixture model per variant and select informative SNPs using the ΔBIC cutoff at the Kneedle (kneed v0.7.0) knee point, explicitly excluding error-prone rCRS regions 302–315, 513–525 and 3105–3109. Assign cells to clones with vireoSNP v0.5.3 variational inference, retaining only cells with assignment probability >0.8 to avoid noisy labels. Build a 2x2 contingency table of mitochondrial clone 1 vs clone 0 against tumor/normal labels and run a two-sided Fisher's exact test, then tabulate a confusion matrix between confidently assigned mtDNA clones and independently derived CNV subclones. Rationale: mtDNA mutations detected by MQuad are meant to complement read-coverage-inferred CNVs, since CNV-based subclonal structure may be less dynamic than subclones defined by propagation of true point mutations, so near-exclusive overlap (e.g., mtDNA clone 1 with CNV subclone B) indicates a genuinely finer subclone nested within a CNV clone.

## Related tools

- **cellsnp-lite** (Piles up mtDNA reads from the 10x BAM into SNP-by-cell AD/DP matrices (v1.2.1) as input to variant informativeness testing) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Selects clonally informative mtDNA variants via a binomial mixture model with ΔBIC cutoffs set at the Kneedle knee point (v0.1.6)) — https://github.com/single-cell-genetics/MQuad
- **vireoSNP** (Assigns individual cells to mitochondrial clones by variational inference on the MQuad-selected SNPs, with a probability cutoff of 0.8 (v0.5.3)) — https://github.com/single-cell-genetics/vireo
- **kneed** (Determines the ΔBIC cutoff for informative-SNP selection at the knee point (v0.7.0))

## Examples

```
from scipy.stats import fisher_exact; odds_ratio, p_value = fisher_exact([[clone1_tumor, clone1_normal], [clone0_tumor, clone0_normal]], alternative='two-sided')
```

## Evaluation signals

- Exactly two informative mtDNA variants are detected in TNBC1 by MQuad
- 1097 of 1146 cells (95.8%) receive clone assignments with probability >0.8
- Two-sided Fisher's exact test for mtDNA clone 1 enrichment among tumor cells yields p ≈ 6e-5
- Confusion matrix shows mitochondrial clone 1 overlapping almost exclusively with CNV subclone B, evidencing a small subclone nested within a CNV clone
- Filter steps include removal of known error-prone rCRS regions 302–315, 513–525, 3105–3109 before clone calling

## Limitations

- Subclonal structures inferred from read-coverage-inferred CNVs might not be as dynamic as subclones shown by the propagation of true point mutations, so concordance is expected to be partial rather than perfect.
- Confounding by sequencing errors and uninformative/noisy mtDNA SNVs can distort clonal inference; high-confidence assignment (>0.8 probability) is used to mitigate this.
- Nuclear SNV callers assume a diploid context that is violated in the mitochondrial genome, so generic variant callers are not a substitute in this workflow.

## Evidence

- [discussion] MQuad variant selection: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] vireoSNP clone reconstruction: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [intro] Complementarity with CNVs: "our analysis reveals that mtDNA mutations detected by MQuad can be used in complement with nuclear SNVs and CNVs to achieve finer clonal resolution."
- [discussion] False positive region masking: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [readme] Pipeline assignment step: "use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
