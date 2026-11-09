---
name: polyadenylation-site-prediction
description: Use when a sample-specific transcriptome (e.g. StringTie GTF) needs accurate
  3'-end/polyA site annotation from genome-aligned RNA-Seq plus genome FASTA, and
  downstream APA or quantitation analyses require refined transcript termini.
when_to_use_negative:
- When the goal is detecting coding APA within upstream exons — aptardi evaluates
  only the 3' terminal exons of input transcripts and does not analyze upstream exons
  or intron junctions.
- When input transcriptomes require de novo transcript structure discovery from RNA-Seq
  alone — aptardi requires an initial transcriptome (GTF/GFF) and refines 3'-ends
  rather than assembling transcripts.
- When only DNA sequence is available — aptardi requires both sample RNA-Seq coverage
  and DNA sequence; DNA-sequence-only predictors cannot capture sample-specific expression.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: aptardi
  role: Predicts polyA sites by machine learning on 100-base bins of 3' terminal exons
    using RNA-Seq coverage plus DNA sequence; outputs an aptardi-modified GTF transcriptome
  repo: https://github.com/luskry/aptardi
- name: HISAT2
  role: Aligns paired-end RNA-Seq reads to the genome with --rna-strandness RF and
    --dta to produce the sorted BAM input for aptardi
- name: SAMtools
  role: Sorts the HISAT2 alignments into the sorted.bam required by aptardi; must
    not be installed via bioconda in the aptardi environment
- name: StringTie
  role: Reconstructs the sample-specific input transcriptome (stringtie.gtf) with
    guide annotation (--rf -G) that aptardi evaluates for 3'-end refinement
- name: RSEM
  role: Quantitates the aptardi transcriptome to enable the recommended 2/3-sample
    count filter for removing redundant aptardi transcripts
- name: TAPAS
  role: Comparator tool for 3'-end annotation accuracy benchmarking against aptardi
    on HBR
  repo: https://github.com/arefeen/TAPAS
- name: APARENT
  role: Deep-learning comparator for polyA signal prediction in benchmarking aptardi
  repo: https://github.com/johli/aparent
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_007
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:196c5223e2da51529f385a188a1eab7142eb0a782656fa97696fd4257e932766
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - APARENT
  - BEDtools
  - Cutadapt
  - FastQC
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - TAPAS
  - aptardi
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# polyadenylation-site-prediction

## Summary

Predict sample-specific polyadenylation (polyA) sites by combining RNA-Seq coverage with DNA sequence features using aptardi, a supervised machine-learning tool that refines transcript 3'-ends into a modified GTF transcriptome.

## When to use

Use when a sample-specific transcriptome (e.g. StringTie GTF) needs accurate 3'-end/polyA site annotation from genome-aligned RNA-Seq plus genome FASTA, and downstream APA or quantitation analyses require refined transcript termini.

## When NOT to use

- When the goal is detecting coding APA within upstream exons — aptardi evaluates only the 3' terminal exons of input transcripts and does not analyze upstream exons or intron junctions.
- When input transcriptomes require de novo transcript structure discovery from RNA-Seq alone — aptardi requires an initial transcriptome (GTF/GFF) and refines 3'-ends rather than assembling transcripts.
- When only DNA sequence is available — aptardi requires both sample RNA-Seq coverage and DNA sequence; DNA-sequence-only predictors cannot capture sample-specific expression.

## Inputs

- Sorted BAM of genome-aligned RNA-Seq reads (e.g. HISAT2-aligned, SAMtools-sorted)
- Genome FASTA with chromosome headers (e.g. hg38.fa, unzipped)
- Input transcriptome in GTF/GFF format (designed for StringTie output)
- Pre-built model file model.hdf5 and scale file scale.pk from src/aptardi/ml_scale (Mode 1)

## Outputs

- Modified transcriptome GTF/GFF (name_aptardi.gtf) containing aptardi transcripts added on top of the unmodified input transcriptome
- High-quality filtered transcriptome after RSEM quantitation and 2/3-sample count filtering
- Per-sample polyA site 3'-end coordinates for APA comparisons

## How to apply

