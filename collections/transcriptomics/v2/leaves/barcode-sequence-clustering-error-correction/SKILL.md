---
name: barcode-sequence-clustering-error-correction
description: Use when raw barcode FASTQ reads contain sequencing errors that would
  inflate unique-barcode counts, and you need to derive a consensus barcode repertoire
  (per-sample counts table) before population-level quantification or cross-tool comparison.
when_to_use_negative:
- The input is already a consensus barcode counts table or frequency table — clustering
  is a pre-quantification step on trimmed reads
- Reads have not been quality-filtered and constant-region trimmed (cutadapt) — untrimmed
  reads will pollute clusters with flanking-sequence errors
- The study uses evolving or lineage-reconstructing barcodes — BARtab/bartools do
  not officially support evolving barcode methods
- A curated reference barcode library with exact-match extraction is appropriate (e.g.
  xcalibr-style template matching) and error clustering is unnecessary
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: starcode
  role: Message-passing DNA sequence clustering of trimmed barcode reads at Levenshtein
    distance to build the consensus repertoire; starcode-umi variant for UMI-aware
    processing
  repo: https://github.com/gui11aume/starcode
- name: BARtab
  role: Nextflow pipeline (single-bulk reference-free mode) that chains FASTQC, fastp
    filtering, cutadapt trimming, and starcode clustering
  repo: https://github.com/DaneVass/BARtab
- name: fastp
  role: Upstream quality filtering of reads (minqual 20, pctqual 80) before clustering
  repo: https://github.com/OpenGene/fastp
- name: cutadapt
  role: Constant-region trimming and length/N filtering (constantmismatches 0.1, min_readlength
    20) so clustering acts only on barcode sequence
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: bartools
  role: R package for downstream barcode set overlap computation and Pearson/Spearman
    correlation to validate the clustered repertoire
  repo: https://github.com/DaneVass/bartools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:c46bd3034dfc276c2e4592a5090de13e02dadd026a3198801fed8abdceb873ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gui11aume/starcode
  tools:
  - BARtab
  - R
  - bartools
  - cutadapt
  - fastp
  - pycashier
  - starcode
  - timemachine
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# barcode-sequence-clustering-error-correction

## Summary

Consensus-clustering of quality-filtered, constant-region-trimmed synthetic barcode reads with starcode's message-passing algorithm to collapse sequencing errors into a barcode repertoire, as implemented in the BARtab Nextflow pipeline and evaluated with bartools against pycashier and TimeMachine.

## When to use

Use when raw barcode FASTQ reads contain sequencing errors that would inflate unique-barcode counts, and you need to derive a consensus barcode repertoire (per-sample counts table) before population-level quantification or cross-tool comparison.

## When NOT to use

- The input is already a consensus barcode counts table or frequency table — clustering is a pre-quantification step on trimmed reads
- Reads have not been quality-filtered and constant-region trimmed (cutadapt) — untrimmed reads will pollute clusters with flanking-sequence errors
- The study uses evolving or lineage-reconstructing barcodes — BARtab/bartools do not officially support evolving barcode methods
- A curated reference barcode library with exact-match extraction is appropriate (e.g. xcalibr-style template matching) and error clustering is unnecessary

## Inputs

- FASTQ files of targeted barcode sequencing reads (single-end bulk mode)
- Quality-filtered reads from fastp (minqual 20, pctqual 80)
- Constant-region-trimmed reads from cutadapt (constantmismatches 0.1, min_readlength 20)
- Clustering parameters: Levenshtein cluster_distance (default 3; 8 for 100bp FateMap construct) and cluster_ratio 3

## Outputs

- starcode/<sample_id>_starcode.tsv per sample: consensus barcode repertoire with counts
- counts/all_counts_combined.tsv: combined per-sample barcode counts via outer join (COMBINE_BARCODE_COUNTS)
- Barcode frequency table (barcodes with at least 0.001% frequency within a sample) for downstream bartools overlap/correlation analysis

