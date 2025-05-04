### 📁 PolyA Site Training Datasets

The following datasets are provided for polyA site identification training, derived from the 3' reads of *Saccharomyces cerevisiae* and *Schizosaccharomyces pombe*:

| File Name                            | Description                                                                 |
|--------------------------------------|-----------------------------------------------------------------------------|
| `Scerevisiae_SRR5276077_PAC_table.Rdata` | PolyA site training data for *Saccharomyces cerevisiae* (genome: **R64-1-1**) |
| `Spombe_SRR5276080_PAC_table.Rdata`   | PolyA site training data for *Schizosaccharomyces pombe* (genome: **ASM294**) |
| `train_mini.fa`  | A small demo training dataset in FASTA format, containing sequences with polyA sites. This dataset is intended for testing and demonstration purposes. |

These datasets can be used for training and evaluating polyA site prediction models for their respective species.
file here. To train a model using the provided training dataset and generate a trained model, execute the following command:
```bash
python DeepIP_train.py \
  -trainSeq train_mini.fa \
  -trainedModel train_mini.fa.model.hdf5 \
  -epoch 10
```
