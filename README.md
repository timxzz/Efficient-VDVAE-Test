
# The Official Pytorch and JAX implementation of "Efficient-VDVAE: Less is more"" [Arxiv preprint]()  
  
<div align="center">  
  <a>Ragavan&nbsp;Thurairatnam</a>   
  &emsp; <b>&middot;</b> &emsp;  
  <a>Ragavan&nbsp;Thurairatnam</a>   
  &emsp; <b>&middot;</b> &emsp;  
  <a>Ragavan&nbsp;Thurairatnam</a>  
</div>  
<br>  
<br>  
  
[Efficient-VDVAE]() is a memory and compute efficient very deep hierarchical VAE. It converges faster and is more stable than current   
hierarchical VAE models. It also achieves SOTA likelihood-based performance on several image datasets.  
  
<p align="center">
    <img src="images/unconditional_samples.png" width="1200">
</p>

## Pre-trained model checkpoints

We provide checkpoints on MNIST, CIFAR-10, Imagenet 32x32, Imagenet 64x64, CelebA 64x64, CelebAHQ 256x256 (5-bits and 8-bits), FFHQ 256x256 (5-bits and 8bits), CelebAHQ 1024x1024 and FFHQ 1024x1024 in [this Google drive directory](). We also provide the training logs of these models in the same drive. 

Note: Some of these models are missing in either Pytorch or JAX for the time being. We will update them over time.

## Pre-requisites 

To run this codebase, you need:

- Machine that runs a linux based OS (tested on Ubuntu 20.04 (LTS))
- GPUs (preferably more than 16GB)
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- Python 3.7 or higher
- CUDA 11.1 or higher (can be installed from [here](https://developer.nvidia.com/cuda-11.1.0-download-archive))

## Installation

To create the docker image used in both the Pytorch and JAX implementations:

```
cd build
docker build -t efficient_vdvae_image .
```

All code executions should be done within a docker container. To start the docker container, we provide a utility script:

```
sh docker_run.sh
```
## Setup datasets

TODO

## Setting the hyper-parameters

In this repository, we use [hparams](https://github.com/Rayhane-mamah/hparams) library (already included in the Dockerfile) for hyper-parameter management:

- Specify all run parameters (number of GPUs, model parameters, etc) in one ".cfg" file
- Hparams saves the configuration of previous runs for reproducibility, resuming training, etc.
- All hparams are saved by name, and re-using the same name will recall the old run instead of making a new one.
- The ".cfg" file is split into sections for readability, and all parameters in the file are accessible as class attributes in the codebase for convenience.
- The HParams object keeps a global state throughout all the scripts in the code.

We highly recommend having a deeper look into how this library works by reading the [README file](https://github.com/Rayhane-mamah/hparams)  before trying to run Efficient-VDVAE.

## Training the Efficient-VDVAE

To run Efficient-VDVAE in Torch:

```
cd torch
# Set the hyper-parameters in "hparams.cfg" file
# Set "NUM_GPUS_PER_NODE" in "train.sh" file
sh train.sh
```

To run Efficient-VDVAE in JAX:

```
cd jax
# Set the hyper-parameters in "hparams.cfg" file
python train.py
```

If you want to run the model with less GPUs than available on the hardware, for example 2 GPUs out of 8:

```
CUDA_VISIBLE_DEVICES=0,1 sh train.sh  # For torch
CUDA_VISIBLE_DEVICES=0,1 python train.py  # For JAX
```

Models automatically create checkpoints during training. To resume a model from its last checkpoint, set its *`<run.name>`* in *`hparams.cfg`*  file and re-run the same training commands.

Since training commands will save the hparams of the defined run in the ".cfg" file. If trying to restart a pre-existing run (by re-using its name in "hparams.cfg"), we provide a convenience script for resetting saved runs:

```
cd torch  # or cd jax
sh reset.sh <run.name>  # <run.name> is the first field in hparams.cfg
```

To make things easier for new users, we provide example "hparams.cfg" files that can be used under the [egs]() folder. Detailed description of the role of each parameter is also inside [hparams.cfg]().

## Monitoring the training process

While writing this codebase, we put extra emphasis on verbosity and logging. Aside from the printed logs on terminal (during training), you can monitor the training progress and keep track of useful metrics using Tensorboard:

```
# While outside torch or jax
tensorboard --logdir . --port <port_id> --reload_multifile=True
```

## Inference with the Efficient-VDVAE

Efficient-VDVAE support multiple inference modes:

- "reconstruction": Encodes then decodes the test set images and computes test NLL and SSIM.
- "generation": Generates random images from the prior distribution. Randomness is controlled by the "run.seed" parameter.
- "div_stats": Pre-computes the average KL divergence stats used to determine turned-off variates (refer to section 7 of the [paper]()). Note: This mode needs to be run before "encoding" mode and before trying to do masked "reconstruction" (Refer to [hparams.cfg]() for a detailed description).
- "encoding": Extracts the latent distribution $q_\phi(z|x)$, pruned to the quantile defined by "synthesis.variates_masks_quantile" parameter. This latent distribution is usable in downstream tasks.

To run the inference:

```
cd torch  # or cd jax
# Set the inference mode in "logs-<run.name>/hparams-<run.name>.cfg"
# Set the same <run.name> in "hparams.cfg"
python synthesize.py
```

### Notes:
- Since training a model with a name *`<run.name>`* will save that configuration under *`logs-<run.name>/hparams-<run.name>.cfg`* for reproducibility and error reduction. Any changes that one wants to make during inference time need to be applied on the saved hparams file (*`logs-<run.name>/hparams-<run.name>.cfg`*) instead of the main file *`hparams.cfg`*.
- The torch implementation currently doesn't support multi-GPU inference. The JAX implementation does.

## Bibtex

TODO