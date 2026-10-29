---
name: synthetic-genotype-data-generation
description: Use when you need controlled synthetic GWAS data (SNP matrix × binary
  phenotype) with known planted associations to benchmark pattern-mining methods such
  as HOGImine, SiNIMin or FastCMH under FWER control.
when_to_use_negative:
- The goal is to evaluate on real experimental data (use the shipped data/athaliana
  or data/mmusculus datasets instead, e.g. data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt)
- The task is mining/significance testing itself rather than generating benchmark
  data — compile and run HOGImine/SiNIMin/FastCMH directly on existing files
- The analysis requires continuous (non-genotype) or non-binary phenotype data; HOGImine
  expects binary labels and binary/additive marker encodings
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: networkx
  role: Samples the Erdos-Renyi G(75,100) gene-interaction prior graph and draws the
    random connected causal subgraph
- name: HOGImine (hogimine_additive)
  role: Smoke-run validation of the generated files (with -f 0.05) to confirm they
    parse as valid HOGImine inputs
  repo: https://github.com/BorgwardtLab/HOGImine
- name: make
  role: Compiles hogimine_additive (gcc-12 toolchain) before the validation smoke
    run
  repo: https://github.com/BorgwardtLab/HOGImine
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:60e976e7e770af032388b61ef9d2d6677a79bd35cb6225a9e67afa8d0e7fb6b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - FastCMH
  - HOGImine
  - HOGImine (hogimine_additive)
  - SiNIMin
  - gcc-12
  - make
  - networkx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# synthetic-genotype-data-generation

## Summary

Generates synthetic binary- and additive-encoded genotype datasets with a planted, ground-truth higher-order association (a random connected subgraph of a gene-interaction network) for power-comparison studies of HOGImine against SiNIMin and FastCMH. Produces all six HOGImine input files plus a ground-truth record of the planted meta-marker.

## When to use

Use when you need controlled synthetic GWAS data (SNP matrix × binary phenotype) with known planted associations to benchmark pattern-mining methods such as HOGImine, SiNIMin or FastCMH under FWER control.

## When NOT to use

- The goal is to evaluate on real experimental data (use the shipped data/athaliana or data/mmusculus datasets instead, e.g. data/mmusculus/BMC.Mode/BMC.Mode_X_T.txt)
- The task is mining/significance testing itself rather than generating benchmark data — compile and run HOGImine/SiNIMin/FastCMH directly on existing files
- The analysis requires continuous (non-genotype) or non-binary phenotype data; HOGImine expects binary labels and binary/additive marker encodings

## Inputs

- Generator parameters: number of nodes/edges for the prior graph (75 nodes, 100 edges), SNPs per gene ~ U[3,10], n.samples = 3000, association strength rho, encoding choice (binary dominant/recessive or additive), covariate range C
- Choice of causal module (random connected subgraph of the sampled gene-interaction network)

## Outputs

- Marker matrix file for -i (n.SNPs rows × 3000 sample columns, binary or additive 0/1/2 values)
- Binary label matrix for -l (n.samples × 1)
- SNP name file for -s
- Covariate file for -c (integer class in {1,...,C} per sample)
- SNP-to-gene map file for -m (gene name followed by its mapped SNP names)
- Gene interaction sets file for -e (the Erdos-Renyi prior)
- Ground-truth file recording the genes, encodings, and SNP positions of the planted meta-marker

## How to apply

Sample an Erdos-Renyi G(n,m) random graph with 75 nodes and exactly 100 edges (e.g. networkx gnm_random_graph) to serve as the gene-interaction prior written to the -e file. Assign each gene a number of SNPs drawn from U[3,10], simulate genotypes for 3000 samples in binary (dominant/recessive per-SNP annotation) and/or additive encoding (0/1/2 allele counts), add a binary phenotype label column and an integer covariate column in {1,...,C}. Draw a random connected subgraph of the network as the causal module and inject phenotype association with tunable strength rho into the per-gene SNP intervals it spans; record the planted genes, encodings, and SNP positions in a ground-truth file. Serialize all six HOGImine-format files: marker matrix (-i, n.SNPs rows × 3000 sample columns), binary labels (-l), SNP names (-s), covariates (-c), SNP-to-gene map (-m, gene name followed by its mapped SNPs), and gene interaction sets (-e). Varying rho and the encoding lets you sweep association strength for power curves (Figure 2) and encoding-loss experiments (Figure 3); always record the true per-SNP encoding because mis-specified binarization (e.g. dominant encoding on recessive signal) destroys power.

## Related tools

- **networkx** (Samples the Erdos-Renyi G(75,100) gene-interaction prior graph and draws the random connected causal subgraph)
- **HOGImine (hogimine_additive)** (Smoke-run validation of the generated files (with -f 0.05) to confirm they parse as valid HOGImine inputs) — https://github.com/BorgwardtLab/HOGImine
- **make** (Compiles hogimine_additive (gcc-12 toolchain) before the validation smoke run) — https://github.com/BorgwardtLab/HOGImine

## Examples

```
import networkx as nx; G = nx.gnm_random_graph(75, 100, seed=42)  # gene-interaction prior for the -e file
```

## Evaluation signals

- All six generated files parse as valid HOGImine inputs
- The marker matrix has 3000 sample columns with binary/additive values and labels are binary n.samples × 1
- The planted causal module is a connected subgraph of the -e interaction prior network
- The ground-truth file records the genes, encodings, and SNP positions of the planted meta-marker
- A smoke run of hogimine_additive on the synthetic data with -f 0.05 executes without error; at sufficient rho, additive mode recovers more significant patterns and a smaller smallest CMH p-value than a mis-specified binary mode

## Limitations

- Binarization must match the true encoding: using the wrong (dominant) encoding for recessive-associated data causes a serious loss of statistical power, so the ground-truth encoding annotation is essential
- The planted signal is restricted to a connected subgraph of the sampled prior network; associations outside it are not modeled
- Generator scale is fixed by design (75-node prior graph, U[3,10] SNPs per gene, 3000 samples); other scales need re-validation

## Evidence

- [readme] binary/additive encodings supported by HOGImine: "HOGImine accepts both binary encodings and additive encodings for the markers."
- [readme] marker file format the generator must emit: "- ```-i file```: marker (binary or additive encoding) file. It should be matrix of size $n. SNPs \times n. samples$"
- [intro] why synthetic meta-markers are planted and mined: "is a pattern-mining-based algorithm for finding genetic meta-markers, i.e. combinations of genetic markers, that show a statistical association with a phenotype"
- [abstract] prior biological knowledge restricts the search space: "Our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways, and protein complexes, to restrict its search space."
