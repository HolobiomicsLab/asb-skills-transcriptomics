---
name: mrsd-sequencing-depth-prediction
description: Use when planning bulk short-read poly-A RNA-seq for splice-junction
  surveillance of Mendelian disease genes and you need to know how many reads (e.g.
  --number_reads 8 over --sj_prop 0.75 junctions) a given tissue requires.
when_to_use_negative:
- The dataset is long-read, single-cell, or spatially resolved RNA-seq — no appropriate
  control datasets exist yet, and MRSD is validated only for bulk short-read poly-A
  enrichment data processed with the specified pipeline
- The goal is to predict suitability for allele-specific expression, differential
  gene expression, or isoform imbalance analysis — the article states the MRSD model
  cannot directly be extended to these
- The patient's disorder involves widespread transcriptomic changes (e.g. interferonopathies,
  chromatin structure disorders, spliceosome disruption) or strong eQTL/sQTL effects,
  which the current model does not account for
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: MRSDexplorer
  role: Python toolkit (bin/ scripts) used to compute MRSD scores from junction counts
    and sequencing depths over a parameter grid of --number_reads, --sj_prop, --mrsd_param,
    --read_type
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: process_cummings_output.py
  role: Merges and reformats Cummings pipeline *.csf.splicing.txt outputs into MRSDexplorer-compatible
    *.junc-counts.txt files
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: transcript_bed_generator.py
  role: Generates *.introns.bed files of transcript intron coordinates from a GTF
    and transcript ID list for transcript-level MRSD queries
  repo: github.com/mcgm-mrsd/mrsd-explorer
- name: MendelianRNA-seq
  role: Upstream scripts for generating transcriptome-wide junction counts from FASTQs
    via split-read alignment (Cummings et al. 2017 pipeline)
  repo: github.com/berylc/MendelianRNA-seq
- name: STAR
  role: Split-read aligner whose *.Log.final.out file provides the sample sequencing
    depths for *.seq-depths.txt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:07196ad0b33ee27ce4abd976d8863217445b0dfa9c9a2a921bd410bfc57ae43a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MRSD
  - MRSDexplorer
  - MendelianRNA-seq
  - Python
  - STAR
  - process_cummings_output.py
  - transcript_bed_generator.py
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# mrsd-sequencing-depth-prediction

## Summary

Apply the MRSD (Minimum Required Sequencing Depth) framework via MRSDexplorer to predict, per gene/transcript and per tissue dataset, the RNA-seq depth needed to achieve a user-specified splice-junction coverage level, and validate predictions with PPV/NPV.

## When to use

Use when planning bulk short-read poly-A RNA-seq for splice-junction surveillance of Mendelian disease genes and you need to know how many reads (e.g. --number_reads 8 over --sj_prop 0.75 junctions) a given tissue requires.

## When NOT to use

- The dataset is long-read, single-cell, or spatially resolved RNA-seq — no appropriate control datasets exist yet, and MRSD is validated only for bulk short-read poly-A enrichment data processed with the specified pipeline
- The goal is to predict suitability for allele-specific expression, differential gene expression, or isoform imbalance analysis — the article states the MRSD model cannot directly be extended to these
- The patient's disorder involves widespread transcriptomic changes (e.g. interferonopathies, chromatin structure disorders, spliceosome disruption) or strong eQTL/sQTL effects, which the current model does not account for

## Inputs

- *.csf.splicing.txt junction count files from the Cummings et al. split-read pipeline (FASTQ-derived, STAR-aligned)
- *.junc-counts.txt merged junction count files (SJID per-sample count matrix)
- *.seq-depths.txt sequencing depth files with sample_id, total_input, uniquely_mapping, multimapping columns (values from STAR *.Log.final.out)
- Gene list text file (GENCODE v19 gene symbol and/or Ensembl gene ID) or transcript introns *.introns.bed
- GENCODE human v19 GTF annotation (for transcript_bed_generator.py)

## Outputs

- *.results.mrsd.txt per-gene/transcript MRSD scores in millions of reads, stratified by control dataset
- *.junc-coverage.txt per-million-read junction coverage per sample (with --output_junctions)
- PPV/NPV validation table per parameter combination (e.g. mrsd_validation_ppv_npv.csv)

## How to apply

