---
name: mitochondrial-variant-pileup-genotyping
description: Use when you have single-cell RNA/DNA/ATAC-seq BAM files (10x cell-barcode
  or Smart-seq2) and need per-cell mtDNA genotype matrices as input for clonal substructure
  discovery with MQuad or vireoSNP.
when_to_use_negative:
- 'Calling nuclear genome SNVs: nuclear SNV callers such as Monovar and Conbase assume
  a diploid context and this pipeline targets the haploid mitochondrial genome'
- Input is FASTQ — reads must first be aligned (e.g., STAR to hg19/mm10 for scRNA-seq,
  CellRanger for 10x) and duplicates marked/removed (Picard MarkDuplicates) before
  pileup
- For Smart-seq2-style cellSNP mode 2 pileups where duplicate/orphan filtering cannot
  be tuned — cellsnp-lite is recommended instead because it offers more flexible read
  filtering
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: cellsnp-lite
  role: 'Primary pileup tool: generates the SNP-by-cell AD/DP genotyping matrices
    from BAM files, used at v1.2.1 in the article''s clonality pipeline'
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: cellSNP
  role: Python predecessor of cellsnp-lite for piling up expressed alleles per cell
    barcode; cellsnp-lite is its faster C version
  repo: https://github.com/single-cell-genetics/cellSNP
- name: MQuad
  role: 'Downstream consumer of the AD/DP matrices: fits binomial mixture models to
    select clonally informative mtDNA variants'
  repo: https://github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: 'Downstream consumer: uses the selected SNPs with variational inference to
    assign cells to mitochondrial clones'
  repo: https://github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:b4adf57f29c369fbbd36ecf48d9cdbfe7502f8802b8c7de1d8748ec43991de2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - CellRanger
  - MQuad
  - cellSNP
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mitochondrial-variant-pileup-genotyping

## Summary

Pile up mitochondrial reads from single-cell BAM files with cellsnp-lite (v1.2.1) to generate SNP-by-cell matrices of alternate-allele depths (AD) and total depths (DP) that feed MQuad and vireoSNP-based clonal inference.

## When to use

Use when you have single-cell RNA/DNA/ATAC-seq BAM files (10x cell-barcode or Smart-seq2) and need per-cell mtDNA genotype matrices as input for clonal substructure discovery with MQuad or vireoSNP.

## When NOT to use

- Calling nuclear genome SNVs: nuclear SNV callers such as Monovar and Conbase assume a diploid context and this pipeline targets the haploid mitochondrial genome
- Input is FASTQ — reads must first be aligned (e.g., STAR to hg19/mm10 for scRNA-seq, CellRanger for 10x) and duplicates marked/removed (Picard MarkDuplicates) before pileup
- For Smart-seq2-style cellSNP mode 2 pileups where duplicate/orphan filtering cannot be tuned — cellsnp-lite is recommended instead because it offers more flexible read filtering

## Inputs

- Analysis-ready BAM/SAM file(s) aligned to hg38 (or hg19/mm10) containing mitochondrial reads with cell-barcode tags (10x) or per-sample BAMs (Smart-seq2)
- Cell barcode list file (-b) for 10x data
- Optional region VCF of candidate SNPs (-R) or target chromosome spec such as chrMT

## Outputs

- SNP-by-cell matrix of alternate-allele depths (AD) per cell per variant
- SNP-by-cell matrix of total depths (DP) per cell per variant
- VCF-style pileup output directly loadable by MQuad and vireoSNP

## How to apply

Run cellsnp-lite (v1.2.1) on the analysis-ready, genome-aligned BAM file to pile up reads over the mitochondrial genome (e.g., --chrom chrMT), splitting counts by cell barcode to produce a SNP-by-cell AD/DP matrix. Retain PCR duplicates for scRNA-seq/10x data (cellSNP uses a large --maxFLAG by default because duplicates are not well flagged in CellRanger and filtering them loses a substantial fraction of SNPs); set --UMItag None for Smart-seq2 or bulk inputs. The resulting AD/DP matrices are consumed directly by MQuad, which fits one- versus two-component binomial mixture models per variant and selects informative SNPs at the Kneedle ΔBIC knee point, then by vireoSNP for clone assignment. Optionally exclude error-prone rCRS regions 302–315, 513–525, and 3105–3109 from downstream variant selection. Judge success by checking the output VCF/matrix schema (cell x SNP AD/DP counts) and that downstream MQuad recovers biologically expected clones (e.g., 24 informative variants / 5 clones in MKN-45, 2 variants / 2 clones in TNBC1).

## Related tools

- **cellsnp-lite** (Primary pileup tool: generates the SNP-by-cell AD/DP genotyping matrices from BAM files, used at v1.2.1 in the article's clonality pipeline) — https://github.com/single-cell-genetics/cellsnp-lite
- **cellSNP** (Python predecessor of cellsnp-lite for piling up expressed alleles per cell barcode; cellsnp-lite is its faster C version) — https://github.com/single-cell-genetics/cellSNP
- **MQuad** (Downstream consumer of the AD/DP matrices: fits binomial mixture models to select clonally informative mtDNA variants) — https://github.com/single-cell-genetics/MQuad
- **vireoSNP** (Downstream consumer: uses the selected SNPs with variational inference to assign cells to mitochondrial clones) — https://github.com/single-cell-genetics/vireo

## Examples

```
cellsnp-lite -s sample.bam -b barcodes.tsv -O out_dir --chrom chrMT -p 20
```

## Evaluation signals

- Output contains paired AD and DP matrices indexed consistently over cells and mitochondrial SNP positions
- Downstream MQuad selection recovers the expected number of informative variants (e.g., 24 for MKN-45 scDNA-seq; 2 for TNBC1 scRNA-seq)
- vireoSNP on the piled-up SNPs assigns the expected clones (e.g., 5 mtDNA clones in MKN-45; 1097/1146 cells at prob >0.8 in TNBC1)
- Error-prone rCRS regions 302–315, 513–525, 3105–3109 are excluded from the selected variant set
- No unexpected coverage loss from duplicate filtering — for 10x/scRNA-seq data, duplicates should be retained (--maxFLAG default) so that variant counts are not depleted

## Limitations

- cellSNP mode 2 (pysam pileup) filters duplicates and orphan reads by default, which may cause unexpected coverage reduction; cellsnp-lite provides more flexible read filtering for this case
- PCR duplicates are not well flagged in CellRanger, so default filtering would lose a substantial fraction of SNPs — a large maxFLAG is used to include duplicates for scRNA-seq
- Pileup of whole chromosomes without a candidate SNP list can output false positives (somatic variants, RNA editing) that confound downstream demultiplexing
- Uninformative and noisy mtDNA SNVs, combined with sequencing errors, confound clonal inference if not filtered by ΔBIC-based model selection

## Evidence

- [discussion] raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23"
- [discussion] SNP-by-cell matrix: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [readme] duplicate filtering: [evidence span withheld — anchor status: external_doc]
- [discussion] MQuad binomial mixture: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] vireoSNP clone assignment: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [readme] recommended pipeline: "use cellSNP or cellsnp-lite (a faster version of cellSNP, still at testing stage so might be unstable) to pileup mtDNA variants from raw .bam file(s)"
