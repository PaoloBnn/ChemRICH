# ChemRICH : Chemical Similarity Enrichment Analysis for metabolomics data

***

## Citation

[Barupal, D.K. and Fiehn, O., 2017. Chemical Similarity Enrichment Analysis (ChemRICH) as alternative to biochemical pathway mapping for metabolomic datasets Scientific Report 2017. (https://www.nature.com/articles/s41598-017-15231-w)

***

***

# ChemRICH WorkFlow Script for statistical results
Note : If you want to compute the statistical results from raw data, use the next script (scroll down). 

## Computer requirement :

R version - R 3.5.1 or latest

Set up Java - follow [these instructions](https://stackoverflow.com/questions/6492361/problem-loading-rjava)

## Step 0 # Prepare the input data.

In RStudio, create a new project environment by clicking on File --> New Project

  Use [chemrich_input_stats.xlsx](https://github.com/barupal/chemrich/blob/master/chemrich_input_stats.xlsx?raw=true) file as a template. Download it and replace it's content with your study's data.
  It has one sheet -

  1) input - details about compounds. First column of this sheet must be "CompoundID", which has to be a unique arbiterary compound identifier such as "CPD0001"
The input file must contain KEGG IDs, SMILES codes, PubChem IDs, p-value and fold-changes. If you have received data from Metabolon, the file should already have these annotations.

Put "chemrich_input_stats.xlsx" file inside the R-studio project directory. 

## Step 1. Install ChemRICH workflow package
```
install.packages("https://github.com/barupal/chemrich/raw/master/ChemRICHWorkFlow_0.1.0.tar.gz", repos = NULL, type = "source")
library(ChemRICHWorkFlow)
```
## Step 2 . Provide a project name
```
project_name <- "chemrich_1" # Provide this analysis a name. This will be prefixed to all the exported files.
```
## Step 3 . Load required R packages.
```
ChemRICHWorkFlow::load.ChemRICH.Packages()
```
## Step 4. Load ChemRICH databases
```
ChemRICHWorkFlow::load.ChemRICH.databases()
```
## Step 5. Data import into R. 
Select below three lines and click on Run or press crtl+enter.
```
 data_dict <- readxl::read_xlsx("chemrich_input_stats.xlsx", sheet="input") # Data Dictionary
```

## Step 6. Prepare Input for ChemRICH analysis
Note : compounds having the SMILES codes will be used for the next steps.
```
chemrich.input.file <- ChemRICHWorkFlow::prepare.chemrich.input()
```
## Step 7. Get Chemical modules
Note : If you already have chemical classes. Add a column "ChemicalClass" into the data_dict sheet in the chemrich_input.xlsx file.
```
chemrich.input.file <- ChemRICHWorkFlow::chemrich.getChemicalClass()
write.table(chemrich.input.file,paste0(project_name,"chemrich_input_file_with_clases.txt"), col.names = T, row.names = F, quote = F, sep="\t")
```
## Step 8. Get significant chemical modules
```
signif.chemrich.cluster <- ChemRICHWorkFlow::chemrich.GetSignificantClasses()
```
## Step 9 . Visualize enriched modules
```
ChemRICHWorkFlow::export.chemrich.impactPlot(signif.chemrich.cluster)
```
## Step 10 . Export Interactive ChemRICH plots.
```
ChemRICHWorkFlow::export.chemrich.interactivePlot(signif.chemrich.cluster)
```
## step 11. Export chemical similarity tree
```
ChemRICHWorkFlow::export.chemrich.similarityTree()
```
It should look like this - 
<img src="https://raw.githubusercontent.com/barupal/chemrich/master/chemrich_tree.png" width="50%">

## Step 12. Export Results Tables.
```
ChemRICHWorkFlow::export.chemrich.tables(signif.chemrich.cluster)
```
See [ChemRICH_results.xlsx](https://github.com/barupal/chemrich/raw/master/ChemRICH_results.xlsx) file for expected results.

***

# ChemRICH Workflow Script to start with raw data. 

## Computer requirement :

R version - R 3.5.1 or latest

Set up Java - follow [these instructions](https://stackoverflow.com/questions/6492361/problem-loading-rjava)


## Step 0 # Prepare the input data.

In RStudio, create a new project environment by clicking on File --> New Project

  Use [chemrich_input.xlsx](https://github.com/barupal/chemrich/blob/master/chemrich_input.xlsx?raw=true) file as a template. Download it and replace it's content with your study's data.
  It has three sheets -

  1) data_dict - details about compounds. First column of this sheet must be "CompoundID"
The data dictionary file must contain KEGG IDs, SMILES codes and chemical class or pathway annotation. If you have received data from Metabolon, the file   should already have these annotations.

 2) sample_metadata - details about samples. First column of this must be "Sample_ID"

 3) data_matrix - metabolite numerical data. First column must be "CompoundID" and rest should be eash sample denoted by it's "Sample_ID"
 
 Put "chemrich_input.xlsx" file inside the R-studio project directory. 

## Step 1. Install ChemRICH workflow package
```
install.packages("https://github.com/barupal/chemrich/raw/master/ChemRICHWorkFlow_0.1.0.tar.gz", repos = NULL, type = "source")
library(ChemRICHWorkFlow)
```

## Step 2 . Provide a project name
```
project_name <- "chemrich_1" # Provide this analysis a name. This will be prefixed to all the exported files.
```
## Step 3 . Load required R packages.
```
ChemRICHWorkFlow::load.ChemRICH.Packages()
```
## Step 4. Load ChemRICH databases
```
ChemRICHWorkFlow::load.ChemRICH.databases()
```
## Step 5. Data import into R. 
Select below three lines and click on Run or press crtl+enter.
```
  data_dict <- readxl::read_xlsx("chemrich_input.xlsx", sheet="data_dict") # Data Dictionary
  sample_metadata <- readxl::read_xlsx("chemrich_input.xlsx", sheet="sample_metadata") # Sample metadata
  data_matrix <- readxl::read_xlsx("chemrich_input.xlsx", sheet="data_matrix") # Data matrix
```
## Step 6. Inspect sample_metadata columns.
```
colnames(sample_metadata)

grouping_variable <- "TISSUE TYPE"

table(sample_metadata[grouping_variable])

```

### Get compound count by a metabolite category variable.

```
colnames(data_dict) # First check what variable names you have in the data dictionary
table(data_dict["SUPER PATHWAY"]) # metabolite count by super pathway
table(data_dict["SUB PATHWAY"]) # metabolite count by sub pathway
table(data_dict["PLATFORM"]) # metabolite count by analytical plateform 
```
## Step 7. Find significant metabolites
If you already have p-value and effect size (fold change or beta coefficient) for your analysis, skip this.
Provide your results in the data_dict sheet in the chemrich_input.xlsx file. Add two columns - pvalue and fc. FC means fold-change.
Negative beta values need to be converted to below 1 eg  -2 will become 0.50.
```
metsig.df <- ChemRICHWorkFlow::getSignifMetabolites(grouping_variable)
write.table(metsig.df,paste0(project_name,"_significant_metabolites.txt"), col.names = T, row.names = F, quote = F, sep="\t")
```
## Step 8. Prepare Input for ChemRICH analysis
Note : compounds having the SMILES codes will be used for the next steps.
```
chemrich.input.file <- ChemRICHWorkFlow::prepare.chemrich.input()
```
## Step 9. Get Chemical modules
Note : If you already have chemical classes. Add a column "ChemicalClass" into the data_dict sheet in the chemrich_input.xlsx file.
```
chemrich.input.file <- ChemRICHWorkFlow::chemrich.getChemicalClass()
write.table(chemrich.input.file,paste0(project_name,"chemrich_input_file_with_clases.txt"), col.names = T, row.names = F, quote = F, sep="\t")
```
## Step 10. Get significant chemical modules
```
signif.chemrich.cluster <- ChemRICHWorkFlow::chemrich.GetSignificantClasses()
```
## Step 11 . Visualize enriched modules
```
ChemRICHWorkFlow::export.chemrich.impactPlot(signif.chemrich.cluster)
```
It should look like this -
<img src="https://github.com/barupal/chemrich/raw/master/chemrich_1_chemrich_impact_plot.png" width="50%">
## Step 12 . Export Interactive ChemRICH plots.
```
ChemRICHWorkFlow::export.chemrich.interactivePlot(signif.chemrich.cluster)
```
## step 13 . Export chemical similarity tree
```
ChemRICHWorkFlow::export.chemrich.similarityTree()
```
It should look like this - 
<img src="https://raw.githubusercontent.com/barupal/chemrich/master/chemrich_tree.png" width="50%">

## Step 14 . Export Results Tables.
```
ChemRICHWorkFlow::export.chemrich.tables(signif.chemrich.cluster)
```
See [ChemRICH_results.xlsx](https://github.com/barupal/chemrich/raw/master/ChemRICH_results.xlsx) file for expected results. 

## Step 15. Visualize correlation, KEGG and Chemical Similarity Links within each module
```
ChemRICHWorkFlow::chemrich.getIntegratedNetwork(moduleName,compoundLabel)
```
for example -
```
ChemRICHWorkFlow::chemrich.getIntegratedNetwork("Adenine Nucleotides", "BIOCHEMICAL NAME")
```
## Step 16. Generate Box and whisker plots
```
ChemRICHWorkFlow::chemrich.generateBWplots(moduleName,compoundLabel)
```
for example 
```
ChemRICHWorkFlow::chemrich.generateBWplots("Adenine Nucleotides", "BIOCHEMICAL NAME")
```
## THE END

***

# ChemRICH Class Enrichment Analysis for user-provided classes
If you do not have SMILES code or InChiKeys for some compounds in your dataset but you do have the class information, you can use this small workflow to perform the ChemRich analysis for your study. 
## Computer requirement :
R version - R 3.5.1 or latest

Set up Java - follow [these instructions](https://stackoverflow.com/questions/6492361/problem-loading-rjava)

## Step 0 Prepare the input data.

In RStudio, create a new project environment by clicking on File --> New Project

  Use [chemrich_class_template.xlsx](https://github.com/barupal/chemrich/blob/master/chemrich_class_template.xlsx?raw=true) file as a template. Download it and replace it's content with your study's data.
  
 Put "chemrich_class_template.xlsx" file inside the R-studio project directory. 

## Step 1. Install ChemRICH workflow package
```
install.packages("https://github.com/barupal/chemrich/raw/master/ChemRICHWorkFlow_0.1.0.tar.gz", repos = NULL, type = "source")
library(ChemRICHWorkFlow)
```

## Step 2 . Provide a project name
```
project_name <- "demo_" # Provide this analysis a name. This will be prefixed to all the exported files.
```
## Step 3 . Load required R packages.
```
ChemRICHWorkFlow::load.ChemRICH.Packages()
```

## Step 4. Compute the ChemRICH Enrichment Analysis
```
ChemRICHWorkFlow::runChemRICHClass("chemrich_class_template.xlsx",project_name)
```

Results should have been exported. 

## THE END


***

# ChemRICH Workflow Script for the analysis of statistical results from multiple conditions

## Computer requirement :
R version - R 3.5.1 or latest

Set up Java - follow [these instructions](https://stackoverflow.com/questions/6492361/problem-loading-rjava)

## Step 0 Prepare the data

In RStudio, create a new project environment by clicking on File --> New Project

  Use [chemrich_multi_input.xlsx](https://github.com/barupal/chemrich/blob/master/chemrich_multi_input.xlsx?raw=true) file as a template. Download it and replace it's content with your study's data.
  It has one sheet-

  1) data_dict - details about compounds. Provide pvalue and foldchange columns for each statistical comparison. 

 Put "chemrich_multi_input.xlsx" file inside the R-studio project directory.

## Step 1. Install ChemRICH workflow package
```
install.packages("https://github.com/barupal/chemrich/raw/master/ChemRICHWorkFlow_0.1.0.tar.gz", repos = NULL, type = "source")
library(ChemRICHWorkFlow)
```

## Step 2 . Load required R packages.
```
ChemRICHWorkFlow::load.ChemRICH.Packages()
```
## Step 3. Load ChemRICH databases
```
ChemRICHWorkFlow::load.ChemRICH.databases()
```

## Step 4. Set global variables 
```
project_name <- "chemrich_multi_1" # Provide this analysis a name. This will be prefixed to all the exported files.
classVariable <- "Class"
inputfile <- "chemrich_multi_input.xlsx"

```
## Step 5. Compute the ChemRICH Multiple Analysis
```
ChemRICHWorkFlow::chemrih_multi_group(inputfile)
```

## ChemRICH plots and excel sheets have been exported, check the project directory. 

***
# Use OpenCPU version of ChemRICH, if you want to run the ChemRICH web-gui locally. 
Make sure you have latest JAVA (JDK and JRE both) installed on your computer. [Latest Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

In R, run the following code.
```
if (!require("devtools"))
install.packages('devtools', repos="http://cran.rstudio.com/")
if (!require("opencpu"))
install.packages('opencpu', repos="http://cran.rstudio.com/")
if (!require("RCurl"))
install.packages('RCurl', repos="http://cran.rstudio.com/")
if (!require("pacman"))
install.packages('pacman', repos="http://cran.rstudio.com/")
library(devtools)
library(RCurl)
library(pacman)
source('https://bioconductor.org/biocLite.R')
pacman::p_load(grid,rcdk, RJSONIO,openxlsx, RCurl, rvg, magrittr, dynamicTreeCut,ape,ggplot2, ggrepel,ReporteRs, officer,phytools, plotrix, plotly, htmlwidgets,DT,extrafont,XLConnect)
install_github('barupal/chemrich')
library(ChemRICH)
library(opencpu)
opencpu::ocpu_start_server()
```
Then go to :
[ChemRICH Local Version](http://localhost:5656/ocpu/library/ChemRICH/www/)

