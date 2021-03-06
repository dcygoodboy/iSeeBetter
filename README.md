# iSeeBetter: Spatio-Temporal Video Super Resolution using Recurrent-Generative Back-Projection Networks

Project for Stanford CS230: Deep Learning

```Python3 | PyTorch | GANs | CNNs | ResNets | RNNs```

![iSeeBetter_Poster](https://github.com/amanchadha/iSeeBetter/blob/master/AmanChadha_CS230_Poster.jpg)

## Required Packages

```
torch==1.3.0.post2
pytorch-ssim==0.1
numpy==1.16.4
scikit-image==0.15.0
tqdm==4.37.0
```

Also needed is [Pyflow](https://github.com/pathak22/pyflow) which is a Python wrapper for [Ce Liu's C++ implementation](https://people.csail.mit.edu/celiu/OpticalFlow/) of Coarse2Fine Optical Flow.
Pyflow binaries have been built for ubuntu and macOS and are available in the repository.
If you need to rebuild Pyflow, follow the instructions on the [Pyflow Git](https://github.com/pathak22/pyflow) and do a ```cp pyflow*.so ..``` once you have built a shared object file on your target machine.

To load,
```pip3 install -r requirements.txt```

## Elevator Pitch

Deep learning has taken the world by storm! 

Amongst the plethora of fields that deep learning has impacted, super resolution (which by definition is upscaling a low-res sample to a high-res sample) is one of them. 

So why did I chose this topic? I felt that I could use my newly minted DL chops to develop something interesting which might propel the state-of-the-art further in the process. 

Lets start with a low-res video sequence. 
The easiest way to super-resolve such an input low-res video is to apply super resolution to every single frame individually. However, this would be wasteful of the temporal details inherent in video sequences, especially motion patterns.

So I thought why not make my algorithm look left, look right - use details from adjacent images and train it with a GAN to extract fine-grained details such as complex textures.

Presenting iSeeBetter, a novel spatio-temporal approach to video super resolution which uses as its generator a Recurrent Back-Projection Network (RBPN) to extract spatial and temporal information from the current and neighboring frames. 

We use the discriminator within Super-Resolution Generative Adversarial Network (SRGAN) as our discriminator. 

Now, as far as the loss function goes, using Mean Squared Error as a primary loss-minimization objective improves PSNR and SSIM which are important image quality metrics, but these metrics may not capture fine details in the image leading to misrepresentation of perceptual quality. 

To address this, we use a four-fold loss function composed of adversarial loss, perceptual loss, MSE loss and Total-Variation loss. 

Finally, with extensive experimentation, our results demonstrate that iSeeBetter offers superior VSR fidelity and surpasses state-of-the-art performance in the vast majority of SR cases.

## Overview

Recently, learning-based models have enhanced the performance of Single-Image Super- Resolution (SISR). However, applying SISR successively to each video frame leads to lack of temporal coherency. On the other hand, Video Super Resolution (VSR) models based on Convolutional Neural Networks (CNNs) outperform traditional approaches in terms of image quality metrics such as Peak Signal to Noise Ratio (PSNR) and Structural SIMilarity (SSIM). However, Generative Adversarial Networks (GANs) offer a competitive advantage in terms of being able to mitigate the issue of lack of finer texture details when super-resolving at large upscaling factors which is usually seen with CNNs. We present iSeeBetter, a novel spatio-temporal approach to VSR. iSeeBetter seeks to render temporally consistent Super Resolution (SR) videos by extracting spatial and temporal information from the current and neighboring frames using the concept of Recurrent Back-Projection Networks (RBPN) as its generator. Further, to improve the "naturality" of the super-resolved image while eliminating artifacts seen with traditional algorithms, we utilize the discriminator from Super-Resolution Generative Adversarial Network (SRGAN). Mean Squared Error (MSE) as a primary loss-minimization objective improves PSNR and SSIM, but these metrics may not capture fine details in the image leading to misrepresentation of perceptual quality. To address this, we use a four-fold (adversarial, perceptual, MSE and Total-Variation (TV)) loss function. Our results demonstrate that iSeeBetter offers superior VSR fidelity and surpasses state-of-the-art performance. 
 
![adjacent frame similarity](https://github.com/amanchadha/iSeeBetter/blob/master/images/iSeeBetter_AFS.jpg)
<p align="center">Figure 1: Adjacent frame similarity</p>
 
![network arch](https://github.com/amanchadha/iSeeBetter/blob/master/images/iSeeBetter_NNArch.jpg)
<p align="center">Figure 2: Network architecture</p>

## Model Architecture

Figure 2 shows the iSeeBetter architecture which uses RBPN and SRGAN as its generator and discriminator respectively. RBPN has two approaches that extract missing details from different sources, namely SISR and MISR. Figure 3 shows the horizontal flow (blue arrows in Figure 2) that enlarges LR(t) using SISR. Figure 4 shows the vertical flow (red arrows in Figure 2) which is based on MISR that computes residual features from a pair of LR(t) to neighbor frames (LR(t−1), ..., LR(t−n) and the flow maps (F(t−1), ..., F(t−n)). At each projection step, RBPN observes the missing details from LR(t) and extracts residual features from neighboring frames to recover details. Within the projection models, RBPN utilizes a recurrent encoder-decoder mechanism for incorporating details extracted in SISR and MISR through back-projection.

![ResNet_MISR](https://github.com/amanchadha/iSeeBetter/blob/master/images/ResNet_MISR.jpg)
<p align="center">Figure 3: ResNet architecture for MISR that is composed of three tiles of five blocks where each block consists of two convolutional layers with 3 x 3 kernels, stride of 1 and padding of 1. The network uses Parametric ReLUs for its activations.</p>

![DBPN_SISR](https://github.com/amanchadha/iSeeBetter/blob/master/images/DBPN_SISR.png)
<p align="center">Figure 4: DBPN architecture for SISR, where we perform up-down-up sampling using 8 x 8 kernels with stride of 4, padding of 2. Similar to the ResNet architecture above, the DBPN network also uses Parametric ReLUs as its activation functions.</p>

![Disc](https://github.com/amanchadha/iSeeBetter/blob/master/images/Disc.jpg)
<p align="center">Figure 5: Discriminator Architecture from SRGAN. The discriminator uses Leaky ReLUs for computing its activations.</p>

## Dataset

To train iSeeBetter, we amalgamated diverse datasets with differing video lengths, resolutions, motion sequences and number of clips. Table 1 presents a summary of the datasets used. When training our model, we generated the corresponding LR frame for each HR input frame by performing 4x down-sampling using bicubic interpolation. We also applied data augmentation techniques such as rotation, flipping and random cropping. To extend our dataset further, we wrote scripts to collect additional data from YouTube, bringing our dataset total to about 170,000 clips which were shuffled for training and testing. Our training/validation/test split was 80\%/10\%/10%.

Get the [SPMCS and Vid4 dataset](https://drive.google.com/drive/folders/1sI41DH5TUNBKkxRJ-_w5rUf90rN97UFn?usp=sharing) and the [Vimeo90K dataset](http://data.csail.mit.edu/tofu/dataset/vimeo_septuplet.zip). You can also use ```DatasetFetcher.py``` to get Vimeo90K.

![results](https://github.com/amanchadha/iSeeBetter/blob/master/images/Dataset.jpg)
<p align="center">Table 1. Datasets used for training and evaluation</p>

## Results

We compared iSeeBetter with six state-of-the-art VSR algorithms: DBPN, B123 + T, DRDVSR, FRVSR, VSR-DUF and RBPN/6-PF.

![results2](https://github.com/amanchadha/iSeeBetter/blob/master/images/Res1.jpg)
<p align="center">Table 2. Visually inspecting examples from Vid4, SPMCS and Vimeo-90k comparing RBPN and iSeeBetter. We chose VSR-DUF for comparison because it was the state-of-the-art at the time of publication. Top row: fine-grained textual features that help with readability; middle row: intricate high-frequency image details; bottom row: camera panning motion.</p>

![results1](https://github.com/amanchadha/iSeeBetter/blob/master/images/Res2.jpg)
<p align="center">Table 3. PSNR/SSIM evaluation of state-of-the-art VSR algorithms using Vid4 for 4x. Bold numbers indicate best performance.</p>

![results3](https://github.com/amanchadha/iSeeBetter/blob/master/images/Res3.jpg)
<p align="center">Table 4. PSNR/SSIM evaluation of state-of-the-art VSR algorithms using Vimeo90K for 4x. Bold numbers indicate best performance.</p>

## Pretrained Model
Model trained for 4 epochs included under ```weights/```

## Usage

### Training 

Train the model using:

```python iSeeBetterTrain.py```

(takes roughly 1.5 hours per epoch with a batch size of 2 on an NVIDIA Tesla V100 with 16GB VRAM)

### Testing

To use the pre-trained model and test on a random video from within the dataset:

```python iSeeBetterTest.py```

## Acknowledgements

Credits:
- [SRGAN Implementation](https://github.com/leftthomas/SRGAN) by LeftThomas.
- We used [RBPN-PyTorch](https://github.com/alterzero/RBPN-PyTorch) as a baseline for our Generator implementation.

## Citation
Cite the work as:
```
Publication WIP
```
