---
name: isoform-ambiguity-aware-colocalization-analysis
description: Use when region sets are defined only by genomic positions without clear
  isoform association (e.g., POSTAR3/RBP or RNA modification sites) and you need transcriptome-level
  association testing against picked regions like 5'UTR, CDS, or stop codons.
when_to_use_negative:
- Region sets already have clear transcript/isoform association and standard region-region
  permutation suffices — use permTestTx or permTestTx_customPick instead
- The analysis target is genome-based colocalization without transcriptome context
  — existing genome-based tools (GREAT, LOLA, GenomeRunner, BEDTools, regioneR) are
  the appropriate comparison point, and RgnTX's transcriptome null model would not
  match
- Input is a feature table without genomic coordinates, so sites cannot be projected
  onto isoforms
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3524
tools:
- name: RgnTX
  role: R/Bioconductor package providing permTestTxIA_customPick (IA-mode permutation
    test for isoform-ambiguous features), overlapCountsTxIA evaluator, customPick
    framework, and visualization of permutation results
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime environment in which RgnTX functions and TxDb annotation objects are
    used
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 transcript annotation supplying isoform structures (exons, CDS,
    UTRs) used for region picking and transcriptome-wide shuffling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:3b90529317c86a6275d1c123d42f0f261d5d58e084e0080cceb0271c5eec2b00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# isoform-ambiguity-aware-colocalization-analysis

## Summary

Performs colocalization analysis of transcriptome elements (e.g., RNA modification sites like m7G or m6A with unknown isoform origin) against regions of interest (e.g., 5'UTR, CDS, stop codons) using RgnTX's IA-mode permutation tests, which explicitly model isoform ambiguity and alternative splicing.

## When to use

Use when region sets are defined only by genomic positions without clear isoform association (e.g., POSTAR3/RBP or RNA modification sites) and you need transcriptome-level association testing against picked regions like 5'UTR, CDS, or stop codons.

## When NOT to use

- Region sets already have clear transcript/isoform association and standard region-region permutation suffices — use permTestTx or permTestTx_customPick instead
- The analysis target is genome-based colocalization without transcriptome context — existing genome-based tools (GREAT, LOLA, GenomeRunner, BEDTools, regioneR) are the appropriate comparison point, and RgnTX's transcriptome null model would not match
- Input is a feature table without genomic coordinates, so sites cannot be projected onto isoforms

## Inputs

- GRanges of feature sites with genomic positions but no isoform assignment (e.g., m6A_sites_data.rds from RgnTX extdata, m7G sites, POSTAR3 RBP sites)
- TxDb transcript annotation (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene)
- customPick function returning regions of interest (stop codons, CDS, 5'UTR) per transcript
- ntimes permutation count (e.g., 50-1000)

## Outputs

- permTestTxIA_customPick results object with observed_overlap, random_overlap_list (null distribution), p-value and z-score
- Summary-statistics plot of observed overlap vs. null distribution (e.g., fig_S2_summary_statistics.png reproducing Fig. S2)
- BH-adjusted p-values when multiple hypotheses are tested

## How to apply

Load the RgnTX package with a transcript annotation TxDb (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene) and read the feature region set (e.g., m7G or m6A sites shipped in extdata as RDS, or user-supplied GRanges without isoform labels). Define a customPick function (e.g., getStopCodon, getCDS, or a 5'UTR extractor built on fiveUTRsBy/transcriptsBy) that maps each input region's isoforms to the regions of interest. Run permTestTxIA_customPick with RS1 = the feature sites, txdb, customPick_function, ntimes (e.g., 50-1000 permutations), ev_function_1 = overlapCountsTxIA and ev_function_2 = overlapCountsTx; the IA evaluator averages overlaps over the K isoforms each site can align to. The function returns observed overlap, the null distribution of random overlaps (random_overlap_list), p-value and z-score; use plotPermResults/summary-statistics plots (as in Fig. S2) to compare observed overlap against the transcriptome-wide shuffling null, and apply Benjamini-Hochberg correction for multiple tests.

## Related tools

- **RgnTX** (R/Bioconductor package providing permTestTxIA_customPick (IA-mode permutation test for isoform-ambiguous features), overlapCountsTxIA evaluator, customPick framework, and visualization of permutation results) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime environment in which RgnTX functions and TxDb annotation objects are used)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 transcript annotation supplying isoform structures (exons, CDS, UTRs) used for region picking and transcriptome-wide shuffling)

## Examples

```
permTestTx_results <- permTestTxIA_customPick(RS1 = m7G_sites, txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, customPick_function = getFiveUTR, ntimes = 100, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- The returned object contains observed_overlap, random_overlap_list (length = ntimes), p-value and z-score; z-score should reflect observed overlap's position in the null distribution
- Randomization is transcriptome-wide: each randomized region must lie on the same transcript (or one of its isoforms) as the original site with matched width, verified by inspecting random_region_list
- For IA mode, per-site observed overlap is averaged over the K isoforms the site can align to (overlap/K), so counts need not be integers
- Plot of permutation results shows observed overlap plotted against the null histogram (Fig. S2-style summary statistics plot)
- When running multiple association tests, BH-adjusted p-values should be computed to control false positives

## Limitations

- Results differ from genome-based methods; RgnTX's novel testing modes return distinct and more significant results than genome-based approaches, so values are not directly comparable to GREAT/LOLA/GenomeRunner outputs
- No version number or R/dependency versions are stated for RgnTX; no changelog is available, so reproducibility requires recording package versions separately
- Requires the feature sites to be mappable to at least one annotated isoform in the TxDb; sites outside annotated transcripts cannot be tested

## Evidence

- [abstract] Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [results] RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives: "RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives"
- [discussion] RgnTX provides randomization functions that can shuffle regions within a transcriptome background. It allows for the direct sampling of regions over mRNA space.: "RgnTX provides randomization functions that can shuffle regions within a transcriptome background. It allows for the direct sampling of regions over mRNA space."
- [readme] The function permTestTxIA_customPick performs permutation test to evaluate association between genomic features (RS) that one only know its genomic positions and do not know which specific transcript: [evidence span withheld — anchor status: external_doc]
- [readme] permTestTx_results <- permTestTxIA_customPick(RS1 = RS1, txdb = txdb, customPick_function = getStopCodon, ntimes = 50,: "permTestTx_results <- permTestTxIA_customPick(RS1 = RS1,
                                       txdb = txdb,
                                       customPick_function = getStopCodon"
