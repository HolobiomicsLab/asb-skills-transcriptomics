---
name: bulk-barcode-count-extraction
description: Use when raw bulk NGS FASTQ files contain expressed synthetic lineage
  barcodes and you need reference-free barcode detection and quantification into a
  per-sample counts table.
when_to_use_negative:
- Input is single-cell (10X) or spatial transcriptomics data — BARtab has dedicated
  single-cell and SAW/spatial modes instead of single-bulk mode
- Barcodes are evolving over time in a lineage tree, which BARtab and bartools do
  not officially support
- You need a gold-standard benchmark with known barcode abundance — the article notes
  no such dataset exists and differing parameter flexibility complicates tool comparisons
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: BARtab
  role: Nextflow pipeline orchestrating QC, filtering, trimming, clustering, and counts
    tabulation in single-bulk reference-free mode
  repo: https://github.com/DaneVass/BARtab/
- name: FastQC
  role: Per-sample QC of FASTQ input
- name: fastp
  role: Quality filtering with minqual 20 and pctqual 80
  repo: https://github.com/OpenGene/fastp
- name: cutadapt
  role: Constant-region trimming and length/N filtering (constantmismatches 0.1, min_readlength
    20)
- name: starcode
  role: Message-passing clustering of barcode reads at Levenshtein distance 3, ratio
    3 to derive consensus counts
  repo: https://github.com/gui11aume/starcode
- name: bartools
  role: R package for downstream joining, overlap, and correlation analysis of the
    combined counts table
  repo: https://github.com/DaneVass/bartools/
- name: Nextflow
  role: Workflow engine; run via `nextflow run DaneVass/bartab` with conda/docker/singularity
    profiles
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:276e8d7d5029b193d9abb0f5ae721d2c06c05863b0c5c0bf9fa28b289292b3f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FastQC
  - Nextflow
  - R
  - bartools
  - cutadapt
  - fastp
  - pycashier
  - starcode
  - timemachine
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bulk-barcode-count-extraction

## Summary

Extract and tabulate synthetic cellular barcode counts from bulk population-level FASTQ data using the BARtab Nextflow pipeline (DaneVass/bartab) in single-bulk reference-free mode, combining FASTQC QC, fastp quality filtering, cutadapt constant-region trimming, and starcode clustering into a per-sample barcode counts table.

## When to use

Use when raw bulk NGS FASTQ files contain expressed synthetic lineage barcodes and you need reference-free barcode detection and quantification into a per-sample counts table.

## When NOT to use

- Input is single-cell (10X) or spatial transcriptomics data — BARtab has dedicated single-cell and SAW/spatial modes instead of single-bulk mode
- Barcodes are evolving over time in a lineage tree, which BARtab and bartools do not officially support
- You need a gold-standard benchmark with known barcode abundance — the article notes no such dataset exists and differing parameter flexibility complicates tool comparisons

## Inputs

- Bulk single-end FASTQ files of population-level barcode amplicon sequencing (e.g. the 22-sample Goyal dataset)
- Optional barcode reference FASTA (for reference-based mode), e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta
- Barcode construct definition: constant region sequence and expected barcode length (e.g. 100bp FateMap construct)

## Outputs

- starcode/<sample_id>_starcode.tsv per-sample consensus barcode counts
- counts/all_counts_combined.tsv — outer-joined barcode counts table across all samples
- fastqc/fastp QC reports per sample

## How to apply

Run `nextflow run DaneVass/bartab` in single-bulk reference-free mode: FASTQC performs QC, fastp quality-filters reads (minqual 20, pctqual 80), and cutadapt trims constant regions (constantmismatches 0.1, min_readlength 20). Cluster the trimmed barcode reads with starcode using message-passing clustering at Levenshtein cluster_distance 3 and cluster_ratio 3 (use a distance of 8 for the 100bp FateMap barcode construct) to derive consensus barcodes, writing starcode/<sample_id>_starcode.tsv per sample. Combine per-sample counts across all samples with COMBINE_BARCODE_COUNTS via an outer join into counts/all_counts_combined.tsv. The conservative default cluster_distance of 3 avoids collapsing truly distinct barcodes; adjust per barcode construct length. Validate that the combined counts table reproduces published repertoires, e.g. >0.96 Pearson/Spearman concordance against Goyal et al. counts.

## Related tools

- **BARtab** (Nextflow pipeline orchestrating QC, filtering, trimming, clustering, and counts tabulation in single-bulk reference-free mode) — https://github.com/DaneVass/BARtab/
- **FastQC** (Per-sample QC of FASTQ input)
- **fastp** (Quality filtering with minqual 20 and pctqual 80) — https://github.com/OpenGene/fastp
- **cutadapt** (Constant-region trimming and length/N filtering (constantmismatches 0.1, min_readlength 20))
- **starcode** (Message-passing clustering of barcode reads at Levenshtein distance 3, ratio 3 to derive consensus counts) — https://github.com/gui11aume/starcode
- **bartools** (R package for downstream joining, overlap, and correlation analysis of the combined counts table) — https://github.com/DaneVass/bartools/
- **Nextflow** (Workflow engine; run via `nextflow run DaneVass/bartab` with conda/docker/singularity profiles)

## Examples

```
nextflow run DaneVass/bartab -profile test_SE_ref_free,docker --minqual 20 --pctqual 80 --constantmismatches 0.1 --min_readlength 20 --cluster_distance 3 --cluster_ratio 3
```

## Evaluation signals

- starcode/<sample_id>_starcode.tsv exists for every input sample with barcode and count columns
- counts/all_counts_combined.tsv is produced by an outer join across all samples and contains no dropped sample
- BARtab-derived counts correlate with an independent quantification (e.g. Goyal et al.) at Pearson and Spearman > 0.96
- On the 22-sample Goyal dataset, ~97.3% of barcodes with ≥0.001% frequency overlap with pycashier and TimeMachine detections (144,745 three-tool shared barcodes)
- QC module (SOFTWARE_CHECK/FASTQC) completes as the first modules of the Nextflow trace

## Limitations

- The default Levenshtein cluster_distance 3 is conservative to avoid collapsing truly distinct barcodes; the correct distance depends on the barcode construct (8 recommended for the 100bp FateMap construct), so mis-parameterization can merge or split barcodes
- BARtab and bartools do not officially support evolving barcode methods
- No gold-standard dataset with known barcode abundance exists, and differing parameter flexibility between tools complicates cross-tool benchmarking; simulated-data benchmarking for accuracy and parameter selection was not performed

## Evidence

- [abstract] we present BARtab, a portable and scalable Nextflow pipeline: "we present BARtab, a portable and scalable Nextflow pipeline"
- [methods] reference-free clustering default: "The default Levenshtein distance for clustering is 3 which is conservative to avoid collapsing truly distingt barcodes."
- [methods] cutadapt trim parameters: "The fraction of mismatches in the constant region can be specified with `constantmismatches` (default 0.1)."
- [methods] fastp filter parameters: "The minimum quality score to keep can be specified with the parameter `minqual`."
- [readme] run command: "Run any of the test datasets using `nextflow run DaneVass/bartab -profile <test_SE,test_PE,test_SE_ref_free,test_sc,test_sc_bam,test_sc_saw_fastq>,<conda,docker,singularity>,<slurm,lsf>`"
