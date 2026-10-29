---
name: transcriptome-reconstruction-from-rnaseq
description: Use when you have genome-aligned (or alignable) RNA-Seq reads and need
  a sample-specific GTF transcriptome with accurate 3'-ends/polyA sites for downstream
  quantitation or APA analysis.
when_to_use_negative:
- You only need annotated, known transcripts and no sample-specific isoform or 3'-end
  discovery — use the guide annotation directly instead of assembling.
- You need de novo detection of coding alternative polyadenylation in upstream exons
  — aptardi only evaluates 3' terminal exons and does not analyze upstream exons.
- Your environment already has bioconda samtools/bedtools installed alongside aptardi,
  since the README warns these packages may prevent aptardi from working in the same
  conda environment.
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: HISAT2
  role: Spliced aligner used to produce genome-aligned BAM with --rna-strandness RF
    --dta for transcript reconstruction
- name: SAMtools
  role: Sorts the aligned BAM prior to StringTie and aptardi; v1.9 required by aptardi
  repo: http://www.htslib.org/download/
- name: StringTie
  role: Assembles the sample-specific transcriptome from the sorted BAM using the
    Ensembl guide annotation (--rf -G)
- name: aptardi
  role: Refines transcript 3'-ends and adds polyadenylation-supported transcripts
    to the StringTie GTF using RNA-Seq coverage plus DNA sequence via a pre-built
    machine-learning model
  repo: github.com/luskry/aptardi
- name: RSEM
  role: Quantitates the aptardi transcriptome so low-count transcripts can be filtered
    and the high-quality transcriptome re-quantitated
- name: BEDtools
  role: Dependency of aptardi (v2.29.2) for genomic interval operations during 3'-end
    annotation
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: FastQC
  role: Read quality assessment before alignment
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:e247dc2ad07a961457d90b3581037f3171a3145db5e6fd52f7606255069e4c7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - APARENT
  - BEDtools
  - FastQC
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - TAPAS
  - aptardi
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# transcriptome-reconstruction-from-rnaseq

## Summary

Reconstruct a sample-specific transcriptome from RNA-Seq by aligning reads with HISAT2, assembling transcripts with StringTie guided by an Ensembl annotation, and optionally refining transcript 3'-ends with aptardi before RSEM quantitation.

## When to use

Use when you have genome-aligned (or alignable) RNA-Seq reads and need a sample-specific GTF transcriptome with accurate 3'-ends/polyA sites for downstream quantitation or APA analysis.

## When NOT to use

- You only need annotated, known transcripts and no sample-specific isoform or 3'-end discovery — use the guide annotation directly instead of assembling.
- You need de novo detection of coding alternative polyadenylation in upstream exons — aptardi only evaluates 3' terminal exons and does not analyze upstream exons.
- Your environment already has bioconda samtools/bedtools installed alongside aptardi, since the README warns these packages may prevent aptardi from working in the same conda environment.

## Inputs

- RNA-Seq FASTQ reads (paired-end, strand-specific)
- Reference genome FASTA (e.g. hg38.fa or mm10)
- Guide transcript annotation GTF (e.g. Ensembl hg38 or mm10 GTF)
- Sorted, genome-aligned BAM file from HISAT2
- Pre-built aptardi model files (model.hdf5 and scale.pk) for 3'-end refinement

## Outputs

- StringTie-reconstructed transcriptome GTF (stringtie.gtf)
- aptardi-modified transcriptome GTF with refined 3'-ends and added aptardi transcripts
- RSEM quantitation of the transcriptome
- Filtered high-quality transcriptome (transcripts with >=1 count in 2/3 of samples)

## How to apply

