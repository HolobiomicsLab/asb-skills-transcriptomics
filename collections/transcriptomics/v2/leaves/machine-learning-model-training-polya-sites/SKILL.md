---
name: machine-learning-model-training-polya-sites
description: Use when building a custom polyA-site prediction model from DNA sequence
  plus RNA-Seq coverage for a sample-specific transcriptome, or when ablation-testing
  feature sets (multi-omics vs DNA-only vs RNA-only).
when_to_use_negative:
- You only need to predict polyA sites with the pre-built HBR model — use aptardi
  Mode 1 (--n model.hdf5 --t scale.pk) instead of retraining in Mode 2.
- Your goal is de novo transcriptome assembly or intron-junction discovery — aptardi
  only evaluates 3' terminal exons and does not evaluate intron junctions.
- You need single-nucleotide polyA site resolution or upstream coding-region APA —
  aptardi predicts on 100-base bins and does not analyze upstream exons.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: aptardi
  role: Trains the biLSTM polyA-site classifier in Mode 2 (--m) from binned DNA sequence
    + RNA-Seq coverage, saving model HDF5 and scale pickle artifacts
  repo: https://github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons
    are binned for feature extraction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:c8d59e9192000d6f21a655f6a1beb92f505e17cd1cf148460a480f020089e7bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# machine-learning-model-training-polya-sites

## Summary

Train a supervised biLSTM classifier (aptardi Mode 2) that predicts sample-specific polyadenylation sites from binned DNA sequence and RNA-Seq coverage features over 3' terminal exons, saving reusable Keras HDF5 model and pickle scale artifacts.

## When to use

Use when building a custom polyA-site prediction model from DNA sequence plus RNA-Seq coverage for a sample-specific transcriptome, or when ablation-testing feature sets (multi-omics vs DNA-only vs RNA-only).

## When NOT to use

- You only need to predict polyA sites with the pre-built HBR model — use aptardi Mode 1 (--n model.hdf5 --t scale.pk) instead of retraining in Mode 2.
- Your goal is de novo transcriptome assembly or intron-junction discovery — aptardi only evaluates 3' terminal exons and does not evaluate intron junctions.
- You need single-nucleotide polyA site resolution or upstream coding-region APA — aptardi predicts on 100-base bins and does not analyze upstream exons.

## Inputs

- Sorted BAM file of genome-aligned RNA-Seq reads (e.g. sorted.bam from HISAT2/SAMtools)
- Reference genome FASTA (e.g. hg38.fa; headers must be chromosomes)
- Input transcriptome GTF/GFF (e.g. stringtie.gtf from StringTie)
- Tab-separated gold-standard polyA sites file (e.g. polya_sites.bed) with known chromosome/site/strand column positions

## Outputs

- Trained Keras HDF5 model file (name_model.hdf5) per feature-set condition
- Pickled feature-scaler file (name_scale.pk) per condition
- Test-split metrics: average precision, precision, recall, F-measure (e.g. multi-omics AP = 0.58, F = 0.51)

## How to apply

Build the HBR feature matrix by binning DNA sequence (hg38.fa) and sorted.bam RNA-Seq read coverage into 100-base bins (--w default 100) over 3' terminal exons of the stringtie.gtf transcriptome, capped at 300 bins per transcript (--i default, ~30,000 bases). Load gold-standard polyA sites from a tab-separated polya_sites.bed using 0-based column indices given via --l (e.g. 0,4,1 for chromosome, site, strand) and train in Mode 2 (--m) with a fixed seed (--c) for reproducibility. The command form is aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --m --s polya_sites.bed --l 0,4,1 --c <seed> --e name_model.hdf5 --k name_scale.pk --o output_dir. Repeat training under each ablated feature-set condition (DNA-only, RNA-only, remove-canonical-PAS, canonical-PAS-only), saving a separate model and scale per condition. Judge success on a held-out test split by average precision, precision, recall, and F-measure; the paper reported multi-omics AP = 0.58 (F = 0.51) versus DNA-only AP = 0.41 and RNA-only AP = 0.44.

## Related tools

- **aptardi** (Trains the biLSTM polyA-site classifier in Mode 2 (--m) from binned DNA sequence + RNA-Seq coverage, saving model HDF5 and scale pickle artifacts) — https://github.com/luskry/aptardi
- **StringTie** (Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons are binned for feature extraction)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --m --s polya_sites.bed --l 0,4,1 --c 42 --e name_model.hdf5 --k name_scale.pk --o output_dir
```

## Evaluation signals

- Test-split average precision reproduces reported values: multi-omics AP = 0.58 (F = 0.51), DNA-only AP = 0.41, RNA-only AP = 0.44, no-canonical-PAS AP = 0.52, canonical-PAS-only AP = 0.54
- Saved artifacts load as a Keras HDF5 model (name_model.hdf5) and a pickle scaler (name_scale.pk)
- Similar AP between training and testing sets, indicating the model is not overfitting
- Bin parameters match defaults: --w 100 (bin size) and --i 300 (max bins per transcript, i.e. 30,000 bases of 3' terminal exon analyzed)
- Fixed seed (--c) yields reproducible model builds across repeated runs

## Limitations

- Predictions are on 100-base bins (annotating the 3'-most base of a positive bin), a resolution limitation since polyA sites wiggle up to 30 nucleotides.
- The multi-omics model outperformed single-omics ablations, but DNA-only and RNA-only variants were not equivalent substitutes (AP 0.41/0.44 vs 0.58).
- Manually engineered DNA sequence features may not apply outside mammals and require further research.
- Bioconda samtools and bedtools packages in the same conda environment may prevent aptardi from working.

## Evidence

- [readme] Mode 2 training command: "--m/-m <machine learning mode>	Enables Mode 2, building your own model, requires reliable genomic locations of polyA sites as the gold standard labels to train model"
- [readme] gold standard label columns: "--l/-l <int,int,int>		0-based coordinates of chromosome, strand, and site columns in polyA sites file (comma separated list with no spaces)"
- [readme] bin parameters: "--i/-i <int>			Maximum length analyzed per transcript (default: 300, which is number of bins analyzed, i.e 300 and bin size of 100 = 300 * 100"
- [intro] training data composition: "we trained aptardi to detect polyA sites from DNA nucleotide sequence and RNA-Seq read coverage by training on polyA sites identified by 3′ sequencing"
- [discussion] no overfitting check: "the similarity of the AP estimates between training and testing sets demonstrate that aptardi is not prone to overfitting"
- [discussion] bin resolution limitation: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
