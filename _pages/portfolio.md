---
layout: page
title: portfolio
permalink: /portfolio/
description: In God we trust, all others bring data. - William Edwards Deming (1900-1993)
nav: true
nav_order: 4
display_categories: []
horizontal: false
images:
  compare: true
  slider: true
---

<style>
  .coloured-slider {
    --divider-color: rgba(0, 0, 0, 0.5);
    --default-handle-color: rgba(0, 0, 0, 0.5);
  }
</style>

---
## **Publications**

<br>

### 1. [ICLR 2024] Compose and Conquer: Diffusion-Based 3D Depth Aware Composable Image Synthesis

<br>

Conditional diffusion models usually take in text, and two different types of conditions to generate an image.

1. **Local conditions**, which conditions the model on structural information through primitives like depth maps and canny edges.
2. **Global conditoins**, which conditions the model on semantic information (color, identity, texture, etc).

However, these models suffer from two limitations: they lack the ability to disentangle the relative depth of multiple local conditions,
and are inable to localize global conditions to a specific area.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/cnc/1.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

- **Limitation 1:** Suppose you'd like to generate an image of two puppies with one in front of another that have the shape of the depthmaps. 
Models aren't able to distinguish that one puppy has to be placed in front of the other, resulting in a fused image.
- **Limitation 2:** How about an elephant standing in a forest, that shares the shape (depthmap) and semantic of the forest image?
Models can't tell that the image semantics are supposed to go behind (or around) the elephant, resulting in the elephant being completely ignored.


We sought to create a model that was able to **1. Distinguish between where objects should be placed in relative depths, and 2. Localize global semantics
onto a user-defined area**, not just the whole image.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/cnc/2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We solved this through proposing a training paradigm and a novel inference algorithm: **Depth Disentanglement Training** and **Soft Guidance**.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/cnc/ddt.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

**Depth Disentanglement Training** parses the training data into three sub-components: the foreground, background, and mask via inpainting.
The background image serves as structural information that's initially occluded by the salient object, forcing the model to learn what's **behind** another object.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/cnc/soft_guidance.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

**Soft Guidance** partially masks out the pixels that are attending to certain semantics in the cross-attention layers of the model, forcing the model to
attend to **parts of the pixels** that correspond to their semantic counterparts.

<img-comparison-slider class="coloured-slider">
  {% include figure.liquid path="assets/img/portfolio/cnc/comparison_after.jpg" class="img-fluid rounded z-depth-1" slot="first" %}
  {% include figure.liquid path="assets/img/portfolio/cnc/comparison_before.jpg" class="img-fluid rounded z-depth-1" slot="second" %}
</img-comparison-slider>
<div class="caption">
    Qualitative/Quantitative comparisons of CnC against baseline models.
</div>

By training our model **Compose and Conquer (CnC)** with Depth Disentanglement Training and Soft Guidance, we've created a model that **outperforms
current conditional-generation SOTA models** both qualitatively and quantitatively!


<br>

---

### 2. [ICLR 2024] Noise Map Guidance: Inversion with Spatial Context for Real Image Editing

<br>

Diffusion models are also refered to as Score-Based Models, due to its recent findings of how they're also able to predict the score of random variables.
These predicted scores can be utilized directly into a PF-ODE (Probability Flow ODE) that diffusion models follow: providing ways to encode/decode images
without any stochasticity. This is favorable for editing real images through diffusion models: The only way to manipulate real images are to first encode them into latents that will result in the image itself after solving the PF-ODE. 

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/nmg/nmg.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

However, there's a caveat. Image editing methods rely on the use of Classifier-Free Guidance (CFG), which results in the reconstruction PF-ODE to diverge from
its original trajectory. Past inversion methods try to overcome this by optimizing each encoded latent in the PF-ODE trajectory, which is extremely slow,
even for diffusion models.

$$
\begin{gathered}
\tilde{\epsilon_\theta}\left(\boldsymbol{z}_t^{N M}, c_T\right)=\epsilon_\theta\left(\boldsymbol{z}_t^{N M}, \emptyset\right)+s_T \cdot\left(\epsilon_\theta\left(\boldsymbol{z}_t^{N M}, c_T\right)-\epsilon_\theta\left(\boldsymbol{z}_t^{N M}, \emptyset\right)\right) \\
\boldsymbol{z}_{t-1}=\sqrt{\frac{\alpha_{t-1}}{\alpha_t}} \boldsymbol{z}_t^{N M}+\sqrt{\alpha_{t-1}}\left(\sqrt{\frac{1}{\alpha_{t-1}}-1}-\sqrt{\frac{1}{\alpha_t}-1}\right) \tilde{\epsilon_\theta}\left(\boldsymbol{z}_t^{N M}, c_T\right)
\end{gathered}
$$

To overcome the slow process of optimizing each latent, we leverage the **noise maps**, intermediate latent representations of the PF-ODE from the timestep before
in re-traversing the reconstruction PF-ODE path. As mentioned, this allows **optimization-free** inversion of real images, while also being robust to the
structure of the original image.

<img-comparison-slider class="coloured-slider">
  {% include figure.liquid path="assets/img/portfolio/nmg/nmg_left.jpg" class="img-fluid rounded z-depth-1" slot="first" %}
  {% include figure.liquid path="assets/img/portfolio/nmg/nmg_right.jpg" class="img-fluid rounded z-depth-1" slot="second" %}
</img-comparison-slider>
<div class="caption">
    Qualitative/Quantitative comparisons of CnC against baseline models.
</div>

We found that using our proposed method Noise Map Guidance (NMG), existing real image editing methods were drastically improved
while being **20 times faster!**

<br>

---

### 3. [CVPR 2024] One-Shot Structure-Aware Stylized Image Synthesis

<br>

Diffusion models are considered very attractive in the domain of stylization due to its potency and expressiveness.
While gan based methods