Assess read quality with FastQC, align reads to the reference genome with HISAT2 using strand-specific settings (--rna-strandness RF --dta), and sort the BAM with SAMtools. Reconstruct the transcriptome with StringTie in guided mode ('stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>') using an Ensembl GTF as the guide, since StringTie assembles a minimum path RNA-Seq cover to zero read coverage, which can truncate 3'-ends. To refine polyA sites, run aptardi in Mode 1 with the pre-built model (model.hdf5 and scale.pk from src/aptardi/ml_scale), the sorted BAM, genome FASTA (e.g. hg38.fa), and the StringTie GTF, keeping the default 100-base bins and 0.5 probability threshold; aptardi adds transcripts to (never removes transcripts from) the input GTF. Quantitate the aptardi-modified transcriptome with RSEM and build a high-quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples, then re-quantitate with RSEM for downstream differential expression.

## Related tools

- **HISAT2** (Spliced aligner used to produce genome-aligned BAM with --rna-strandness RF --dta for transcript reconstruction)
- **SAMtools** (Sorts the aligned BAM prior to StringTie and aptardi; v1.9 required by aptardi) — http://www.htslib.org/download/
- **StringTie** (Assembles the sample-specific transcriptome from the sorted BAM using the Ensembl guide annotation (--rf -G))
- **aptardi** (Refines transcript 3'-ends and adds polyadenylation-supported transcripts to the StringTie GTF using RNA-Seq coverage plus DNA sequence via a pre-built machine-learning model) — github.com/luskry/aptardi
- **RSEM** (Quantitates the aptardi transcriptome so low-count transcripts can be filtered and the high-quality transcriptome re-quantitated)
- **BEDtools** (Dependency of aptardi (v2.29.2) for genomic interval operations during 3'-end annotation) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **FastQC** (Read quality assessment before alignment)

## Examples

```
stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file> && aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --g hbr_aptardi.gtf --n model.hdf5 --t scale.pk --o output_dir
```

## Evaluation signals

- The reconstructed transcriptome GTF contains the original guide/StringTie transcripts plus aptardi-added transcripts (aptardi never removes input transcripts).
- Refined 3'-ends fall within 100 bases of true PolyA-Seq sites for the majority of high-quality transcripts (e.g. 62,688 HBR transcripts vs 22,804 with TAPAS in the benchmark).
- Positive predictive value of 3'-termini improves relative to comparator assemblers/tools (0.44 vs 0.31 for aptardi vs TAPAS on HBR).
- Post-filtering, retained transcripts have at least 1 count in 2/3 of samples when quantitated with RSEM.
- Mode 1 runs use the pre-built model with 100-base bins and probability threshold 0.5; altering --w with the pre-built model is not recommended.

## Limitations

- aptardi makes predictions on 100-base bins, a resolution limitation, since polyA sites can wiggle up to 30 nucleotides.
- aptardi does not distinguish transcripts overlapping the same genomic region and may attribute a single 3'-end to multiple transcript isoforms of a gene, possibly annotating a stop site on the wrong transcript.
- Upstream exons are not analyzed, so coding alternative polyadenylation cannot be identified.
- Manually engineered DNA sequence features may not apply to taxa outside mammals; the pre-built model should not be used with altered bin sizes (--w).
- Detection of polyA sites is modest at low sequencing depths (as in the mouse tissue analysis).

## Evidence

- [methods] stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>: "stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>"
- [methods] HISAT2 alignment command: "(hisat2 -q -p 5 --reorder -t --rna-strandness RF --dta -x <hisat2_index> -1"
- [methods] Filter rule for high-quality transcriptome: "Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples"
- [abstract] aptardi combines RNA-Seq and DNA sequence: "as input aptardi takes DNA nucleotide sequence, genome-aligned RNA-Seq data, and an initial transcriptome"
- [readme] aptardi output behavior: "Aptardi analyzes the input gtf file and outputs a new gtf file with aptardi transcripts added along with original transcripts, i.e. aptardi does not remove any transcripts"
- [readme] Bioconda dependency conflict warning: "The bioconda samtools and bedtools packages may prevent aptardi from working, please make sure these packages are not installed in the same environment as aptardi"
