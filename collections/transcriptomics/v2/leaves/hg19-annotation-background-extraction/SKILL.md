---
name: hg19-annotation-background-extraction
description: Use when a permutation-based colocalization or randomization analysis
  in RgnTX/regioneR needs matched genome and transcriptome background region sets
  built from hg19 annotation for a defined gene set.
when_to_use_negative:
- Your elements are defined on a genome assembly other than hg19 (e.g. hg38) — the
  TxDb.Hsapiens.UCSC.hg19.knownGene coordinates will not match
- You only have a ready-made feature table or BED file of observed regions and no
  need to build matched genome/transcriptome backgrounds
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0085
tools:
- name: RgnTX
  role: Provides randomizeTx and overlap counting functions for extracting and using
    transcriptome-type backgrounds (mature, full, fiveUTR, CDS, threeUTR) from the
    txdb annotation
  repo: https://bioconductor.org/packages/RgnTX
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: Supplies the UCSC hg19 transcript/exon/CDS annotation from which the DNA,
    exonic DNA, pre-mRNA and mRNA backgrounds are extracted
- name: R
  role: Execution environment for loading the annotation and running the background
    extraction
- name: regioneR
  role: Used downstream (numOverlaps) to count overlaps on the genome-based backgrounds
    built from the annotation
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:13f759e75b3e1d429e7d41ac8c4248b19f5aa23f4486364e260b152c334a8247
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  - regioneR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hg19-annotation-background-extraction

## Summary

Extracts genome- and transcriptome-based background region sets (DNA, exonic DNA, pre-mRNA, mRNA) from the UCSC hg19 annotation database (TxDb.Hsapiens.UCSC.hg19.knownGene) restricted to a defined gene set, as the substrate for RgnTX permutation-based colocalization analysis.

## When to use

Use when a permutation-based colocalization or randomization analysis in RgnTX/regioneR needs matched genome and transcriptome background region sets built from hg19 annotation for a defined gene set.

## When NOT to use

- Your elements are defined on a genome assembly other than hg19 (e.g. hg38) — the TxDb.Hsapiens.UCSC.hg19.knownGene coordinates will not match
- You only have a ready-made feature table or BED file of observed regions and no need to build matched genome/transcriptome backgrounds

## Inputs

- TxDb.Hsapiens.UCSC.hg19.knownGene annotation database (R/Bioconductor txdb object)
- A defined gene or transcript set (e.g. 300 randomly picked hg19 genes, or transcript IDs such as '170', '782', '974', '1364', '1387')
- Random seed value for reproducibility (paper used set.seed(12345677))

## Outputs

- Four matched background region sets over the same gene set: DNA, exonic DNA, pre-mRNA, and mRNA (GRanges/GRangesList objects)
- Transcriptome region sets by type (mature/full/fiveUTR/CDS/threeUTR) usable as randomization backgrounds in randomizeTx

## How to apply

Load RgnTX together with TxDb.Hsapiens.UCSC.hg19.knownGene in R and fix the random seed (e.g. set.seed(12345677)) for reproducibility. Randomly pick the gene set of interest (in the paper, 300 randomly picked hg19 genes) and use the UCSC annotation to extract four matched backgrounds: whole DNA, exonic DNA, pre-mRNA, and mRNA, all restricted to the same genes so null distributions are comparable across spaces. Transcriptome-type regions (e.g. mature mRNA, CDS, 5'UTR, 3'UTR) can be retrieved via RgnTX functions such as randomizeTx with the 'type' argument set to 'mature', 'full', 'fiveUTR', 'CDS', or 'threeUTR' over specified transcript IDs. Validate the extraction by confirming all backgrounds derive from the identical gene set and that downstream overlap counting (regioneR::numOverlaps for genome spaces, RgnTX::overlapCountsTx for the transcriptome space) yields distinguishable null distributions.

## Related tools

- **RgnTX** (Provides randomizeTx and overlap counting functions for extracting and using transcriptome-type backgrounds (mature, full, fiveUTR, CDS, threeUTR) from the txdb annotation) — https://bioconductor.org/packages/RgnTX
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (Supplies the UCSC hg19 transcript/exon/CDS annotation from which the DNA, exonic DNA, pre-mRNA and mRNA backgrounds are extracted)
- **R** (Execution environment for loading the annotation and running the background extraction)
- **regioneR** (Used downstream (numOverlaps) to count overlaps on the genome-based backgrounds built from the annotation)

## Examples

```
library(TxDb.Hsapiens.UCSC.hg19.knownGene); txdb <- TxDb.Hsapiens.UCSC.hg19.knownGene; randomResults <- randomizeTx(txdb = txdb, trans_ids = c("170","782","974","1364","1387"), random_num = 10, type = "mature", random_length = 100)
```

## Evaluation signals

- All four backgrounds (DNA, exonic DNA, pre-mRNA, mRNA) are derived from exactly the same randomly picked gene set (e.g. 300 hg19 genes), so null distributions are directly comparable
- Extracted regions carry hg19 seqinfo consistent with TxDb.Hsapiens.UCSC.hg19.knownGene (chromosomes chr1–chrX, correct strands)
- Randomizing region sets of 500 equal-length (200 nt) regions over each background and counting overlaps produces five distinguishable null distributions, with fewest overlaps on the transcriptome space
- Repeated extraction with the same seed (set.seed(12345677)) reproduces identical background region sets

## Limitations

- Tied to the hg19 UCSC knownGene annotation; coordinates are not portable to other genome builds
- Null distributions differ significantly across permutation spaces, so backgrounds extracted this way cannot be interchanged between genome-based and transcriptome-based tests
- No R version or dependency versions are specified in the article or README, so environment reproducibility must be managed by the practitioner

## Evidence

- [results] All the backgrounds were region sets related to the same (randomly picked) 300 genes in hg19: "All the backgrounds were region sets related to the same (randomly picked) 300 genes in hg19"
- [results] We used the annotation database from UCSC to extract the following backgrounds: "We used the annotation database from UCSC to extract the following backgrounds"
- [readme] randomizeTx picks regions over specified transcripts with type mature/full/fiveUTR/CDS/threeUTR: "IF type is one from "mature"/"full"/"fiveUTR"/"CDS"/"threeUTR"
               region_i <- Pick corresponding type of region from txdb that has transcript id trans_id_i"
- [results] Although all the backgrounds were restricted to only 300 genes, the overlapping counts in each background were significantly different: "Although all the backgrounds were restricted to only 300 genes, the overlapping counts in each background were significantly different"
- [discussion] RgnTX provides randomization functions that can shuffle regions within a transcriptome background: "RgnTX provides randomization functions that can shuffle regions within a transcriptome background. It allows for the direct sampling of regions over mRNA space."
