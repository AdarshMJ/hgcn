Hyperbolic Graph Convolutional Networks in PyTorch
==================================================

## 1. Overview

This repository is a graph representation learning library, containing an implementation of Hyperbolic Graph Convolutions [1] in PyTorch, as well as multiple embedding approaches including:

#### Shallow methods (```Shallow```)

  * Shallow Euclidean
  * Shallow Hyperbolic [2]
  * Shallow Euclidean + Features (see [1])
  * Shallow Hyperbolic + Features (see [1])
  
#### Neural Network (NN) methods 

  * Multi-Layer Perceptron (```MLP```)
  * Hyperbolic Neural Networks (```HNN```) [3]
  
#### Graph Neural Network (GNN)  methods 

  * Graph Convolutional Neural Networks (```GCN```) [4]
  * Graph Attention Networks (```GAT```) [5]
  * Hyperbolic Graph Convolutions (```HyperGCN```) [1]

All models can be trained for 

  * Link prediction (```lp```)
  * Node classification (```nc```)
  * Network reconstruction (```rec```)

## 2. Setup

### 2.1 Installation

If you don't have conda installed, please install it following the instructions [here](https://conda.io/projects/conda/en/latest/user-guide/install/index.html).

```git clone https://github.com/ines-chami/hgcn.git```

```cd hgcn```

```conda env create -f environment.yml```

### 2.2 Datasets

The ```data/``` folder contains source files for cora and pubmed benchmarks. 
To run this code on new datasets, please add corresponding data processing and loading in ```load_data_nc``` and ```load_data_rec``` functions in ```utils/data_utils.py```.

## 3. Usage

### 3.1 ```set_env.sh```

Before training, run 

```source set_env.sh```

This will create environment variables that are used in the code. 

### 3.2  ```train.py```

This script trains models for link prediction, node classification or network reconstruction tasks. 
Metrics are printed at the end of training or can be saved in a directory by adding the command line argument ```--save=1```.

```
python train.py 

    optional arguments:
      -h, --help            show this help message and exit
      --lr LR               learning rate
      --dropout DROPOUT     dropout probability
      --cuda CUDA           which cuda device to use (-1 for cpu training)
      --epochs EPOCHS       maximum number of epochs to train for
      --weight-decay WEIGHT_DECAY
                            l2 regularization strength
      --optimizer OPTIMIZER
                            which optimizer to use, can be any of [Adam,
                            RiemannianAdam]
      --momentum MOMENTUM   momentum in optimizer
      --patience PATIENCE   patience for early stopping
      --seed SEED           seed for training
      --log-freq LOG_FREQ   how often to compute print train/val metrics (in
                            epochs)
      --eval-freq EVAL_FREQ
                            how often to compute val metrics (in epochs)
      --save SAVE           1 to save model and logs and 0 otherwise
      --save-dir SAVE_DIR   path to save training logs (defaults to
                            logs/task/date/run/)
      --sweep-c SWEEP_C
      --lr-reduce-freq LR_REDUCE_FREQ
                            reduce lr every lr-reduce-freq or None to keep lr
                            constant
      --gamma GAMMA         gamma for lr scheduler
      --split-seed SPLIT_SEED
                            seed for data splits (train/test/val)
      --print-epoch PRINT_EPOCH
      --grad-clip GRAD_CLIP
                            max norm for gradient clipping, or None for no
                            gradient clipping
      --min-epochs MIN_EPOCHS
                            do not early stop before min-epochs
      --task TASK           which tasks to train on, can be any of [lp, rec, nc]
      --model MODEL         which encoder to use, can be any of [Shallow, MLP,
                            HNN, GCN, GAT, HyperGCN]
      --dim DIM             embedding dimension
      --manifold MANIFOLD   which manifold to use, can be any of [Euclidean, PoincareBall]
      --c C                 hyperbolic radius, set to None for trainable curvature
      --r R                 fermi-dirac decoder parameter for lp
      --t T                 fermi-dirac decoder parameter for lp
      --pretrained-embeddings PRETRAINED_EMBEDDINGS
                            path to pretrained embeddings (.npy file) for Shallow node
                            classification
      --pos-weight POS_WEIGHT
                            whether to upweight positive class in node
                            classification tasks
      --num-layers NUM_LAYERS
                            number of hidden layers in encoder
      --bias BIAS           whether to use bias (1) or not (0)
      --act ACT             which activation function to use (or None for no
                            activation)
      --n-heads N_HEADS     number of attention heads for graph attention
                            networks, must be a divisor dim 
      --alpha ALPHA         alpha parameter for Graph Attention Networks
      --use-att USE_ATT     whether to use hyperbolic attention in HyperGCN model
      --dataset DATASET     which dataset to use
      --val-prop VAL_PROP   proportion of validation edges for link prediction
      --test-prop TEST_PROP
                            proportion of test edges for link prediction
      --use-feats USE_FEATS
                            whether to use node features or not
      --normalize-feats NORMALIZE_FEATS
                            whether to normalize input node features
      --normalize-adj NORMALIZE_ADJ
                            whether to row-normalize the adjacency matrix
```

## 4. Examples

### 4.1 Link prediction on cora dataset

 * Shallow Euclidean (Test ROC-AUC=86.40): 

