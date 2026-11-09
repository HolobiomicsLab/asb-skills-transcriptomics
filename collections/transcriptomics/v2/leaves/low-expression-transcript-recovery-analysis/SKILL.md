---
name: low-expression-transcript-recovery-analysis
description: Use when benchmarking a genome-guided transcriptome assembler (e.g. Tiglon
  vs StringTie2/Scallop) on RNA-seq alignments and you must show whether gains come
  specifically from recovering lowly expressed reference transcripts.
when_to_use_negative:
- Input data is long-read RNA-seq (PacBio or ONT) — the current Tiglon version is
  not compatible with long-read datasets
- The analysis goal is annotation-guided assembly — the current Tiglon version has
  no option for using known transcript annotations to guide assembly
- Only a single aligner's BAM is available — Tiglon is designed to integrate alignments
  of different mapping tools (Hisat2 and Star), so single-aligner input forfeits the
  labeled-splice-graph advantage being evaluated
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Tiglon
  role: Candidate assembler whose low-expression recovery is measured; integrates
    Hisat2 and Star alignments into labeled splice graphs
  repo: https://github.com/yutingsdu/Tiglon
- name: Scallop
  role: Baseline assembler run on Star alignments for low-expression recovery comparison
  repo: https://github.com/Kingsford-Group/scallop
- name: StringTie2
  role: Baseline assembler; StringTie2 --merge provides the merged-assembly baseline
  repo: https://github.com/gpertea/stringtie
- name: Hisat2
  role: Produces one of the two splice-aware BAM alignment sets (with --dta) consumed
    by Tiglon
  repo: https://github.com/DaehwanKimLab/hisat2
- name: Star
  role: Produces the second splice-aware BAM alignment set consumed by Tiglon
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:1bcce749712550f7bff4602a7befcd4bea667cc834e60b88930ca9d7de08f7b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yutingsdu/Tiglon
  tools:
  - Hisat2
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# low-expression-transcript-recovery-analysis

## Summary

Quantify how many lowly expressed reference transcripts an assembler recovers by stratifying Cuffcompare evaluation results against the reference annotation (e.g. mm10.ncbiRefSeq.gtf or hg38.ncbiRefSeq.gtf) by expression level. In the Tiglon paper this analysis is what reveals Tiglon's key advantage: it is 'significantly powerful in recovering lowly expressed transcripts' compared with StringTie2, Scallop, and StringTie2-Merge.

## When to use

Use when benchmarking a genome-guided transcriptome assembler (e.g. Tiglon vs StringTie2/Scallop) on RNA-seq alignments and you must show whether gains come specifically from recovering lowly expressed reference transcripts.

## When NOT to use

- Input data is long-read RNA-seq (PacBio or ONT) — the current Tiglon version is not compatible with long-read datasets
- The analysis goal is annotation-guided assembly — the current Tiglon version has no option for using known transcript annotations to guide assembly
- Only a single aligner's BAM is available — Tiglon is designed to integrate alignments of different mapping tools (Hisat2 and Star), so single-aligner input forfeits the labeled-splice-graph advantage being evaluated

## Inputs

- Assembled transcript GTF files from each assembler (e.g. Tiglon.gtf, StringTie2 GTF, Scallop GTF)
- Reference annotation GTF with expression level information (mm10.ncbiRefSeq.gtf or hg38.ncbiRefSeq.gtf)
- Splice-aware BAM alignments from Hisat2 (--dta) and STAR, or a bamFile_list combining both aligners
- Cuffcompare evaluation output (correct transcript matches per assembler)

## Outputs

- Per-assembler low-expression transcript recovery counts stratified by reference transcript expression level
- Percent improvement of low-expression recovery of Tiglon over baseline assemblers (e.g. 108.64%–139.58% vs Scallop+Star, 25.36%–39.91% vs StringTie2-Merge)
- Aggregated benchmark table (e.g. mouse_benchmark.csv / human_benchmark.csv) with correct transcripts, precision, and F-score per assembler

## How to apply