Install aptardi via conda (conda create -n <env> --channel conda-forge -c bioconda aptardi), ensuring bioconda samtools and bedtools are NOT in the same environment. Align reads with HISAT2 (--rna-strandness RF --dta for firststrand libraries), sort with SAMtools, and reconstruct the transcriptome with StringTie using a guide annotation. Run aptardi in Mode 1 with the pre-built HBR model (model.hdf5) and scale.pk from src/aptardi/ml_scale, keeping canonical defaults: 100-base bins (--w 100, do not alter with the pre-built model), max 300 bins per transcript (--i 300 = 30,000 bases), probability threshold 0.5 (--p 0.5), and fr mate orientation (--a fr). aptardi bins only the 3' terminal exons of input transcripts into 100-base increments, computing 27 features per bin (14 RNA-Seq, 12 DNA-sequence, 1 transcriptome-derived), and annotates the 3'-most base of positive-prediction bins as the transcript stop site. Because aptardi annotates genomic regions, filter outputs by quantitating the aptardi transcriptome with RSEM and removing transcripts without at least 1 count in 2/3 of samples to eliminate redundant false-positive aptardi transcripts.

## Related tools

- **aptardi** (Predicts polyA sites by machine learning on 100-base bins of 3' terminal exons using RNA-Seq coverage plus DNA sequence; outputs an aptardi-modified GTF transcriptome) — https://github.com/luskry/aptardi
- **HISAT2** (Aligns paired-end RNA-Seq reads to the genome with --rna-strandness RF and --dta to produce the sorted BAM input for aptardi)
- **SAMtools** (Sorts the HISAT2 alignments into the sorted.bam required by aptardi; must not be installed via bioconda in the aptardi environment)
- **StringTie** (Reconstructs the sample-specific input transcriptome (stringtie.gtf) with guide annotation (--rf -G) that aptardi evaluates for 3'-end refinement)
- **RSEM** (Quantitates the aptardi transcriptome to enable the recommended 2/3-sample count filter for removing redundant aptardi transcripts)
- **TAPAS** (Comparator tool for 3'-end annotation accuracy benchmarking against aptardi on HBR) — https://github.com/arefeen/TAPAS
- **APARENT** (Deep-learning comparator for polyA signal prediction in benchmarking aptardi) — https://github.com/johli/aparent

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --n model.hdf5 --t scale.pk --g name_aptardi.gtf --o output_dir
```

## Evaluation signals

- Output name_aptardi.gtf parses as valid GTF/GFF and contains both the original input transcripts and added aptardi transcripts (aptardi removes no transcripts).
- Predicted 3' ends fall at the 3'-most base of positive 100-base bins called at the default 0.5 probability threshold.
- Cross-dataset validation: average precision of the HBR-trained model should be ~0.65 on HBR and 0.55–0.63 on external datasets (2nd HBR, UHR, BNLx, SHR), with F-measures 0.56/0.51/0.53/0.47/0.48.
- Benchmark recovery: 62,688 HBR transcripts with 3' termini within 100 bases of true PolyA-Seq sites (vs 22,804 for TAPAS), PPV 0.44 vs 0.31, and 82% (22,846/27,853) of added aptardi transcripts matching true sites.
- After RSEM quantitation and the 2/3-sample filter, redundant aptardi transcripts (same region probed for multiple isoforms sharing 3' terminal exons) are reduced.

## Limitations

- 100-base bin resolution: for positive predictions aptardi annotates the 3'-most base of the bin, though polyA sites can wiggle up to ~30 nucleotides.
- aptardi incorporates a single 3'-end into multiple transcripts per gene because it does not distinguish transcripts overlapping the same genomic region, possibly attributing a stop site to the wrong transcript.
- Manually engineered DNA sequence features may not apply to taxa outside of mammals.
- Bioconda samtools and bedtools packages in the same conda environment may prevent aptardi from working.

## Evidence

- [abstract] aptardi combines RNA-Seq and DNA sequence to identify polyA sites: "aptardi (alternative polyadenylation transcriptome analysis from RNA-Seq data and DNA sequence information), which leverages both DNA sequence and RNA sequencing in a machine"
- [readme] aptardi evaluates only 3' terminal exons: "aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly."
- [readme] Output GTF retains all input transcripts: "outputs a new gtf file with aptardi transcripts added along with original transcripts, i.e. aptardi does not remove any transcripts from the input transcriptome."
- [methods] 2/3-sample filter for high-quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [discussion] 100-base bin resolution limitation: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [readme] Do not alter bin size with pre-built model: "we DO NOT recommend altering this argument using the pre-built model since this model was built using 100 base bins"
