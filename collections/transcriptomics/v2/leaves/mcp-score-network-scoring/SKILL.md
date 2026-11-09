---
name: mcp-score-network-scoring
description: Use when you need to reverse engineer a gene regulatory network from
  a gene expression matrix (HDF5/CSV/EXP) and quantify network quality via AUPRC/AUROC
  against a ground-truth interaction file, especially at genome scale where speed
  matters.
when_to_use_negative:
- When no ground-truth interaction matrix is available and you only need an unsupervised
  network — the AUPR/AUROC reporting requires `-x <groundtruth>.csv`
- When only partial ground truth is available and you need the ensemble-method train/test
  evaluation — use the `eval_mcpnet` binary instead of `mcpnet`
- When you only need pairwise association scores (Pearson correlation or mutual information)
  without MCP-score-based indirect-interaction correction — the standalone `pearson`
  or `mi` binaries suffice
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: MCPNet
  role: 'Primary pipeline: computes MI, MCP scores, and AUPR/AUROC metrics from an
    expression matrix; run via the `mcpnet` executable with -m 1 2 3 and --mi-method
    1'
  repo: https://github.com/AluruLab/MCPNet
- name: C++
  role: Language in which MCPNet is implemented; build with a modern C++ compiler
    (GNU g++, clang++, or Intel icpc) with C++11 support via cmake and make
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad373
    title: 'MCPNet: a parallel maximum capacity-based genome-scale gene network construction
      framework'
  source_content_hash: sha256:74d3ecb8bd84e2f7183eb4a6c160adda48b554df1e9d5b67c35fe802bbe5119b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AluruLab/MCPNet
  tools:
  - C++
  - MCPNet
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mcp-score-network-scoring

## Summary

Reconstruct genome-scale gene regulatory networks with MCPNet, which scores direct and indirect gene-gene interactions using the maximum-capacity-path (MCP) score, and evaluate network quality (AUPRC/AUROC) against a ground-truth interaction matrix. Enables reproducible AUPRC and runtime benchmarking against methods like CLR and Arboreto.

## When to use

Use when you need to reverse engineer a gene regulatory network from a gene expression matrix (HDF5/CSV/EXP) and quantify network quality via AUPRC/AUROC against a ground-truth interaction file, especially at genome scale where speed matters.

## When NOT to use

- When no ground-truth interaction matrix is available and you only need an unsupervised network — the AUPR/AUROC reporting requires `-x <groundtruth>.csv`
- When only partial ground truth is available and you need the ensemble-method train/test evaluation — use the `eval_mcpnet` binary instead of `mcpnet`
- When you only need pairwise association scores (Pearson correlation or mutual information) without MCP-score-based indirect-interaction correction — the standalone `pearson` or `mi` binaries suffice

## Inputs

- Gene expression matrix in HDF5 (.h5), EXP, or CSV format (e.g., A. thaliana Development-category microarray data or bundled gnw2000.h5)
- Ground-truth interaction matrix as CSV (e.g., gnw2000_truenet.csv) supplied via -x for AUPR/AUROC evaluation
- Optional transcription factor list (restricts evaluation to TF-to-gene interactions)

## Outputs

- Reconstructed gene regulatory network matrices per method variant, written using the -o output prefix
- AUPR and AUROC metrics for each method variant when ground truth is supplied
- Wall-clock runtime measurements for benchmarking against CLR/Arboreto baselines

## How to apply

Clone the AluruLab/MCPNet repository, initialize submodules, configure with cmake (leave MPI, OpenMP, HDF5, and SIMD options ON by default), and compile with make to produce binaries in {MCPNet_build_dir}/bin. Convert the expression matrix to HDF5 if needed with the `convert` utility (HDF5, EXP, and CSV are supported), then run the `mcpnet` executable with `-i <expression>.h5`, method flags `-m 1 2 3` (Pearson, MI-based variants), `--mi-method 1` for the MI estimator, and `-t 4` threads per compute node; the `-o` parameter takes an output prefix because each method variant produces its own network. Supply the ground-truth interaction matrix with `-x <groundtruth>.csv` so the pipeline computes AUPR and AUROC for each variant — if a transcription factor list is supplied, only TF-to-gene interactions are evaluated. Record wall-clock runtime for comparison against baselines (e.g., CLR 18.5 h on 1 CPU core, Arboreto 44.1 h on eight nodes with 437 GB memory on the A. thaliana dataset, versus MCPNet in <10 min on one node), and write AUPRC values for each variant (e.g., R4 and M4) to a results CSV, checking they lie in [0,1] and reproduce published rankings.

## Related tools

- **MCPNet** (Primary pipeline: computes MI, MCP scores, and AUPR/AUROC metrics from an expression matrix; run via the `mcpnet` executable with -m 1 2 3 and --mi-method 1) — https://github.com/AluruLab/MCPNet
- **C++** (Language in which MCPNet is implemented; build with a modern C++ compiler (GNU g++, clang++, or Intel icpc) with C++11 support via cmake and make)

## Examples

```
{MCPNet_build_dir}/bin/mcpnet -i {MCPNet_source_dir}/data/gnw2000.h5 -o gnw2000 -m 1 2 3 -t 4 --mi-method 1 -x {MCPNet_source_dir}/data/gnw2000_truenet.csv
```

## Evaluation signals

- Each reported AUPRC value lies in [0,1]
- R4 and M4 variant AUPRC values reproduce the published Table 4 results (R4 and M4 are the best variants)
- Recorded MCPNet runtime is far below the 18.5 h CLR baseline (published result: <10 min on one node for the A. thaliana dataset)
- The `mcpnet -h` help output lists the method algorithms supported for each binary, confirming flags are valid
- AUROC/AUPR output is produced only when the ground-truth matrix is supplied via -x, confirming the evaluation path ran

## Limitations

- Software is released as source code only, so it must be compiled on your own system (requires C++11 compiler, MPI such as openmpi or mvapich2, cmake, and HDF5)
- Ensemble-method evaluation with `eval_mcpnet` is only suitable when full groundtruth is available, hence synthetic expression data; with only partial truth the groundtruth is divided into training and testing sets
- AUPR/AUROC metrics are only reported if groundtruth is supplied; without -x, the run produces networks but no quality metrics

## Evidence

- [readme] MCP score uses maximum-capacity-path, a graph theoretical measure, to quantify the relative strengths of direct and indirect gene-gene interactions.: "MCP score uses maximum-capacity-path, a graph theoretical measure, to quantify the relative strengths of direct and indirect gene-gene interactions."
- [readme] mcpnet runs the full pipeline including MI computation, MCP score calculation, and AUPR/AUROC metrics if groundtruth is supplied: "this is the primary executable that runs the MCPNet pipeline, including MI computation, MCP score calculation, and AUPR and AUROC metrics if groundtruth is supplied."
- [results] CLR and Arboreto required 18.5 h and 44.1 h with 437 GB of memory, while MCPNet completed the computation in <10 min on one node: [evidence span withheld — anchor status: not_found]
- [abstract] On the A. thaliana dataset, MCPNet achieves the best AUPRC network quality compared to next-best methods: "we demonstrate that MCPNet produces better quality networks as measured by AUPRC, is significantly faster than all other gene network reconstruction software"
- [readme] If a list of transcription factors are supplied, then only TF to gene interactions are evaluated.: "If a list of transcription factors are supplied, then only TF to gene interactions are evaluated."