```python train.py --task lp --dataset cora --model Shallow --manifold Euclidean --lr 0.01 --weight-decay 0.0005 --dim 16 --num-layers 0 --use-feats 0 --dropout 0.2 --act None --bias 0 --optimizer Adam --cuda 0```

 * Shallow Hyperbolic (Test ROC-AUC=85.97): 

```python train.py --task lp --dataset cora --model Shallow --manifold PoincareBall --lr 0.01 --weight-decay 0.0005 --dim 16 --num-layers 0 --use-feats 0 --dropout 0.2 --act None --bias 0 --optimizer RiemannianAdam --cuda 0 --epochs 5000```

 * GCN (Test ROC-AUC=89.2): 

```python train.py --task lp --dataset cora --model GCN --lr 0.01 --dim 16 --num-layers 2 --act relu --bias 1 --dropout 0.2 --weight-decay 0 --manifold Euclidean --log-freq 5 --cuda 0```

 * HNN (Test ROC-AUC=90.43): 

```python train.py --task lp --dataset cora --model HNN --lr 0.01 --dim 16 --num-layers 2 --act None --bias 1 --dropout 0.2 --weight-decay 0.001 --manifold PoincareBall --log-freq 5 --cuda 0 --c 1```

 * HyperGCN (Test ROC-AUC=93.81): 

```python train.py --task lp --dataset cora --model HyperGCN --lr 0.01 --dim 16 --num-layers 2 --act relu --bias 1 --dropout 0.5 --weight-decay 0.001 --manifold PoincareBall --log-freq 5 --cuda 0 --c None```
 
### 4.2 Node classification on pubmed dataset

 * HNN (Test accuracy=70.3): 
 
``` python train.py --task nc --dataset pubmed --model HNN --lr 0.01 --dim 16 --num-layers 2 --act None --bias 1 --dropout 0.5 --weight-decay 0 --manifold PoincareBall --log-freq 5 --cuda 0```

 * MLP (Test accuracy=73.00):
  
```python train.py --task nc --dataset pubmed --model MLP --lr 0.01 --dim 16 --num-layers 2 --act None --bias 0 --dropout 0.2 --weight-decay 0.001 --manifold Euclidean --log-freq 5 --cuda 0```

 * GCN (Test accuracy=78.30): 
 
```python train.py --task nc --dataset pubmed --model GCN --lr 0.01 --dim 16 --num-layers 2 --act relu --bias 1 --dropout 0.7 --weight-decay 0.0005 --manifold Euclidean --log-freq 5 --cuda 0```

 * GAT (Test accuracy=78.70): 

```python train.py --task nc --dataset pubmed --model GAT --lr 0.01 --dim 16 --num-layers 2 --act elu --bias 1 --dropout 0.5 --weight-decay 0.0005 --alpha 0.2 --n-heads 4 --manifold Euclidean --log-freq 5 --cuda 0```

 * HyperGCN (Test accuracy=80.40): 
 
First pre-train embeddings for link prediction (Test ROC-AUC: 95.17. For simplicity, we saved pre-trained HyperGCN embeddings in ```data/pubmed/hypergcn.npy```: 
 
``` python train.py --task lp --dataset pubmed --model HyperGCN --lr 0.01 --dim 16 --num-layers 2 --act relu --bias 1 --dropout 0.4 --weight-decay 0.0001 --manifold PoincareBall --log-freq 5 --cuda 0``` 
 
Then train a MLP for node classification on pre-trained embeddings:
 
```python train.py --task nc --dataset pubmed --model GCN --lr 0.01 --dim 16 --num-layers 2 --act relu --bias 1 --dropout 0.2 --weight-decay 0.0005 --manifold Euclidean --log-freq 5 --cuda 0 --use-feats 0 --pretrained-embeddings data/pubmed/hypergcn.npy``` 

## Some of the code was forked from the following repositories

 * [pygcn](https://github.com/tkipf/pygcn/tree/master/pygcn)
 * [gae](https://github.com/tkipf/gae/tree/master/gae)
 * [hyperbolic-image-embeddings](https://github.com/KhrulkovV/hyperbolic-image-embeddings)
 * [pyGAT](https://github.com/Diego999/pyGAT)
 * [poincare-embeddings](https://github.com/facebookresearch/poincare-embeddings)
 * [geoopt](https://github.com/geoopt/geoopt)
 
## Coming soon

 * Hyperboloid implementation of HyperGCN
 * Hyperbolic Attention in HyperGCN
 * Reproducible training commands for other datasets 
 * More efficient negative sampling implementation for link prediction

## References

[1] Chami, I., Ying, R., Ré, C. and Leskovec, J. Hyperbolic Graph Convolutional Neural Networks. NIPS 2019. 

[2] Nickel, M. and Kiela, D. Poincaré embeddings for learning hierarchical representations. NIPS 2017.

[3] Ganea, O., Bécigneul, G. and Hofmann, T. Hyperbolic neural networks. NIPS 2017. 

[4] Kipf, T.N. and Welling, M. Semi-supervised classification with graph convolutional networks. ICLR 2017.

[5] Veličković, P., Cucurull, G., Casanova, A., Romero, A., Lio, P. and Bengio, Y. Graph attention networks. ICLR 2018. 