Run the candidate assemblers (Tiglon on combined Hisat2+Star BAMs, plus baselines such as Scallop on Star alignments and StringTie2-Merge) with their default filters — for Tiglon keep min_trans_length 500, min_average_frac 0.03, min_unbalance_frac 0.03, min_gap_length 200, since tightening these thresholds would directly suppress low-expression assemblies. Evaluate each resulting GTF against the matching RefSeq annotation (mm10.ncbiRefSeq.gtf for mouse GRCm38/mm10, hg38.ncbiRefSeq.gtf for human GRCh38) using Cuffcompare to obtain correct-transcript matches. Then stratify the reference transcripts by expression level and count, per expression bin, how many reference transcripts each assembler reconstructed, expressing the comparison as percent improvement of low-expression recovery (Tiglon reported 108.64%–139.58% over Scallop+Star and 25.36%–39.91% over StringTie2-Merge on mouse). The rationale is that integrating alignments from multiple aligners (labeled splice graphs) restores junction coverage that a single aligner misses, which disproportionately benefits weakly expressed transcripts with sparse read support.

## Related tools

- **Tiglon** (Candidate assembler whose low-expression recovery is measured; integrates Hisat2 and Star alignments into labeled splice graphs) — https://github.com/yutingsdu/Tiglon
- **Scallop** (Baseline assembler run on Star alignments for low-expression recovery comparison) — https://github.com/Kingsford-Group/scallop
- **StringTie2** (Baseline assembler; StringTie2 --merge provides the merged-assembly baseline) — https://github.com/gpertea/stringtie
- **Hisat2** (Produces one of the two splice-aware BAM alignment sets (with --dta) consumed by Tiglon) — https://github.com/DaehwanKimLab/hisat2
- **Star** (Produces the second splice-aware BAM alignment set consumed by Tiglon) — https://github.com/alexdobin/STAR

## Examples

```
Tiglon -B bamFiles_list -s first -o Tiglon_outdir -p 2
```

## Evaluation signals

- Cuffcompare against mm10.ncbiRefSeq.gtf yields Tiglon average precision ≈51.71% and average F-score ≈0.2341 on the four mouse samples
- Low-expression recovery improvement of Tiglon (Hisat2+Star) over Scallop (Star) falls in the 108.64%–139.58% range and over StringTie2-Merge in the 25.36%–39.91% range in mouse_benchmark.csv
- On the eight human datasets, Tiglon recovers ~9.51% more correct transcripts than StringTie2-Merge and 11.25%–13.68% more than StringTie2/Scallop in human_benchmark.csv
- Every assembler was run with default filters (e.g. Tiglon min_trans_length 500, min_average_frac 0.03) so low-expression losses are attributable to the algorithm, not to threshold choices
- Each assembled output is a valid GTF whose transcript structures can be matched to reference transcripts by Cuffcompare

## Limitations

- The current Tiglon version has no option for annotation-guided assemblies, so known annotations cannot be used to boost low-expression recovery
- Tiglon is not compatible with long-read RNA-seq datasets (e.g. PacBio or ONT), limiting the analysis to short-read alignments
- Reported recovery-gain ranges are dataset-specific (8 human and 4 mouse samples plus one simulated dataset) and may not transfer to other tissues, depths, or species

## Evidence

- [abstract] Especially, Tiglon is significantly powerful in recovering lowly expressed transcripts.: "Especially, Tiglon is significantly powerful in recovering lowly expressed transcripts."
- [methods] Default filter thresholds suppress low-expression recovery if tightened: "--min_average_frac/-d <float> 	: Minimum junction coverage fraction by average junction coverage, default: 0.03."
- [readme] Tiglon input/output format: "The software takes a file that lists alignment files generated by different aligners as input, and outputs all assembled candidate transcripts in GTF format."
- [intro] Motivating observation justifying the integrating strategy behind low-expression gains: "both the Hisat2 and Star (two of the best current aligners) alignments cover the whole exons of this transcript, but neither of them covers the entire junctions"
