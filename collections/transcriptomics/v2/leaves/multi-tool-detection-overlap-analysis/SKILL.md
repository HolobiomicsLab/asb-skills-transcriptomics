---
name: multi-tool-detection-overlap-analysis
description: Use when benchmarking circRNA/BSJ detection tools on identical ribo−
  RNA-seq FASTQ inputs and you need pairwise/three-way call-set overlaps plus per-tool
  BSJ fragment counts to expose sensitivity differences such as DCC misses.
when_to_use_negative:
- You only need to quantify circular vs. linear expression per locus (FPB/CIRCscore)
  rather than compare detection sensitivity across tools — use CLEAR's circ_quant
  directly instead
- Input RNA-seq is poly(A)+ rather than ribo−-depleted, since circRNA/BSJ detection
  pipelines like CLEAR are designed for ribo− data
- You lack the raw FASTQ or a clean HISAT2 BAM (DCC requires a BAM without unmapped
  reads, so read-mapping-only comparisons cannot be performed)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIRCexplorer3-CLEAR
  role: Pipeline (clear_quant) that embeds CIRCexplorer2 to detect circRNAs and count
    BSJ-mapped fragments (readNumber) from FASTQ via HISAT2 alignment
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Back-end circRNA caller (>=2.3.6) invoked within clear_quant to produce BSJ
    call sets filtered at readNumber >= 3
  repo: https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- name: HISAT2
  role: Spliced aligner (>=2.0.5) producing the shared BAM used by CLEAR/circ_quant
    and DCC
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: MapSplice
  role: Independent circRNA caller run on the same FASTQ files to generate the second
    call set for overlap intersection
- name: DCC/circtools
  role: Third circRNA detection pipeline run on the HISAT2 BAM; its calls are filtered
    to >=3 BSJ fragments and compared to expose missed circRNAs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:4df57da9ffc0fe4450c10516d30236f2f9810ae1dbecb650d09603407ef559f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - DCC/circtools
  - HISAT2
  - MapSplice
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-tool-detection-overlap-analysis

## Summary

Runs three circRNA detection pipelines (CLEAR-embedded CIRCexplorer2, MapSplice, and circtools-embedded DCC) on the same PA1 ribo− RNA-seq data, filters each call set to ≥3 BSJ-supporting fragments, and intersects them by genomic coordinates to quantify tool agreement, including circRNAs missed by DCC.

## When to use

Use when benchmarking circRNA/BSJ detection tools on identical ribo− RNA-seq FASTQ inputs and you need pairwise/three-way call-set overlaps plus per-tool BSJ fragment counts to expose sensitivity differences such as DCC misses.

## When NOT to use

- You only need to quantify circular vs. linear expression per locus (FPB/CIRCscore) rather than compare detection sensitivity across tools — use CLEAR's circ_quant directly instead
- Input RNA-seq is poly(A)+ rather than ribo−-depleted, since circRNA/BSJ detection pipelines like CLEAR are designed for ribo− data
- You lack the raw FASTQ or a clean HISAT2 BAM (DCC requires a BAM without unmapped reads, so read-mapping-only comparisons cannot be performed)

## Inputs

- Paired-end PA1 ribo− RNA-seq FASTQ files (mate_1/mate_2)
- hg38 genome FASTA, annotation GTF, HISAT2 index, and bowtie1 (TopHat-Fusion) index
- HISAT2-aligned BAM without unmapped reads (for DCC and circ_quant)

## Outputs

- circrna_tool_overlap_counts.tsv with Venn-style pairwise and three-way overlap counts (e.g., 356 CIRCexplorer2-and-MapSplice-only, 787 three-way)
- Per-circRNA per-tool BSJ fragment counts (readNumber) for the overlapping call set
- Filtered circRNA call sets from CIRCexplorer2/CLEAR, MapSplice, and DCC, each restricted to >=3 BSJ-supporting fragments

## How to apply

Install the CLEAR pipeline via 'python ./setup.py install' and prepare the hg38 genome FASTA, annotation GTF, HISAT2 and bowtie1 indexes. Align paired PA1 ribo− RNA-seq FASTQ files with HISAT2 (>=2.0.5) and run clear_quant with CIRCexplorer2 (>=2.3.6), keeping only circRNAs with readNumber >= 3 BSJ-supporting fragments; run MapSplice independently on the same FASTQs with the same >=3 BSJ-fragment filter, and run DCC on the HISAT2 BAM (which must contain no unmapped reads) with the same filter. Intersect the three filtered sets by (chrom, start, end, strand) to compute pairwise and three-way overlap counts, and tabulate per-tool BSJ fragment counts (readNumber) for the overlapping circRNAs to reveal DCC's reduced BSJ-fragment calling. The rationale is that identical filtering and coordinate-based intersection on shared alignments makes the three pipelines' detection efficiency directly comparable, recovering the reported Figure 3B-C results.

## Related tools

- **CIRCexplorer3-CLEAR** (Pipeline (clear_quant) that embeds CIRCexplorer2 to detect circRNAs and count BSJ-mapped fragments (readNumber) from FASTQ via HISAT2 alignment) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Back-end circRNA caller (>=2.3.6) invoked within clear_quant to produce BSJ call sets filtered at readNumber >= 3) — https://github.com/YangLab/CIRCexplorer2/tree/master/circ2
- **HISAT2** (Spliced aligner (>=2.0.5) producing the shared BAM used by CLEAR/circ_quant and DCC) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **MapSplice** (Independent circRNA caller run on the same FASTQ files to generate the second call set for overlap intersection)
- **DCC/circtools** (Third circRNA detection pipeline run on the HISAT2 BAM; its calls are filtered to >=3 BSJ fragments and compared to expose missed circRNAs)

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- The overlap table must contain 356 as the count of circRNAs detected by both CIRCexplorer2 and MapSplice but missed by DCC
- The three-way (CIRCexplorer2 ∩ MapSplice ∩ DCC) intersection count must equal 787, matching Figure 3B
- Per-tool BSJ fragment counts for the 787 overlapping circRNAs must show systematically reduced readNumber values for DCC, as in Figure 3C
- Every call set is restricted to >=3 BSJ-supporting fragments before intersection — unfiltered counts indicate the workflow was misapplied
- The DCC input HISAT2 BAM must contain no unmapped reads; a BAM with unmapped records is an invalid run

## Limitations

- DCC inefficiently calls BSJ-mapped fragments in general for overlapping circRNAs, so DCC counts should not be used as an expression-depth reference
- The HISAT2 BAM fed to DCC must exclude unmapped reads, adding a preprocessing constraint
- No software version numbers are stated for CLEAR itself, and no data accessions are deposited for the RNA-seq comparisons, so exact reproduction depends on matching CIRCexplorer2 (>=2.3.6) and HISAT2 (>=2.0.5)

## Evidence

- [abstract] we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR): "we update the previously-reported CIRCexplorer pipeline to version 3 for circular and linear RNA expression analysis from ribosomal-RNA depleted RNA-seq (CIRCexplorer3-CLEAR)"
- [discussion] Compared to other multi-step methods for circular and linear RNA comparison, such as DCC/circtools: "Compared to other multi-step methods for circular and linear RNA comparison, such as DCC/circtools"
- [readme] clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf: "clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf"
- [readme] *hisat_aligned.bam should not contain unmapped reads.*: "*hisat_aligned.bam should not contain unmapped reads.*"
