# Classification for Alzheimer's disease of the ADNI brain data

## Problem this project solves
This project utilises a modified dataset sourced from [ADXI dataset for Alzheimer's disease](https://adni.loni.usc.edu/). The dataset contains two repositories which are AD(disease) and NC(normal) and each repository contains training and test repositories. This project aims to utilise the Vision Transformer to classify healthy brain images and unhealthy images.

## Description of the Algorithm  
The vision transformer is the algorithm posed by the thesis *An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale* ([Alexey Dosovitskiy,2020](https://arxiv.org/abs/2010.11929)). This algorithm utilises the concept ***self-attention*** mechanism, which receives an excellent performance in the field of NLP. This algorithm can be demonstrated in the following figure.  
 
![Alt text](https://drive.google.com/uc?export=view&id=14jUX7ixXWCY0y7ByJCOEvVpPaaeMIlmW)  
  
### Algorithm Workflow
Similar to the token in the NLP, ViT segments the image into several small patches. For example under the original ViT base/16 algorithm, the input image size is 224x224 and the 16 means the algorithm segments the image to 16 patches on each height and width. Hence, the patch size is 224/16=14. To be specific, this transformation is implemented by the convolutional operation with the kernel size of 16, stride of 16 and with the output channel of 768. After the transformation, the image dimension changed from 224x224x3 to 14x14x768. A similar transformation is shown in the following figure.  
![Alt text](https://drive.google.com/uc?export=view&id=1zM8QXAyIXppFKzqFq4DKAGMA5FWAF0RJ)  
  
After segmenting patches, the ViT flatten all the patches because the ViT will utilise the multi-head self-attention in the following Transformer Encoder block and this block requires a 2-D input. In addition, after the flattening operation, the ViT add class token and position embedding for future classification task and records the position information of each patch. Hence the input shape transform as: Flatten (14x14x768->196x768), add class token (196x768->197x768), add position embedding (add operation) (197x768->197x768). A similar transformation is shown in the following figure.   
![Alt text](https://drive.google.com/uc?export=view&id=1da1kpkKhMDaAxfQ1Sa7N9Nd4bOvc52Rv)  
  
The transformer block is the most important part of the ViT and it utilises the Multi-head self-attention mechanism to extract features. First of all, I will explain the mechanism of the self-attention mechanism [Ashish Vaswani,2017](https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf).  
The attention utilises 3 parameters to calculate:  
  
q: query(to match others)  
$$q^i=a^iW^q$$  
  
k: key(to be matched)  
$$k^i=a^iW^k$$  
  
v: information to be extracted  
$$v^i=a^iW^v$$  
  
$$Attention\left( Q,K,V \right) =softmax\left( \frac{QK^T}{\sqrt{d_k}} \right) V$$  
Note: the $d_k$ is the dimension of the key.
  
The overall calculation can be divided into 3 procedures.
* Calculate the q k and v  
The following figure shows how to computer these three parameters and the W is the parameter that can be trained in the network.

![Alt text](https://drive.google.com/uc?export=view&id=1EEkemp5WAcJs6qu9cC-Fc3CyiNkDlanv)  

* Scaled Dot-Production Attention  
This is the part to calculate the $softmax\left( \frac{QK^T}{\sqrt{d_k}} \right)$, the reason for using the softmax function is that after the dot product, the value will very larger. Utilising the softmax can make the gradient in a small value.  The detailed formula like:  
$a_{1,i}=\frac{q^1\cdot k^i}{\sqrt{d_k}}$  $a_{2,i}=\frac{q^2\cdot k^i}{\sqrt{d_k}}$
  
The following figure shows how to computer the Scaled Dot-Production Attention:  
![Alt text](https://drive.google.com/uc?export=view&id=1QVbwZQ5rl1J39RjjyuUZRQeOAg7_S4R2)
  
* Overall Attention value Computing      
After the second procedure, we can get the value of $a_{1,i}$ and $a_{2,i}$. In this procedure, we will get the final result of the attention procedure. The following formula shows how to compute it:  
$$b^1=\sum_i{a_{1,i}}\times v^i$$
$$b^2=\sum_i{a_{2,i}}\times v^i$$  
  
The following figure shows how to computer the Attention value:  
![Alt text](https://drive.google.com/uc?export=view&id=1mMrHKzyU21PVudcRdVF2OKeP0tNs0pRn)
  
After the description of the self-attention mechanism, the difference between multi-head self-attention and self-attention is that the multi-head self-attention average split the value of Q, K and V to the number of head. The calculation of each head also follows the procedure of the self-attention mechanism. The following is the formula of the multi-head self-attention mechanism.   
$$MultiHead\left( Q,K,V \right) =Concat\left( head_1,\cdots head_h \right) W^O$$
$$head_i=Attention\left( QW_{i}^{Q},KW_{i}^{K},VW_{i}^{V} \right)$$
  
The following is an example of a two-head self-attention computation workflow:  
![Alt text](https://drive.google.com/uc?export=view&id=1taipUHj1mNU2LxhMQsOAywPDqKHpiRLV)
  
After the Transformer Encoder block, the ViT utilises an MLP block for the further classification task.  

## How it works
### Documents detail
│  dataset.py  
│  modules.py  
│  predict.py  
│  processing.py  
│  README.md  
│  train.py  
│     
├─config  
│  │  config.py  
│  │  
│  └─__pycache__  
├─model  
│  │  resnet.py  
│  │  vit.py  
│  │  __init__.py  
│  │  
│  └─__pycache__  
└─utils  
    │  utilis_.py  
    │  utilis_augs.py  
    │  utils_fit.py  
    │  utils_loss.py  
    │  utils_model.py  
    │  __init__.py  
    │  
    └─__pycache__  

The function of each document:
* processing.py  
This document helps segment the training set to training and validation set by patient ID. The training set contains 80% patients of the original dataset and the validation set contains the other 20%. In addition, it also changes the storing format to the same format as the ImageNet. 
Example:    
Before:      
├─dataset  
│  ├─train  
│  │  ├─AD  
│  │  └─NC  
│  └─test  
│     ├─AD  
│     └─NC  
After:      
├─dataset   
│  ├─train             // 80% of the original train set  
│  │  ├─0  
│  │  └─1  
│  ├─val               // 20% of the original train set  
│  │  ├─0  
│  │  └─1  
│  ├─test  
│  │  ├─0  
│  │  └─1  
│  └─ label.txt        // store the label information  

* dataset.py  
This document provides the image transformation for the training, validation and test data and provides data augmentation for the training data. In addition, this document also provides the built-in dataloader function to load these three data sets.  
For the image preprocessing of the training set, firstly, this project utilises the `CenterCrop` function to resize the image to *224x224*. Secondly, `RandomHorizontalFlip` and `RandomHorizontalFlip` are utilised for the augmentation operation. Thirdly, after transforming the image to the tensor with `ToTensor`, this project utilises a function `get_mean_and_std` to calculate the *mean* and *standard deviation* of the data in the training set. Finally, this project added some *GaussianNoise* to the image in the training set with the *mean* of 0 and the *standard deviation* of 0.05 for the classifier to have more robustness on the test set.  
For the image preprocessing of the validation and test set, firstly, this project also utilises the `CenterCrop` function to resize the image to *224x224*. Secondly, after transforming the image to the tensor with `ToTensor`, this project utilises a function `get_mean_and_std` to calculate the *mean* and *standard deviation* of the data in the validation and test set.  
  
* modules.py  
This document uses the `select_model` function in `utils/utils_model` to select the model that is set up in the `opt.model_name`. To be specific, the vit model is stored in the `model/vit.py`. By utilising this structure, the user can add different models to the repository `model` and utilise them in the future.  
Note:  
When adding a new model to the repository `model`, the user should import the added document in `model/__init__.py`.  
Example: if the user adds a resnet to the model they should add the following code to the `model/__init__.py`.  
```
from .resnet import *
```
  
* train.py  
This document is the main document for training and validating purposes. It utilises the `dataset.py` to transform and load the images and utilises the `modules.py` to load the vit model. In addition, this document provides various functions including `resume`, `mix precision`, `warm up`, `early stopping` and `learning rate scheduler`.  
This document utilises the `fitting` function in `utils/utils_fit` for training and evaluation purposes. For the training phase, this document updates the *loss*, and *optimizer* on each batch size. For the evaluation phase, this document calculates the validating loss and accuracy on each epoch.  
During the training and validating procedure, this document also provides information on the *training loss*, *validating loss*, *learning rate*, *train acc* and *validating acc* etc. on each epoch. In addition, it stores the best parameters of the model on the given `opt.save_path`.  
  
* predict.py  
This document is the main document for testing purposes. The document *train.py* stores the model parameters of the best epoch on the *save_path* as the name of `best. pt`. This document will load the parameters to the model for the future prediction task. In addition, it also utilises the `dataset.py` to transform and load the test images.  
For the prediction, the model loads the weight from `best.pt` and predicts the result under the inference mode. In addition, this document will store the *accuracy*, *f1-score*, *auc*, *recall* and *precision* for each class and the overall *accuracy*, *recall*, *precision* and *f1-score* with a txt file in the path of `opt.save_path`.   
This document also provides an option to store the detailed information of the correctly predicted images and incorrectly predicted images by adding `--visual` when running the *predict.py*. In addition, it also provides the option to visualize the *tsne* plot by adding `--tsne`.
  
### Model architecture

### Dependencies required
python 3.9  
pytorch 2.0.1  
torchvision  
tqdm  
numpy  
scikit-learn  
matplotlib  
prettytable  
grad-cam  
opencv-python  
pillow  

Installation Example
```
pip install grad-cam
```
### How to run  
  
Train and Validate  
```
python train.py --config config/config.py --save_path runs/vit --lr 1e-4 --warmup --amp  --batch_size 128 --epoch 300
```

Predict
```
python predict.py --task test --save_path runs/vit --visual --tsne
```

### Result demonstration