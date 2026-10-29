---
name: starsolo-high-throughput-alignment
description: Use when raw Seq-Scope 2nd-seq FASTQ pairs must be aligned and converted
  to digital expression matrices at ultra-high resolution (>1M spatial barcodes/mm2),
  especially within the STtools Step A3 stage.
when_to_use_negative:
- Input is already a digital expression matrix or Seurat object rather than raw 2nd-seq
  FASTQ — proceed directly to Steps C1–C3 instead
- The platform data can be handled by lighter tools and does not involve sub-micrometer-resolution
  Seq-Scope-scale barcode counts (alignment step is tuned for >1M spatial barcodes/mm2)
- Running on a non-Linux OS — the README states a Linux operating system is necessary
  to run the STtools package
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: STtools
  role: Pipeline wrapper; Step A3 invokes STARsolo on the whitelist and 2nd-seq FASTQs
    via sttools.py --run-all
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Performs the cDNA-to-reference alignment and DGE generation (Gene/GeneFull/Velocyto
    matrices)
  repo: https://github.com/alexdobin/STAR
- name: STAR >=2.7.5c
  role: Required aligner binary; version floor stated in installation instructions
  repo: https://github.com/alexdobin/STAR
- name: seqtk
  role: Sequence-toolkit dependency used in the STtools preprocessing stage feeding
    alignment
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:7d11742a783fa719079dce08d17b291961ab80163d35b16d8817e2fc651062d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STAR >=2.7.5c
  - STARsolo
  - STtools
  - seqtk
  - spacemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# starsolo-high-throughput-alignment

## Summary

Running STARsolo (STAR >=2.7.5c) alignment of Seq-Scope 2nd-seq cDNA FASTQ pairs against a STAR genome index, seeded by the spatial-barcode whitelist from STtools Step A1, to produce digital expression matrices (Gene/GeneFull/Velocyto formats) that feed STtools Steps C1–V2.

## When to use

Use when raw Seq-Scope 2nd-seq FASTQ pairs must be aligned and converted to digital expression matrices at ultra-high resolution (>1M spatial barcodes/mm2), especially within the STtools Step A3 stage.

## When NOT to use

- Input is already a digital expression matrix or Seurat object rather than raw 2nd-seq FASTQ — proceed directly to Steps C1–C3 instead
- The platform data can be handled by lighter tools and does not involve sub-micrometer-resolution Seq-Scope-scale barcode counts (alignment step is tuned for >1M spatial barcodes/mm2)
- Running on a non-Linux OS — the README states a Linux operating system is necessary to run the STtools package

## Inputs

- 1st-seq FASTQ (fastq.gz) with spatial barcode coordinates in Illumina sequence identifiers (Step A1 input)
- 2nd-seq cDNA FASTQ pair (fastq.gz) (Step A3 input)
- Valid-barcode whitelist.txt produced by Step A1
- STAR genome index built from reference FASTA and GTF (e.g. mm10)

## Outputs

- Digital expression matrix (DGE) in Gene format
- Digital expression matrix (DGE) in GeneFull format
- Digital expression matrix (DGE) in Velocyto format (spliced/unspliced)
- Spatial coordinates .txt files and barcode whitelist (from prerequisite Step A1)

## How to apply

First run STtools Step A1 on the 1st-seq FASTQ (spatial barcodes with coordinates in Illumina sequence identifiers, e.g. --seqscope1st 'HiSeq') to obtain the spatial coordinates .txt file and the barcode whitelist consumed by STARsolo. Then in Step A3, supply the valid barcodes whitelist.txt, the 2nd-seq fastq.gz pair, and the STAR genome index (built per the igordot/genomics gtf/fasta recipe if not provided) to STARsolo; set the STAR binary path via --star-path and require STAR >=2.7.5c. STARsolo outputs the DGE in Gene, GeneFull, and Velocyto formats — the plain Gene matrix feeds Step C1/C2 grid collapsing, while the Velocyto-format DGE feeds Steps V1/V2 subcellular plots. For full-dataset scalability (e.g. GSM5212844, 15M spatial barcodes, 1.9 billion raw reads), launch via 'python3 sttools.py --run-all' on an HPC node with at least ~6 CPUs and 30GB RAM and expect ~16 hours total across A1–V2. Validate by checking that all pipeline steps (A1–V2) complete and that DGE outputs exist for each of the three STARsolo matrix flavors.

## Related tools

- **STtools** (Pipeline wrapper; Step A3 invokes STARsolo on the whitelist and 2nd-seq FASTQs via sttools.py --run-all) — https://github.com/seqscope/STtools
- **STARsolo** (Performs the cDNA-to-reference alignment and DGE generation (Gene/GeneFull/Velocyto matrices)) — https://github.com/alexdobin/STAR
- **STAR >=2.7.5c** (Required aligner binary; version floor stated in installation instructions) — https://github.com/alexdobin/STAR
- **seqtk** (Sequence-toolkit dependency used in the STtools preprocessing stage feeding alignment) — https://github.com/lh3/seqtk

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --lane-tiles 1_2106 --binsize 300 --window 150
```

## Evaluation signals

- Step A3 produces DGE outputs in all three STARsolo flavors (Gene, GeneFull, Velocyto) that downstream steps C1/C2 and V1/V2 can consume
- The whitelist.txt supplied to STARsolo matches the spatial-coordinate extraction from the 1st-seq FASTQ (Step A1 output)
- Full GSM5212844 run (15M spatial barcodes, 1.9B reads) completes all eight steps A1–V2 in wall-clock time consistent with the reported ~16 hours on 6 CPUs / 30GB RAM
- pipeline_run_log.md records per-step completion of STtools A1–V2 for the full dataset
- STAR binary version satisfies the stated floor of >=2.7.5c

## Limitations

- Linux operating system is required; STtools is not supported on other OSes
- Subcellular analysis with the modified sliding window (MSSW) is not feasible because each spatial segment must be larger than subcellular compartments (~10um)
- Downstream analysis is constrained by very low UMI counts per spatial barcode at sub-micrometer resolution, and no spatial smoothing or imputation of expression profiles is yet available in STtools

## Evidence

- [readme] Step A3 inputs and outputs: "**Step A3** takes valid barcodes `whitelist.txt`, 2nd-seq `fastq.gz` files, and the STAR indices of reference genome as input to run `STARsolo` alignment"
- [methods] STARsolo role in STtools: "STtools aligns each cDNA sequence to the reference genome using STARsolo (Kaminow et al., 2021)."
- [methods] Alignment step scope: "The alignment step performs QC, alignment, and spatial expression matrix generation from raw sequence data."
- [results] Full-dataset runtime: "we ran both tools with the same mouse liver SeqScope data (GSM5212844)"
- [readme] Version requirement: "STAR>=2.7.5c (Click for instructions to install [STAR](https://github.com/alexdobin/STAR))"
