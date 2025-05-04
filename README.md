# DeepIP - Remove internal priming artifacts.
DeepIp is a component of [PolyAseqTrap](https://github.com/APAexplorer/PolyAseqTrap), a deep learning model designed to remove internal priming artifacts in polyA site identification. Inspired by the **DeepPASTA** model ([Arefeen, et al., 2019](https://doi.org/10.1093/bioinformatics/btz283)) that predicts polyA sites from DNA sequences, we designed a deep learning model called DeepIP to predict internal priming artifacts from A-rich polyA sites. DeepIP utilizes both convolutional neural network (CNN) and recurrent neural network (RNN). CNN extracts features from sequences, and RNN is used to combine the extracted feature effects for predicting internal priming artifacts.The corresponding DeepIP scripts are available in the PolyAseqTrap GitHub repository.

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
