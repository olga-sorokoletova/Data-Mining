# GAN: generating brushstroke paintings

This is the repository of the project proposed for the course of *Data Mining*.

This work is centered on the idea of **Intrinsic Style Transfer** from the paper by [Reiichiro Nakano, **"Neural Painters: A learned differentiable constraint for generating brushstroke paintings"** (2019)](https://arxiv.org/abs/1904.08410) and [**MyPaintBrushstrokes**](https://www.kaggle.com/reiinakano/mypaint_brushstrokes) dataset.

## Introduction

The core idea behind the Neural Painters technology is to provide **the painter agent** trained to generate the source image as a painting in a stroke-by-stroke fashion as a human artist would do.

```Input: source image.```

```Output: painting of this image by brushstrokes```.

The following **4 source images** have been borrowed from the repository of the original paper and preprocessed to be for testing of the built models and experimenting with the results:

<p align="center">
  <img src="/images/source_images/vanc.jpg" width="247" height="151"/><img src="/images/source_images/louvre.png" width="190" height="151"/> <img src="/images/source_images/eiffel.jpeg" width="151" height="151"/> <img src="/images/source_images/castle.png" width="151" height="151"/> 
</p>

### Dataset
Dataset is not stored in the project repository due to its large memory consumption, but can be downloaded from the corresponding Kaggle web-page [**MyPaintBrushstrokes**](https://www.kaggle.com/reiinakano/mypaint_brushstrokes) as:

```!kaggle datasets download -d reiinakano/mypaint_brushstrokes```

It is structured in the following way:

|        Name         | N samples | File size |
| -----------------   |:---------:|:---------:|
|```episodes_0.npz``` |   100000  |  196.3MB  |
|        ...          |    ...    |    ...    |
|```episodes_77.npz```|   100000  |  196.3MB  |

```episodes_0.npz```, ```episodes_1.npz```, ```episodes_2.npz``` were mainly used in the project.


Each data sample is represented by **action** and the **stroke**. 
From the semantical point of view given action realizes given stroke, and

- **action:** array of length ```12```,
- **stroke:** RGB image with ```(img_width, img_height) = (64, 64)```.

Examples of the stroke images are printed below:

<p align="center">
  <img src="/images/readme_images/dataset_examples.png" width="640" height="320"/>
</p>

### Implementation

Implementation is split into 4 Colaboratory notebooks containing each its own model. 

Code contained in each notebook performs:
- *loading* of the appropriate input data,
- *defining the model*,
- *training*,
- *printing/saving* of the results.

These models are the self-sufficient components of an **inheritive 4-block architecture** of the painting agent, which is described in details in the following section. 

Notebooks are stored in a [```scripts/```](https://github.com/Olllga/DM_project/tree/main/scripts) folder:
1. [generator.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/generator.ipynb) - code for the **Generator** model training;
2. [discriminator.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/discriminator.ipynb) - code for the **Discriminator** model training;
3. [gan.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/gan.ipynb) - code for the **Generative Adversarial Network** training;
4. [paint_image.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/paint_image.ipynb) - code for the **Intrinsic Style Transfer** model training.

Best weights of the models are available for loading from the [```models/```](https://github.com/Olllga/DM_project/tree/main/models) folder.

## Models

In principal, Neural Painter could consist of two models only: [gan.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/gan.ipynb) and [paint_image.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/paint_image.ipynb). However, throughout the project evolvement **3 important conceptual decisions** have been taken, and [generator.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/generator.ipynb) and [discriminator.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/discriminator.ipynb) present one of them: ***Transfer Learning***.
___

***Concept 1: Transfer Learning***

**Idea:** The underlined idea of Transfer Learning techniique is that model can be *pre-trained on a related task* and then loaded to a given task with already learnt parameters, and then only fine-tuning to adapt to an original task is needed. 

**Motivation:** Training GAN is typically expensive in a sense of time and memory consumption, and therefore *needs to be accelerated*/performed in optimal way. This is especially demanding in a situation when *resources are limited*, such as performing training using Google Colab, where users are allocated with a temporal token on GPU.

**Realization:** In our case, at first, the *Generator* network is pre-trained independently to learn how to *produce stroke images given action* arrays. Then at the second step, the *Discriminator* network is again an independent unit, which given strokes from dataset and generated by the Generator from the previous step strokes is trained *as binary classifier*. Finally, *both Generator and Discriminator are loaded as pre-trained* at the third step and combined into a GAN to be trained in an adversarial manner.

**Reference:** [fastai.ai](https://www.fast.ai/2019/05/03/decrappify/).

___

### Model 1: Generator

#### Code and best learnt weights: 
[generator.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/generator.ipynb), [generator parameters](https://github.com/Olllga/DM_project/blob/main/models/generator_5152w_15e_1d_best.pth). 

#### Architecture

```Input: actions from dataset and images of brushstrokes from the dataset.```

```Output: generated images of brushstrokes```.

<p align="center">
  <img src="/images/readme_images/block_schemes/generator.png" width="574" height="535"/>
</p>

- **Generator block:** fully-connnected layer + CNN.
- **Feature extractor:** to extract image features VGG19 is loaded as pre-trained.
- **Feature loss:** is another important technology which is used in the project.

___

***Concept 2: Feature (Perceptual) Loss***

**Idea:** Perceptual loss functions based not on differences between pixels but instead on differences between *high-level image feature representations* extracted from pretrained convolutional neural networks. 

**Motivation:** To measure distance between images while capturing perceptual differences. Perceptual losses measure image similarities more *robustly* than per-pixel losses.

**Realization:** *Feature Reconstruction Loss* + Style Reconstruction Loss + Pixel Loss.

**Reference:** [Perceptual Loss Paper](https://github.com/Olllga/DM_project/blob/main/materials/Perceptual%20Loss%20Paper.pdf).

___

#### Results

##### Generated Strokes

<p align="center">
  <img src="/images/gen_strokes_images/generator_5152w_15e_1d_best.jpeg" width="1024" height="539"/>
</p>

##### Train Loss

<p align="center">
  <img src="/images/loss_plots/generator_15e_4seed.png" width="376" height="278"/>
</p>

### Model 2: Discriminator

#### Code and best learnt weights: 
[discriminator.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/discriminator.ipynb), [discriminator parameters](https://github.com/Olllga/DM_project/blob/main/models/discriminator_10e.pth.pth).

#### Architecture

```Input: generated by Model 1 images of brushstrokes and images of brushstrokes from the dataset, labels.```

```Output: class prediction (0/1: fake/true)```.

<p align="center">
  <img src="/images/readme_images/block_schemes/discriminator.png" width="906" height="538"/>
</p>

- **Generator block:** pre-trained Model 1 with freezed weights.
- **Discriminator block:** CNN.
- **BCE loss:** Binary Cross-Entropy.

#### Results

##### Train Loss

<p align="center">
  <img src="/images/loss_plots/discriminator_2e_10seed.png" width="392" height="278"/>
</p>

### Model 3: GAN

#### Code and best learnt weights: 
[gan.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/gan.ipynb), [GAN generator parameters](https://github.com/Olllga/DM_project/blob/main/models/GAN_generator_2e.pth), [GAN discriminator parameters](https://github.com/Olllga/DM_project/blob/main/models/GAN_discriminator_2e.pth). 

#### Architecture

```Generator Input: actions from dataset and images of brushstrokes from the dataset.``` 

```Discriminator Input: generated by Generator images of brushstrokes and images of brushstrokes from the dataset, labels (true or fake).```

```Generator Output: generated images of brushstrokes```.

```Discriminator Output: class prediction (0/1: fake/true)```.

<p align="center">
  <img src="/images/readme_images/block_schemes/gan.png" width="960" height="540"/>
</p>

- **Generator block:** pre-trained Model 1.
- **Discriminator block:** pre-trained Model 2.
- **Feature extractor:** to extract image features VGG19 is loaded as pre-trained.
- **Adversarial loss:** Perceptual and BCE loss are both involved.

#### Results

##### Generated Strokes

<p align="center">
  <img src="/images/gen_strokes_images/GAN_generator_2e.jpeg" width="1082" height="192"/>
</p>

##### Train Loss

<p align="center">
  <img src="/images/loss_plots/gan_2e_4seed.png" width="378" height="278"/>
</p>

### Model 4: Painter

#### Code: 
[paint_image.ipynb](https://github.com/Olllga/DM_project/blob/main/scripts/paint_image.ipynb).

Before start of training of the painter agent itself, some mechanism for **Blending** is needed.

___

***Concept 3: Blending***

**Idea:** It is a mechanism *to apply successively brushstrokes to a canvas*. 

**Motivation:** Painted image consists of a collection of mixed brushstrokes. The model that produces brushstrokes has been obtained, and its indeed the core component of the Neural Painter, but some instruments to be able to paint are still missing. One of them: methodology of combining individual brushstrokes.

**Realization:** We calculate *the opacity of each pixel* in a brushstroke by computing how dark it is relative to the darkest pixel (full opacity) in the brushstroke and then *use this value as a ratio for blending* the stroke with the existing paint on the canvas.

**Reference:** [Web-article about Teaching agents to paint](https://reiinakano.com/2019/01/27/world-painters.html).

Examples of the blended stroke images (32 dataset samples) are printed below:

<p align="center">
  <img src="/images/painted_images/blending.png" width="512" height="256"/>
</p>

___

#### Architecture

```Input: random actions and source image.```

```Output: stroke-by-stroke painting for the source image```.

<p align="center">
  <img src="/images/readme_images/block_schemes/paint.png" width="579" height="540"/>
</p>

- **Generator block:** loaded Generator from the Model 3.
- **Feature extractor:** to extract image features VGG19 is loaded as pre-trained.
- **Feature loss:** is used to update actions.

#### Results

##### Generated Strokes

<p align="center">
  <img src="/images/painted_images/painted_vanc_50s_3000e_4seed.png" width="64" height="64"/><img src="/images/painted_images/painted_louvre_50s_3000e_4seed.png" width="64" height="64"/><img src="/images/painted_images/painted_eiffel_50s_3000e_4seed.png" width="64" height="64"/><img src="/images/painted_images/painted_castle_50s_3000e_4seed.png" width="64" height="64"/>
</p>

##### Train Loss

<p align="center">
  <img src="/images/loss_plots/paint_50s_3000e_4seed.png" width="393" height="278"/>
</p>












## Author
- Olga Sorokoletova - 1937430