Prepare *.junc-counts.txt files by running the Cummings et al. split-read pipeline (scripts at berylc/MendelianRNA-seq) and merging *.csf.splicing.txt outputs with process_cummings_output.py; pair each count file with a *.seq-depths.txt file (columns sample_id, total_input, uniquely_mapping, multimapping, depths taken from STAR *.Log.final.out) whose sample IDs exactly match the counts file. Move both into MRSDexplorer's files/ directory, supply a gene list or transcript introns BED (generated from a GTF with transcript_bed_generator.py), and run MRSDexplorer.py over the parameter grid --number_reads, --sj_prop, --mrsd_param (percentile, default 0.95), --read_type (total, unique, or multimap; default unique). Rationale: MRSD scores are computed from control-sample junction count distributions (GTEx, Illumina 75 bp paired-end poly-A) and returned as millions of reads in *.results.mrsd.txt; --output_junctions additionally emits per-million-read junction coverage (*.junc-coverage.txt). To judge correctness, classify each gene/sample as covered or not covered at the predicted depth and compute PPV and NPV; on 68 independent samples (12 blood, 4 LCL, 52 muscle) expected mean PPV is 96% (range 90.1%-98.2%) and mean NPV 79% (range 56.4%-94.7%), reflecting a conservative model.

## Related tools

- **MRSDexplorer** (Python toolkit (bin/ scripts) used to compute MRSD scores from junction counts and sequencing depths over a parameter grid of --number_reads, --sj_prop, --mrsd_param, --read_type) — github.com/mcgm-mrsd/mrsd-explorer
- **process_cummings_output.py** (Merges and reformats Cummings pipeline *.csf.splicing.txt outputs into MRSDexplorer-compatible *.junc-counts.txt files) — github.com/mcgm-mrsd/mrsd-explorer
- **transcript_bed_generator.py** (Generates *.introns.bed files of transcript intron coordinates from a GTF and transcript ID list for transcript-level MRSD queries) — github.com/mcgm-mrsd/mrsd-explorer
- **MendelianRNA-seq** (Upstream scripts for generating transcriptome-wide junction counts from FASTQs via split-read alignment (Cummings et al. 2017 pipeline)) — github.com/berylc/MendelianRNA-seq
- **STAR** (Split-read aligner whose *.Log.final.out file provides the sample sequencing depths for *.seq-depths.txt)

## Examples

```
python MRSDexplorer.py --tissues all --number_reads 8 --sj_prop 0.75 --mrsd_param 0.95 --read_type unique --output_junctions --output_prefix example transcripts.introns.bed
```

## Evaluation signals

- Computed PPVs across the parameter grid fall within the reported 90.1%-98.2% precision range, with mean PPV ~96% on the 68-sample validation cohort (12 blood, 4 LCL, 52 muscle)
- Computed NPVs fall within 56.4%-94.7%, mean NPV ~79%, consistent with a conservative model
- Sample IDs in *.seq-depths.txt precisely match those in *.junc-counts.txt; IDs present in only one file are excluded from analysis
- *.results.mrsd.txt reports a non-trivial 'Number of genes successfully cross-referenced' and '-' only for genes whose median junction coverage was 0 reads
- Each *.seq-depths.txt has four ordered columns (sample_id, total_input, uniquely_mapping, multimapping) and passes input file validation

## Limitations

- Pre-computed GTEx-derived MRSDs are most accurate for Illumina 75 bp paired-end poly-A enrichment workflows; other sequencing methodologies require generating custom MRSD scores
- The model does not explicitly account for eQTLs or sQTLs, or for disorders with widespread transcriptomic changes
- MRSD cannot be directly extended to predict suitability for allele-specific expression or differential gene expression; long-read, single-cell, and spatial data lack control datasets for model development
- Sample IDs appearing in only one of the *.junc-counts.txt / *.seq-depths.txt pair are silently excluded from analysis

## Evidence

- [readme] MRSD scores derived from GTEx poly-A short-read data: "GTEx samples are sequenced using an Illumina 75 bp paired-end read poly-A enrichment workflow. As such, the pre-computed MRSDs are most accurate for assessing transcript coverage with similar RNA-seq"
- [readme] Sample ID matching requirement: "Importantly, the IDs in the sample_id column must precisely match those listed in the *output_prefix.junc-counts.txt* file. Sample IDs identified in only one of the two files will be excluded from"
- [readme] MRSDexplorer command with key parameters: "python MRSDexplorer.py --tissues all --number_reads 10 --sj_prop 0.8 --mrsd_param 0.9 --output_prefix example transcripts.introns.bed"
- [abstract] Validation precision of MRSD: "MRSD has high precision (90.1%-98.2%) and overcomes transcript region-specific sequencing biases"
- [methods] MRSD definition and purpose: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