## How to apply

First quality-filter reads with fastp (minqual 20, pctqual 80) and trim constant regions with cutadapt (constantmismatches 0.1, min_readlength 20) so only barcode-proximal reads enter clustering. Then cluster trimmed barcode reads with starcode using the message-passing clustering method at Levenshtein cluster_distance 3 and cluster_ratio 3 — the default distance of 3 is conservative to avoid collapsing truly distinct barcodes; for longer constructs such as the 100bp FateMap barcode a distance of 8 is the recommended exemplar. Emit starcode/<sample_id>_starcode.tsv per sample as the consensus repertoire, then combine per-sample counts with an outer join via COMBINE_BARCODE_COUNTS into counts/all_counts_combined.tsv. Rationale: clustering-based error correction is chosen over exact-match reference counting (e.g. xcalibr) when no gold-standard barcode reference is available and parameters must flex to different barcode designs and lengths.

## Related tools

- **starcode** (Message-passing DNA sequence clustering of trimmed barcode reads at Levenshtein distance to build the consensus repertoire; starcode-umi variant for UMI-aware processing) — https://github.com/gui11aume/starcode
- **BARtab** (Nextflow pipeline (single-bulk reference-free mode) that chains FASTQC, fastp filtering, cutadapt trimming, and starcode clustering) — https://github.com/DaneVass/BARtab
- **fastp** (Upstream quality filtering of reads (minqual 20, pctqual 80) before clustering) — https://github.com/OpenGene/fastp
- **cutadapt** (Constant-region trimming and length/N filtering (constantmismatches 0.1, min_readlength 20) so clustering acts only on barcode sequence) — https://cutadapt.readthedocs.io/en/stable/
- **bartools** (R package for downstream barcode set overlap computation and Pearson/Spearman correlation to validate the clustered repertoire) — https://github.com/DaneVass/bartools

## Examples

```
nextflow run DaneVass/bartab -profile test_SE_ref_free,docker
```

## Evaluation signals

- Per-sample output starcode/<sample_id>_starcode.tsv exists for all 22 samples and contains clustered consensus barcodes, not raw reads
- Three-tool (BARtab vs pycashier vs TimeMachine) barcode overlap reaches 144,745 barcodes, i.e. 97.3% of barcodes with at least 0.001% frequency within a sample
- BARtab-derived counts vs the published Goyal et al. dataset show Pearson and Spearman correlation above the 0.96 concordance level
- tools_comparison_results.tsv reports the 1,810-barcode overlap shared uniquely by BARtab and TimeMachine and per-tool runtime/memory (BARtab 35 min / 6.83 GB)
- Chosen Levenshtein distance matches construct length: 3 for short barcodes (conservative default), 8 for the 100bp FateMap construct

## Limitations

- The default Levenshtein distance of 3 is deliberately conservative to avoid collapsing truly distinct barcodes, which may under-merge genuine error clusters for some constructs
- Construct-specific tuning is required: the 100bp FateMap barcode construct needs a recommended distance of 8 rather than the default
- Benchmarking was hampered by the lack of a gold-standard dataset with known barcode abundance and by differing parameter flexibility between tools; simulated-data benchmarking was not performed
- BARtab and bartools do not officially support evolving barcode methods

## Evidence

- [methods] Conservative default Levenshtein clustering distance: "The default Levenshtein distance for clustering is 3 which is conservative to avoid collapsing truly distingt barcodes."
- [methods] Constant-region trimming parameters before clustering: "The fraction of mismatches in the constant region can be specified with `constantmismatches` (default 0.1)."
- [readme] starcode is a DNA sequence clustering tool: [evidence span withheld — anchor status: external_doc]
- [discussion] Benchmark validation of the BARtab clustering workflow: "We benchmarked BARtab by reproducing published results, showing improved runtime and sensitivity."
