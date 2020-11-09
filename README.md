# gui_workflow
The snakemake workflow for the Gulf UnderSea Interconnector feasibility study

## Installation

Install snakemake using conda into a new environment called `snakemake`:

```bash
conda install -c conda-forge mamba
mamba create -c bioconda -c conda-forge -n snakemake snakemake-minimal pandas
```

Then, activate the environment using `conda activate snakemake` on Mac and Linux, or `activate snakemake` on Windows.

Now install the other dependencies using pip:

```python
pip install -r requirements.txt
```

## Configure the workflow

Update the paths in the `config/config.yaml` file for `datapackage:` and `model_file:` keys. These should both hold relative paths to the combined GUI model `datapackage.json` and osemosys model file (e.g. `osemosys_fast.txt`).

Each of these paths can then point to the repositories outside of the gui_workflow. So deployment to an HPC will involve:

1. Copying a release package of OSeMOSYS to the server and unzipping
2. Cloning the gui_osemosys repository to the server
3. Cloning this repository to the server
4. Updating the config so that the paths point to the correct locations

### Main configuration file

A YAML file `config.yaml` must be placed in the config directory.

```yaml
# Populate the scenarios.csv file with a list of scenario names
# An (OSeMOSYS datapackage with scenario name suffix must exist)
# scenarios: config/scenarios.csv

# Tell the workflow which model results to plot
result_params: config/results.csv
agg_results: config/agg_results.csv

# Define the uncertain parameters used to define the Monte Carlo sample
parameters: config/parameters.csv

# Path to datapackage of OSeMOSYS model
datapackage: ../gui_osemosys/combined_model/combined_datapackage/datapackage.json

# Model version
model_file: ../osemosys/OSeMOSYS_GNU_MathProg/src/osemosys_fast.txt

# Sampling - how large should the sample be?
replicates: 1
```

### Parameters file

A CSV file containing the following structure should be placed in the config directory:

```csv
name,group,indexes,min_value,max_value,dist,interpolation_index,action
CapitalCost,capex,"SIMPLICITY,NGCC",500,1100,unif,YEAR,interpolate
DiscountRate,discountrate,"GLOBAL,NGCC",0.05,0.20,unif,None,fixed
```

column_name | description
--- | ---
name | the name of the OSeMOSYS parameter file into which the values should be written
group | the group to which the parameter belonngs (groups of like names are moved together)
indexes | a string of comma-separated entries matching the set elements for the parameter
min_value | the minimum value that the parameter will be sampled
max_value | the maximum value that the parameter will be sampled
dist | the probability distribution - currently, only 'unif' for uniform is supported
interpolation_index | the index name over which the values will be interpolated
action | 'interpolate' will interpolate with a straight-line between the start and end years, where the sample value replaces the end year value; 'fixed' will replace all values in the interpolation index with the sampled value

### Results file

A CSV file containing the following structure should be placed in the config directory:

```csv
name
ProductionByTechnologyAnnual
```

This lists the result files which will be generated by the otoole results processing script.

### Result aggregation file

This CSV file allows users to specify parts of the OSeMOSYS result files to extract and aggregate,
adding the model run as an index.

```csv
resultfile,indices,filename
ProductionByTechnologyAnnual,"SIMPLICITY,NGCC,SEC_EL",electricity_from_gas
ProductionByTechnologyAnnual,"SIMPLICITY,RIVER_2,WATIN",water_from_rivers
```

### Scenarios file

A CSV file containing the following structure should be placed in the config directory:

```csv
name,description,path
0,"Interconnector Optimised",../gui_osemosys/combined_model/combined_datapackage/datapackage.json
```

This file is used to point to master models. Importantly, each of the master models is used as
a base for the N replicates defined in `config.yaml. If you define 3 master models in this file,
and N=100, then 300 model runs will be scheduled, but with the same 100 parameter values.

Use these master models to define macro scenarios - e.g. forcing in and out a key technology.

## Running the workflow

To run the workflow, using the command `snakemake --use-conda --cores 4--resources mem_mb=16000 disk_mb=30000`

## Plotting the workflow

To visualise the workflow, run the following rule: `snakemake plot_dag --use-conda  --cores 2`

## Folder structure

This repository follows the snakemake guidelines for reproducibility:

    ├── .gitignore
    ├── README.md
    ├── LICENSE.md
    ├── modelrun
    ├── workflow
    │   ├── rules
    |   │   ├── module1.smk
    |   │   └── module2.smk
    │   ├── envs
    |   │   ├── tool1.yaml
    |   │   └── tool2.yaml
    │   ├── scripts
    |   │   ├── script1.py
    |   │   └── script2.R
    │   ├── notebooks
    |   │   ├── notebook1.py.ipynb
    |   │   └── notebook2.r.ipynb
    │   ├── report
    |   │   ├── plot1.rst
    |   │   └── plot2.rst
    |   └── Snakefile
    ├── config
    │   ├── config.yaml
    │   └── some-sheet.csv
    ├── results
    └── resources