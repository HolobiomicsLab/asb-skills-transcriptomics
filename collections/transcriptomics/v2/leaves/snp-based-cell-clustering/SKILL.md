---
name: snp-based-cell-clustering
description: Use when you have a 10x scRNA-seq BAM (possorted_genome_bam.bam) plus
  barcodes.tsv from a multi-person mixture and need to assign each cell to its contributing
  individual based on genotype SNPs.
when_to_use_negative:
- Cells are not live (the 10x Genomics scRNA-seq platform requires live cells for
  successful genetic separation)
- Mixture contains fewer than ~150 cells without deeper sequencing — this has not
  been empirically confirmed
- Biological mixtures other than blood — transferability to other tissue types is
  not yet demonstrated
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3170
tools:
- name: De-goulash
  role: Snakemake pipeline implementing the two-iteration deconvolution (genotyping
    and clustering)
  repo: https://github.com/genid/de-goulash.git
- name: freebayes
  role: Haplotype-based SNP variant caller run in parallel over generated region files
  repo: https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py
- name: Snakemake
  role: Workflow engine executing deconvolution (step 1) and statistical analysis
    (step 2)
  repo: https://github.com/genid/de-goulash.git
- name: Docker
  role: geniderasmusmc/de-goulash:1 container image; working directory mounted to
    /single-cell
- name: R
  role: Clustering Rscript rule performing UMAP-based clustering (n_neighbors=5, n_components=300)
- name: Python
  role: Merging-cells rule applying coverage/base-calling thresholds to retain usable
    cells
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04557-z
    title: Single-cell transcriptome sequencing allows genetic separation, characterization
      and identification of individuals in multi-person biological mixtures
  source_content_hash: sha256:c7f8942e090e1ec6668ce34779beb7a27a219ded2aaa9c310d625cb5789d84b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genid/de-goulash.git
  tools:
  - De-goulash
  - Docker
  - Python
  - R
  - Snakemake
  - freebayes
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# snp-based-cell-clustering

## Summary

Two-iteration SNP-genotyping-and-clustering workflow (de-goulash) that separates cells from multi-person scRNA-seq mixtures by contributor: first using mtDNA-SNPs (freebayes over MT regions), then genome-wide SNPs, followed by UMAP-based clustering of cells into per-individual clusters.

## When to use

Use when you have a 10x scRNA-seq BAM (possorted_genome_bam.bam) plus barcodes.tsv from a multi-person mixture and need to assign each cell to its contributing individual based on genotype SNPs.

## When NOT to use

- Cells are not live (the 10x Genomics scRNA-seq platform requires live cells for successful genetic separation)
- Mixture contains fewer than ~150 cells without deeper sequencing — this has not been empirically confirmed
- Biological mixtures other than blood — transferability to other tissue types is not yet demonstrated

## Inputs

- possorted_genome_bam.bam (10x scRNA-seq alignment BAM, e.g. possorted_genome_trimmed.bam)
- barcodes.tsv / barcodes_reduced.txt (cells to use onward, from 10x output)
- genome.fasta (Human reference genome, e.g. hg19 or hg38)
- MT.fasta (mitochondrial DNA sequence, same build as genome)
- regions.txt and MT_regions.txt (generated with python process_reference.py [path/genome.fasta])
- config.yaml (input paths and pipeline parameters)

## Outputs

- output/iter2/cells_merge_clusters.vcf (variant calls for merged cell clusters)
- Per-cell contributor cluster assignments (cells separated by contributing individual)
- Fraction of cells separated per iteration per dataset

## How to apply

Populate config.yaml with sample_bam, barcodes, genome.fasta, MT.fasta and region files (regions.txt, MT_regions.txt generated with process_reference.py), then run de-goulash Snakemake iteration 1 (mtDNA-SNP based) in the geniderasmusmc/de-goulash:1 Docker container with freebayes variant calling in parallel, using dp=50, qual=60, thr_cell_1=10 minimal SNPs per cell. Apply the merging-cells Python rule to keep only cells with threshold_coverage=10 total coverage, threshold_coverage_pos=5 coverage per SNP, and threshold_base_calling=90; in the balanced two-person blood mixture M2 this mtDNA iteration separates ~21.3% of cells. Run iteration 2 (genome-wide-SNP based) targeting output/iter2/cells_merge_clusters.vcf with thr_cell_2=20 minimal SNPs per cell, then execute the clustering Rscript rule (UMAP, n_neighbors=5, n_components=300; nBclust only if clusters > 1) to assign cells to contributor clusters. Rationale: mtDNA provides high-copy initial separation while genome-wide SNPs refine clusters to near-complete separation (M2 ~97%, M3 ~96.5%, M4 ~98%).

## Related tools

- **De-goulash** (Snakemake pipeline implementing the two-iteration deconvolution (genotyping and clustering)) — https://github.com/genid/de-goulash.git
- **freebayes** (Haplotype-based SNP variant caller run in parallel over generated region files) — https://github.com/nh13/freebayes/blob/master/scripts/fasta_generate_regions.py
- **Snakemake** (Workflow engine executing deconvolution (step 1) and statistical analysis (step 2)) — https://github.com/genid/de-goulash.git
- **Docker** (geniderasmusmc/de-goulash:1 container image; working directory mounted to /single-cell)
- **R** (Clustering Rscript rule performing UMAP-based clustering (n_neighbors=5, n_components=300))
- **Python** (Merging-cells rule applying coverage/base-calling thresholds to retain usable cells)

## Examples

```
docker run -it -v /current/directory/de-goulash/:/single-cell geniderasmusmc/de-goulash:1 output/iter2/cells_merge_clusters.vcf --snakefile Snakefile --configfile config.yaml --cores 1
```

## Evaluation signals

- output/iter2/cells_merge_clusters.vcf is produced for each mixture
- Iteration-1 mtDNA separation of the balanced two-person blood mixture M2 is ~21.3% of cells
- Final genome-wide-iteration separation matches reported values: M2 ~97%, M3 ~96.5%, M4 ~98%
- All cells retained by the merging rule pass threshold_coverage=10, threshold_coverage_pos=5, and threshold_base_calling=90
- Cells retained at each iteration carry at least thr_cell_1=10 (iteration 1) or thr_cell_2=20 (iteration 2) SNPs per cell

## Limitations

- mtDNA-SNP iteration 1 alone separates only a minority of cells (~21.3% for M2); near-complete separation requires the second genome-wide-SNP iteration
- The 10x scRNA-seq platform requires live cells, restricting application to mixtures containing living cells; fixed-cell platforms are untested
- Deconvolution of more than nine individuals and from fewer than 150 cells with deeper sequencing has not been empirically tested
- ScSplit comparison on a two-person 5000-cell in silico mixture left 45.16% of donor-1 cells unassigned, showing tool choice affects assignment completeness

## Evidence

- [readme] Two-iteration workflow parameters: "thr_cell_1: 10 #Minimal number of SNPs per cell"
- [readme] Merging thresholds: "threshold_coverage: 10 #treshhold total coverage of selected SNPs per cell"
- [readme] Clustering settings: "n_neighbors: 5 #setting for UMAP clustering"
- [abstract] Separation result: "For all 2 up to 9-person balanced and imbalanced blood mixtures with ratios up to 1:60, we achieved a clear single-cell separation according to the contributing individuals."
- [discussion] Live-cell limitation: "requires live cells for successful genetic separation, which consequently limits the application to biological mixtures containing living cells"
- [methods] Task-card expected separation values: [evidence span withheld — anchor status: not_found]
