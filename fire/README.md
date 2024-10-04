# Fire example

Suppose we want to look at a histogram of fires in different countries using the `Argrofood_co2_emission.csv` dataset. 
First download the dataset via either `curl` or `wget`. 
Via `curl`:

```bash
# Download fire data via curl or wget. Via curl:
curl -L https://docs.google.com/uc\?export\=download\&id\=1Wytf3ryf9EtOwaloms8HEzLG0yjtRqxr > fire_data.csv
```

Setup the `conda` environment:

```bash
conda create -n snakemake -y
conda activate snakemake-tutorial
# Install libraries
# - need graphviz to plot the Snakemake DAG's using graphviz's command-line `dot` command
conda install -c conda-forge -c bioconda snakemake graphviz matplotlib  -y
```

Next, we need to extract the fire data for a given country and store it in a file. E.g., for Afghanistan:

```bash
python get_data.py fire_data.csv Afghanistan Afghanistan.txt
```

Last we want to make a histogram of those counts. `hist.py` takes a file with one number per line and creates a histogram of that data.

```bash
python hist.py Afghanistan.txt Afghanistan.png Afghanistan Fires Freq
```

A simple Snakemake workflow/`Snakefile` to do this would be:

```
rule country_fire_counts:
  input:
    'fire_data.csv'
  output:
    'Afghanistan.txt'
  shell:
    'python get_data.py {input} Afghanistan {output}'

rule country_plot_hist:
  input:
    'Afghanistan.txt'
  output:
    'Afghanistan.png'
  shell:
    'python hist.py {input} {output} Afghanistan Fires Freq'
```

Run `snakemake` and tell it which output file you want:

```bash
snakemake Afghanistan.png
# or, if you want to just see what would happen but not actually perform the steps, run `snakemake -np Afghanistan.png
```

In the `Snakefile` above, we take advantage of variables `{input}` and `{output}` that resolve to the values of the input and output directives. 
NOTE: Snakemake abuses the strings wrapped in curly brackets syntax. 
In some instances they are variables and in others they are wildcards which we will cover next.

The issue with these rules can only create the fire counts files for Afghanistan. 
With rules like these, we would need to create new rules for every country. 
To help make our rules more general, Snakemake uses wildcards. 
A [wildcard](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#wildcards) is a placeholder used within rule definitions to generalize filenames and allow for pattern matching. 
Wildcards make rules more flexible and adaptable to different inputs and outputs without having to specify each filename explicitly. 
Wildcards are denoted by curly braces {} in rule definitions. 
For example, if we replace the output with a wildcard, then country_file_counts will work for any country.
Replace your previous `Snakefile` with this one and run `snakemake` again:

```
rule country_fire_counts:
  input:
   'fire_data.csv'
  output:
    '{country}.txt'
  shell:
    'python get_data.py {input} {wildcards.country} {output}'

rule country_plot_hist:
  input:
    '{country}.txt'
  output:
    '{country}.png'
  shell:
    'python hist.py {input} {output} {wildcards.country} Fires Freq'
```

When Snakemake determines that this rule can be applied to generate a target file by replacing the wildcard `{country}` in the output file with an appropriate value, it will propagate that value to all occurrences of `{country}` in the input files and thereby determine the necessary input for the resulting job.

You can now use Snakemake to plot the data from Afghanistan and multiple countries via, e.g.,

```
snakemake -np Afghanistan.png
snakemake -np Albania.png Algeria.png
snakemake -np {"American Samoa",Andorra}.png

# See the DAG
snakemake --dag {"American Samoa",Andorra}.png | dot -T png > dag.png
```


