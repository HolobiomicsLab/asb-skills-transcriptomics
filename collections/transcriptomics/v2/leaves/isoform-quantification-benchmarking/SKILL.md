---
name: isoform-quantification-benchmarking
description: Use when evaluating how accurately a long-read isoform detection/quantification
  tool recovers and quantifies known transcripts from aligned PacBio/ONT reads, using
  spike-in references like SIRVs as ground truth.
when_to_use_negative:
- Input is a genome-wide, high-throughput transcriptome discovery project rather than
  a selected-gene/low-throughput splicing assay — VIsoQLR is explicitly 'tailored
  to thoroughly analyze mRNA expression and maturation in low-throughput splicing
  assays' and analyzed 'one gene at a time'
- The reads are unaligned raw FASTQ with no reference, or the goal is de novo transcriptome
  assembly without spike-in ground truth — no gold-standard isoform set exists to
  benchmark against
- Evaluation requires short-read or hybrid spliced-alignment methods (e.g., tools
  needing short-read data to infer splice sites like FLAIR with short-read correction
  or IDP) outside the transcriptome-based tool comparison scope
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: VIsoQLR
  role: Candidate isoform detection/quantification tool run with default parameters
    via Docker; consensus splice sites, isoform identification and quantification
    from GMAP-aligned BAM
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: FLAIR
  role: Comparator tool run with default parameters in the SIRV benchmark (detected
    37/54% of isoforms)
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
- name: StringTie2
  role: Comparator tool run with default parameters in the SIRV benchmark (detected
    12/18% of isoforms); known to extend isoform boundaries as an artifact
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
- name: GMAP
  role: Aligner used with --format=2 to produce GFF3/BAM alignments of PacBio reads;
    its index is built from the uploaded reference sequence
- name: TBLabFJD/VIsoQLR_benchmark scripts
  role: isoform_analysis.sh (preprocessing, GMAP index, FLAIR/StringTie2 runs), isoform_comparison.R
    (tool comparison), isoform_analysis_visoqlr_perc.R (read-threshold sweep)
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:f114ea25e11fec66bebffca983a441cb408e1067510747c4bfad6a896394780f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - GMAP
  - StringTie2
  - TBLabFJD/VIsoQLR_benchmark scripts
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# isoform-quantification-benchmarking

## Summary

Benchmark isoform detection and quantification accuracy of long-read transcript tools (e.g., VIsoQLR vs FLAIR vs StringTie2) against spike-in gold standards such as Lexogen SIRV Isoform Mix E0, by counting recovered reference isoforms, measuring exon-coordinate accuracy, and computing cosine similarity of abundance profiles.

## When to use

Use when evaluating how accurately a long-read isoform detection/quantification tool recovers and quantifies known transcripts from aligned PacBio/ONT reads, using spike-in references like SIRVs as ground truth.

## When NOT to use

- Input is a genome-wide, high-throughput transcriptome discovery project rather than a selected-gene/low-throughput splicing assay — VIsoQLR is explicitly 'tailored to thoroughly analyze mRNA expression and maturation in low-throughput splicing assays' and analyzed 'one gene at a time'
- The reads are unaligned raw FASTQ with no reference, or the goal is de novo transcriptome assembly without spike-in ground truth — no gold-standard isoform set exists to benchmark against
- Evaluation requires short-read or hybrid spliced-alignment methods (e.g., tools needing short-read data to infer splice sites like FLAIR with short-read correction or IDP) outside the transcriptome-based tool comparison scope

## Inputs

- GMAP-aligned PacBio UHR Iso-Seq BAM file (e.g., cond_VIsoQLR_GMAP from TBLabFJD/VIsoQLR_benchmark)
- Lexogen SIRV reference transcripts with gold-standard abundances
- Optionally, raw FASTQ reads and a reference genome for building a GMAP index

## Outputs

- Detected isoform list per tool (isoform information panel and exonic starting/ending points)
- Isoform abundance profiles (proportions per transcript)
- Benchmark metrics: isoform detection counts, exon-coordinate accuracy, overall and partial cosine similarity to gold standard

