# Basic example 

Commands to run:

```bash
# Setup conda environment
conda create -n snakemake-tutorial -y
conda activate snakemake-tutorial
# Install libraries
# - need graphviz to plot the Snakemake DAG's using graphviz's command-line `dot` command
conda install -c conda-forge -c bioconda snakemake graphviz matplotlib  -y

# Create a DAG of the workflow
snakemake --dag | dot -T png > dag.png

# Inspect the steps that would be run in a workflow but DON'T run them
snakemake --dry-run --printshellcmds

# Run workflow
snakemake
```