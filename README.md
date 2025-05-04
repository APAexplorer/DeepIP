# DeepIP - Remove internal priming artifacts.
（**Note**）: The [DeepIP_R](https://github.com/APAexplorer/DeepIP_R) repository offers a complete R-only version of DeepIP for users who prefer working in a native R environment.
DeepIp is a component of [PolyAseqTrap](https://github.com/APAexplorer/PolyAseqTrap), a deep learning model designed to remove internal priming artifacts in polyA site identification. Inspired by the **DeepPASTA** model [Arefeen et al., 2019](https://doi.org/10.1093/bioinformatics/btz283)
that predicts polyA sites from DNA sequences, we designed a deep learning model called DeepIP to predict internal priming artifacts from A-rich polyA sites. DeepIP utilizes both convolutional neural network (CNN) and recurrent neural network (RNN). CNN extracts features from sequences, and RNN is used to combine the extracted feature effects for predicting internal priming artifacts. The corresponding DeepIP scripts can also be found in the [PolyAseqTrap](https://github.com/APAexplorer/PolyAseqTrap) GitHub repository. The schematic diagram of DeepIP is shown below.

<img src="https://github.com/APAexplorer/PolyAseqTrap/blob/main/img/DeepIP_schema.png" alt="schema" style="width:90%;"/>

## Install DeepIP

DeepIP can run on both Linux, and Windows, systems. To install and use DeepIP, you need to have [Conda](https://anaconda.org/anaconda/conda) installed on your machine. Please follow the steps below to set up DeepIP:


* <span style="color: blue;"> **Prerequisites** </span>

Ensure that **Conda** is installed on your system. If not, you can download and install Miniconda or Anaconda from the following links:

   * [Miniconda](https://docs.conda.io/en/latest/miniconda.html)
  
   * [Anaconda](https://docs.anaconda.com/anaconda/install/)


* <span style="color: blue;"> **Install DeepIP** </span>

Once Conda is installed, you can create a new Conda environment and install DeepIP by running the following commands in your terminal (Linux) or command prompt (Windows):

```bash
# Create a new conda environment
conda create -n DeepIP python=3.7
conda env list 
# Activate the environment
conda activate DeepIP 
# Install additional dependencies
pip install keras
pip install tensorflow
pip install pandas
pip install sklearn 
pip install scikit-klearn 

```

## Build the training model (optional)
Currently, we provide pre-trained models for human, mouse, and Arabidopsis species. If you would like to build a model for your own species, you can follow the steps below.

* <span style="color: blue;"> **Prepare training data** </span>

For model training, you need to prepare your training data. The training sequences should consist of the 100bp sequences upstream and downstream of the polyA site. 

* <span style="color: blue;"> **Build a training model** </span>

For our pre-trained models for human, mouse, and Arabidopsis, the positive data of the model are A-rich sequences with polyA sites and the negative data are A-rich sequences without polyA sites. 

```bash
python DeepIP_train.py \
  -trainSeq train_mini.fa \
  -trainedModel train_mini.fa.model.hdf5 \
  -epoch 10
```
Where:

 *  `trainSeq`: is the input training data (a FASTA file containing 200 bp geneome sequences).
 
 *  `trainedModel`: is the name of the output model (this will be saved as an HDF5 file).
 
 * `epoch`: the number of training iterations (default is 100). You can set a different number based on your training needs.
 
Alternatively, you can run it in R.

```{r message=FALSE, eval=FALSE}
library(reticulate)
use_condaenv("/path/miniconda3/envs/DeepIP/")
# Build your own model
py_run_string("trainSeq='train_mini.fa';
              trainedModel='train_mini.model.hdf5'; 
              epoch=10; seqLabel='01'")
py_run_file('DeepIP_train.py')

```
 
## Test the model
Once your model is trained or if you are using a pre-trained model, you can proceed to test it on new sequences. The model accepts a 200 nt genomic sequence as input and predicts whether the middle position of the input sequence corresponds to an internal priming site. Here's how to perform the testing:

```bash
python DeepIP_test.py \
 -testSeq test_mini.fa \
 -trainedModel train_mini.fa.model.hdf5 \
 -outputFile test_result.csv
```

Alternatively, you can run it in R.
```{r message=FALSE, eval=FALSE}
library(reticulate) 
use_condaenv("/path/miniconda3/envs/DeepIP/")
# to identify whether PACs are internal priming artifacts
py_run_string("testSeq='test_mini.fa'; 
              trainedModel='train.mini.model.hdf5';
              outputFile='train.mini.predicted.csv'; 
              seqLabel='0/1'")
py_run_file('DeepIP_test.py')
```

## Remove internal priming and regroup nearby cleavage sites
3’seq techniques based on oligo(dT), such as PAC-seq, PAS-seq, polyA-seq, and WTTS-seq, are prone to internal priming artifacts, which can lead to inaccurate identification of PACs. To mitigate this issue, `PolyAseqTrap` integrates a deep learning-based model, `DeepIP`, to accurately identify whether A-rich PACs are internal priming artifacts. To utilize `DeepIP`, first extract the 100 bp upstream and downstream sequences of A-rich polyA sites from the PACs results generated by the `FindPTA` function. These sequences are then classified by `DeepIP` to identify potential internal priming artifacts.

* <span style="color: blue;">**Extract a sequence of 200 nt surrounding A-rich polyA sites**</span> 
```{r message=FALSE, eval=FALSE}
library(PolyAseqTrap, warn.conflicts = FALSE, quietly=TRUE)
library("BSgenome.Hsapiens.UCSC.hg38", quietly = TRUE)
bsgenome <-BSgenome.Hsapiens.UCSC.hg38
## load identified PACs in human genome generated by FindPTA function
data(PACs_human)

## extract 200 bp genome sequences surrounding A-rich polyA sites
generateFASTA(reads=PACs_human$pa.table,
              bsgenome=bsgenome,
              output.name="human_Arich_PACs.fasta")
```

* <span style="color: blue;">**Use DeepIP to classify A-rich polyA sites**</span> 
```{r message=FALSE, eval=FALSE}
## use DeepIP to classify A-rich polyA sites
library(reticulate) 
use_condaenv("/path/miniconda3/envs/DeepIP/")
py_run_string("testSeq='human_Arich_PACs.fasta';
              trainedModel='human.train.model.hdf5'; 
              outputFile='DeepIP_result_hg.csv'")
py_run_file("DeepIP_test.py")

# Alternatively, run the following Python command outside R:
# python DeepIP_test.py -testSeq human_Arich_PACs.fasta  -trainedModel human.train.model.hdf5 -outputFile DeepIP_result_hg.csv
```

* <span style="color: blue;">**Remove internal priming artifacts and regroup nearby cleavage sites**</span> 
```{r message=FALSE, eval=FALSE}
### remove internal priming artifacts
ip.table <- read.csv("DeepIP_result_hg.csv")
head(ip.table)
#title                score     true_label predict_label res
#>chr22_19354941_-_0 0.4143335          0             0  TN
#>chr22_19382772_-_0 0.4143623          0             0  TN
#>chr22_19849373_-_0 0.4143335          0             0  TN

## extract and remove internal priming artifacts
ip.table <- subset(  ip.table,predict_label==0)
ip.table$title <- gsub("^>","",  ip.table$title)
PACs_human$pa.table$label <- paste0(
  PACs_human$pa.table$chr,"_",
  PACs_human$pa.table$coord,"_",
  PACs_human$pa.table$strand,"_0")
PACs_human$pa.table$level <- as.character(PACs_human$pa.table$level)
index <- which(PACs_human$pa.table$label %in% ip.table$title)

# remove internal priming artifacts
PACs_human$pa.table$level[index] <- "Count"
PACs_human$pa.table$level <- factor( PACs_human$pa.table$level,
                                     levels=c("V1","V2","V3","V4","V5","V6","V7","V8","Count"))
                              
                                
## regroup nearby cleavage sites
PACs_human <- resut.PA(aln.result=PACs_human$pa.table,d=24)
``` 

**Note:** When running the analysis for other species, ensure that both the `bsgenome` object and the corresponding `training model` are updated to reflect the appropriate species. For example, for mouse, use `BSgenome.Mmusculus.UCSC.mm10` as the `bsgenome` object and `mouse.train.model.hdf5` as the training model. However, if a species-specific model is not available, the human model (`human.train.model.hdf5`) can still be used and should provide reliable results.