## How to apply

Run each candidate tool with default parameters on the same aligned BAM (e.g., GMAP-aligned PacBio UHR Iso-Seq data from the TBLabFJD/VIsoQLR_benchmark repo), then upload the Lexogen SIRV reference transcripts as known isoforms for comparison. For VIsoQLR, keep defaults: CEC read threshold 2%, default padding, and merge-close-splice-sites settings for consensus exon coordinate detection; optionally sweep the read threshold via isoform_analysis_visoqlr_perc.R since most missed isoforms fall below the default 2%. Count detected SIRV isoforms against the 68-isoform gold standard, compute exon-coordinate accuracy, and compute cosine similarity between the tool's isoform abundance profile and Lexogen gold standard abundances (overall and per partial-SIRV set, e.g., SIRV7). Judge success by reproducing benchmark-level agreement — e.g., VIsoQLR: 49/68 isoforms (72%), 99% exon coordinate accuracy, overall cosine similarity 0.78.

## Related tools

- **VIsoQLR** (Candidate isoform detection/quantification tool run with default parameters via Docker; consensus splice sites, isoform identification and quantification from GMAP-aligned BAM) — https://github.com/TBLabFJD/VIsoQLR
- **FLAIR** (Comparator tool run with default parameters in the SIRV benchmark (detected 37/54% of isoforms)) — https://github.com/TBLabFJD/VIsoQLR_benchmark
- **StringTie2** (Comparator tool run with default parameters in the SIRV benchmark (detected 12/18% of isoforms); known to extend isoform boundaries as an artifact) — https://github.com/TBLabFJD/VIsoQLR_benchmark
- **GMAP** (Aligner used with --format=2 to produce GFF3/BAM alignments of PacBio reads; its index is built from the uploaded reference sequence)
- **TBLabFJD/VIsoQLR_benchmark scripts** (isoform_analysis.sh (preprocessing, GMAP index, FLAIR/StringTie2 runs), isoform_comparison.R (tool comparison), isoform_analysis_visoqlr_perc.R (read-threshold sweep)) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Detected isoform count matches benchmark expectation: VIsoQLR 49/68 SIRV isoforms (72%) vs FLAIR 37 (54%) vs StringTie2 12 (18%)
- Exon-coordinate accuracy of detected isoforms is 99% for VIsoQLR on SIRVs
- Cosine similarity between VIsoQLR abundance profile and Lexogen gold standard is ~0.78 overall, with SIRV7 partial similarity highest at 0.97
- StringTie2-specific artifact check: isoform boundary extension relative to the input BAM must be flagged as tool artifact, not a data property
- Threshold-sensitivity check: most missing isoforms lie below the 2% default read threshold; lowering the threshold recovers them at the cost of extra isoform calls

## Limitations

- Missing isoforms in the automatic VIsoQLR analysis are mostly below the 2% default threshold and require manual parameter tuning; per-gene curation is expected
- StringTie2 extends isoform boundaries (an artifact not present in the BAM files) and fails to call isoforms lacking certain exons, biasing detection counts
- Small isoform abundances (e.g., Iso5 at 4% and Iso8 at 1.7%) may be overrepresented by LRS and hard to distinguish from noise in electropherograms
- Minimap2 alignment has a limitation advised on its GitHub relevant to splice-aware use; no software version numbers were reported for VIsoQLR, StringTie2, or FLAIR in the benchmark
- No data accession identifiers for the underlying LRS datasets are given in the article's supplementary files

## Evidence

- [discussion] VIsoQLR detects 72% of known SIRV isoforms with 99% exon coordinate accuracy, outperforming StringTie2 and FLAIR: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [discussion] Default CEC read threshold 2% and its cost: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [discussion] StringTie2 artifact: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [methods] Docker run command: "docker run -it -p 8888:8888 tblabfjd/visoqlr:latest"
- [readme] Benchmark repo contents: [evidence span withheld — anchor status: external_doc]
