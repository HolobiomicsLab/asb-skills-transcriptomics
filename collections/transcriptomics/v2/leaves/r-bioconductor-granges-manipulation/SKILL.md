---
name: r-bioconductor-granges-manipulation
description: Use when region sets are genomic features (e.g. m6A sites, stop codon
  regions) that must be shifted, randomized or overlap-counted over isoform/exon structures
  in R for transcriptome colocalization analysis.
when_to_use_negative:
- Regions are plain genome-based intervals with no transcriptome/isoform context —
  existing genome-based approaches (BEDTools, LOLA, GREAT) are the article's stated
  contrast case
- Every element already has a clear, unambiguous isoform association and you only
  need standard overlapCountsTx counting rather than the isoform-ambiguity (IA) weighting
- No TxDb transcriptome annotation is available for the genome build of the regions
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RgnTX
  role: Provides the GRanges-manipulation functions (shiftExonTx, randomizeTx) and
    overlap evaluators (overlapCountsTx, overlapCountsTxIA) used to shift, randomize
    and count transcriptomic regions
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime environment in which the GRanges/GRangesList objects are constructed,
    the custom IA overlap score is implemented, and outputs are written to CSV
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: Supplies the UCSC hg19 transcriptome annotation (exons, CDS, transcript ids)
    from which GRangesList backgrounds are extracted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:4016ed6ebd183c6837cb358426623d67302dcd03e7261e07c9ca10209a37622c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# r-bioconductor-granges-manipulation

## Summary

Manipulate transcriptomic region sets as GRanges/GRangesList objects in R using RgnTX's helpers (exonsBy over TxDb.Hsapiens.UCSC.hg19.knownGene, shiftExonTx, randomizeTx) and quantify overlaps via overlapCountsTx / overlapCountsTxIA for colocalization analysis.

## When to use

Use when region sets are genomic features (e.g. m6A sites, stop codon regions) that must be shifted, randomized or overlap-counted over isoform/exon structures in R for transcriptome colocalization analysis.

## When NOT to use

- Regions are plain genome-based intervals with no transcriptome/isoform context — existing genome-based approaches (BEDTools, LOLA, GREAT) are the article's stated contrast case
- Every element already has a clear, unambiguous isoform association and you only need standard overlapCountsTx counting rather than the isoform-ambiguity (IA) weighting
- No TxDb transcriptome annotation is available for the genome build of the regions

## Inputs

- GRanges/GRangesList region sets (e.g. exonsBy(txdb) transcript exons, m6A site GRanges)
- TxDb annotation object (TxDb.Hsapiens.UCSC.hg19.knownGene)
- Transcript ids (knownGene names, e.g. '170', '1387', '4113', '10715')
- start/width vectors for region shifting over mRNA space
- Custom isoform-picking function (e.g. getStopCodon) returning per-isoform ROI

## Outputs

- Shifted GRangesList regions (shiftExonTx output spanning multiple exons)
- Randomized region sets of random_num regions with specified random_length
- Per-feature weighted overlap scores (c(i)/n(i) ratios in [0,1])
- Comparison table written to ia_overlap_verification.csv
- Permutation test outputs: observed_overlap, random_overlap_list, pvalue, zscore

## How to apply

Load RgnTX together with TxDb.Hsapiens.UCSC.hg19.knownGene and build GRangesList structures with exonsBy(txdb), indexing transcripts by their knownGene ids (e.g. '170', '1387'). Shift regions over mRNA space with shiftExonTx(regions, start, width), where a positive width moves the start rightward (3') and a negative width leftward (5') across multiple exons. Randomize regions within a transcriptome background with randomizeTx(txdb, trans_ids, random_num, random_length, type='mature') to simulate realistic alternative splicing backgrounds. For features with isoform ambiguity (no clear transcript association), compute the weighted overlap as a per-feature ratio c(i)/n(i) — numerator the count of overlaps with the transcriptome ROI, denominator the number of associated isoforms K — clamped to [0, 1], and validate the custom implementation against RgnTX's overlapCountsTxIA outputs within numerical tolerance. Write per-feature comparison tables (feature, custom IA score, overlapCountsTxIA score, c(i)/n(i) ratio) to CSV (e.g. ia_overlap_verification.csv) so the equality assertion is auditable.

## Related tools

- **RgnTX** (Provides the GRanges-manipulation functions (shiftExonTx, randomizeTx) and overlap evaluators (overlapCountsTx, overlapCountsTxIA) used to shift, randomize and count transcriptomic regions) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime environment in which the GRanges/GRangesList objects are constructed, the custom IA overlap score is implemented, and outputs are written to CSV)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (Supplies the UCSC hg19 transcriptome annotation (exons, CDS, transcript ids) from which GRangesList backgrounds are extracted)

## Examples

```
txdb <- TxDb.Hsapiens.UCSC.hg19.knownGene; exons.tx0 <- exonsBy(txdb); shifted_regions <- shiftExonTx(exons.tx0[c("170","1387")], c(3624255, 55158197), c(200, -200))
```

## Evaluation signals

- Every per-feature c(i)/n(i) ratio lies within the interval [0, 1]
- Custom IA overlap scores match RgnTX overlapCountsTxIA outputs within numerical tolerance for every feature on a synthetic feature set
- shiftExonTx output widths match the requested width split across exons (e.g. width 200 over one transcript returned intervals of 101 and 99 in the README example)
- randomizeTx returns exactly random_num regions, each of the specified random_length, sampled from the given trans_ids
- Comparison table (feature, custom IA score, overlapCountsTxIA score, c(i)/n(i) ratio) is written to ia_overlap_verification.csv and non-empty

## Limitations

- Overlapping counts differ significantly between genome-based and transcriptome-based backgrounds (DNA, exonic DNA, pre-mRNA, mRNA), so scores are not interchangeable across backgrounds
- Isoform ambiguity is inherent to sources like POSTAR3 RBP target sites and miCLIP-seq m6A data, where the specific isoform of origin is unknown; the IA weighting (division by K isoforms) is a modeled approximation
- No R version or dependency versions are specified in the article, and no changelog is available for RgnTX

## Evidence

- [abstract] RgnTX as R/Bioconductor GRanges-based colocalization tool: "Here, we developed RgnTX, an R/Bioconductor tool for the colocalization analysis of transcriptome elements with permutation tests."
- [readme] shiftExonTx shifts GRanges over exon structure: [evidence span withheld — anchor status: external_doc]
- [readme] weighted IA overlap formula per feature: [evidence span withheld — anchor status: external_doc]
- [discussion] randomization within transcriptome background: "RgnTX provides randomization functions that can shuffle regions within a transcriptome background."
- [results] overlap counts differ across backgrounds: "Although all the backgrounds were restricted to only 300 genes, the overlapping counts in each background were significantly different"
