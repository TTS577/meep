# meep

A Snakemake pipeline for human data depletion and bacterial whole-genome
sequencing (WGS) based on Oxford Nanopore Technology (ONT) long reads.

## Pipeline overview

| Step | Rule | Tool | Description |
|------|------|------|-------------|
| 01 | `nanostat_raw` | NanoStat | QC of raw reads |
| 02 | `human_depletion_minimap2` | minimap2 + samtools | Deplete human reads |
| 03 | `porechop` | Porechop | Adapter trimming |
| 04 | `nanostat_clean` | NanoStat | QC of cleaned reads |
| 05 | `filtlong` | Filtlong | Quality filtering |
| 06 | `flye` | Flye | De novo assembly |
| 07 | `quast` | QUAST | QC of unpolished assembly |
| 08 | `medaka` | Medaka | Long-read polishing |
| 09 | `checkm2` | CheckM2 | Genome completeness & contamination |
| 10 | `quast_polished` | QUAST | QC of polished assembly |
| 11 | `multiqc` | MultiQC | Aggregate all QC reports |

## Requirements

- [Snakemake](https://snakemake.readthedocs.io/) ≥ 7.0
- For container execution: [Singularity/Apptainer](https://apptainer.org/) ≥ 3.0
- For conda execution: [Conda](https://docs.conda.io/) or [Mamba](https://mamba.readthedocs.io/)

## Setup

### 1. Sample sheet

Edit `config/samples.tsv` to list your samples and their FASTQ paths:

```
sample	long_reads
sample1	/path/to/sample1_reads.fastq.gz
sample2	/path/to/sample2_reads.fastq.gz
```

### 2. Config file

Edit `config/config.yaml` to set:

| Parameter | Description |
|-----------|-------------|
| `outdir` | Output directory (default: `meep_pipeline/results`) |
| `human_ref` | Path to pre-built human reference minimap2 index (`.mmi`) |
| `samples` | Path to sample sheet TSV |
| `filtlong.min_length` | Minimum read length in bp |
| `filtlong.min_mean_q` | Minimum mean Phred quality score |
| `filtlong.keep_percent` | Percentage of best bases to keep |
| `flye.read_type` | Flye read type flag (e.g. `--nano-hq`) |
| `flye.genome_size` | Expected genome size (e.g. `5m`) |
| `flye.min_overlap` | Minimum overlap for Flye |
| `flye.extra_args` | Additional Flye flags (e.g. `--meta`) |
| `medaka.model` | Medaka model (e.g. `r1041_e82_400bps_hac_g632`) |
| `medaka.chunk_len` | Consensus chunk length (default: `800`) |
| `medaka.chunk_ovlp` | Overlap between chunks (default: `400`) |
| `checkm2.db` | Path to CheckM2 diamond database (`.dmnd`) |

### 3. Resources

- **Human reference index**: place or symlink your pre-built GRCh38 minimap2
  index at `meep_pipeline/resources/GRCh38.mmi` (or update `human_ref` in the
  config).
- **CheckM2 database**: place the database at
  `meep_pipeline/resources/checkm2_db/uniref100.KO.1.dmnd` (or update
  `checkm2.db` in the config).

> **Note**: The `meep_pipeline/resources/` directory is listed in `.gitignore`
> to prevent large database and reference files from being committed. You must
> create this directory locally and populate it with the required files before
> running the pipeline.

## Running the pipeline

### With containers (recommended)

Uses per-rule Singularity/Apptainer images pulled automatically from
BioContainers:

```bash
snakemake --use-singularity --cores <N>
```

### With conda environments

Uses per-rule conda environments defined in `envs/`:

```bash
snakemake --use-conda --cores <N>
```

### Dry run

Preview jobs without executing:

```bash
snakemake --use-singularity --cores <N> -n
```

## Output structure

```
meep_pipeline/results/
├── <sample>/
│   ├── 01_nanostat_raw/
│   ├── 02_human_depletion/
│   ├── 03_porechop/
│   ├── 04_nanostat_clean/
│   ├── 05_filtlong/
│   ├── 06_flye/
│   ├── 07_quast/            ← QUAST report for unpolished assembly
│   ├── 08_medaka/
│   ├── 09_checkm2/
│   ├── 10_quast_polished/   ← QUAST report for polished assembly
│   └── logs/
└── multiqc/
    └── multiqc_report.html  ← Aggregated QC report
```
