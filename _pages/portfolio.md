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

### **1. [ICLR 2024] Compose and Conquer: Diffusion-Based 3D Depth Aware Composable Image Synthesis**
**First Author, accepted to ICLR 2024 - 2024.01.16**

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

[Paper link](https://arxiv.org/abs/2401.09048), [Code link](https://github.com/tomtom1103/compose-and-conquer/)

Tools used:
[![](https://img.shields.io/badge/PyTorch-white?style=flat-square&logo=Pytorch)]()
[![](https://img.shields.io/badge/Docker-white?style=flat-square&logo=Docker)]()
[![](https://img.shields.io/badge/Bash-white?style=flat-square&logo=GNU Bash)]()
[![](https://img.shields.io/badge/CUDA-white?style=flat-square&logo=NVIDIA)]()
[![](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=OpenCV)]()
[![](https://img.shields.io/badge/Git-white?style=flat-square&logo=Git)]()
[![](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=NumPy)]()


<br>

---

### **2. [ICLR 2024] Noise Map Guidance: Inversion with Spatial Context for Real Image Editing**
**Second Author, accepted to ICLR 2024 - 2024.01.16**

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

[Paper link](https://arxiv.org/abs/2402.04625), [Code link](https://github.com/hansam95/NMG)

Tools used:
[![](https://img.shields.io/badge/PyTorch-white?style=flat-square&logo=Pytorch)]()
[![](https://img.shields.io/badge/Docker-white?style=flat-square&logo=Docker)]()
[![](https://img.shields.io/badge/Bash-white?style=flat-square&logo=GNU Bash)]()
[![](https://img.shields.io/badge/CUDA-white?style=flat-square&logo=NVIDIA)]()
[![](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=OpenCV)]()
[![](https://img.shields.io/badge/Git-white?style=flat-square&logo=Git)]()
[![](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=NumPy)]()

<br>

---

### **3. [CVPR 2024] One-Shot Structure-Aware Stylized Image Synthesis**
**Second Author, accepted to CVPR 2024 - 2024.02.27**

<br>

Diffusion models are considered very attractive in the domain of stylization due to its potency and expressiveness.
While GAN based stylization methods have been thoroughly explored, stylization via diffusion models are relatively under explored.
GAN based stylization methods often fail in preserving rarely-seen attributes in images, and Diffusion based stylization methods
require many style images in order to perform stylization (Multi-shot).

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/osasis/osasis.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Note: Me and the authors of OSASIS still haven't decided on how to pronounce the model. We're going back and forth on 
    either making the first S silent (oʊˈeɪsɪs, like wonderwall) or not (əʊˈsasɪs, like oh-SAH-sis).
</div>


To this end, we've proposed One-Shot Structure-Aware Stylized Image Synthesis (OSASIS), a diffusion model capable of disentangling the structure
and semantics of an image, and **stylizing real images with just a single reference style.**
OSASIS leverages the CLIP space, and we train the model by making sure the CLIP space between four images stay parallel,
respective to its counterpart. We also formulate a Structure Preserving Network (SPN) that ensures the stylized output
retains the structure of the original input image.

<img-comparison-slider class="coloured-slider">
  {% include figure.liquid path="assets/img/portfolio/osasis/osasis_left.jpg" class="img-fluid rounded z-depth-1" slot="first" %}
  {% include figure.liquid path="assets/img/portfolio/osasis/osasis_right.jpg" class="img-fluid rounded z-depth-1" slot="second" %}
</img-comparison-slider>
<div class="caption">
    Qualitative/Quantitative comparisons of OSASIS against baseline models.
</div>

Designed to be a **training dataset-free one-shot model** (only requiring a single reference image), OSASIS is robust in preserving
the structure of rarely-seen attributes of a dataset!


[Paper link](https://arxiv.org/abs/2402.17275) [Code coming soon!]

Tools used:
[![](https://img.shields.io/badge/PyTorch-white?style=flat-square&logo=Pytorch)]()
[![](https://img.shields.io/badge/Docker-white?style=flat-square&logo=Docker)]()
[![](https://img.shields.io/badge/Bash-white?style=flat-square&logo=GNU Bash)]()
[![](https://img.shields.io/badge/CUDA-white?style=flat-square&logo=NVIDIA)]()
[![](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=OpenCV)]()
[![](https://img.shields.io/badge/Git-white?style=flat-square&logo=Git)]()
[![](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=NumPy)]()

<br>

---

## **Projects**

<br>

### **1. AI Spark Challenge - IRDIS (Immediate Rescue Based Disaster Response System)**

**Ministry of Science and ICT lead AI Tournament, Final Selection - 2022.04**

<br>

For the hackathon, I lead team **IRDIS** (Immediate Rescue Based Disaster Response System), where we were tasked to develp a disaster response solution.

developed and trained a satellite-image segmentation model, developed a disaster response solution ranking areas in need of immediate rescue via segmentation maps and the Analytical Hierarchy Process

We decided to utilize the [xView2 Dataset](https://xview2.org/dataset), containing 22K satellite images. Each image has a pre/post disaster counterpart, and labels containing information about
what disaster had occured, the level of damage, and segmentation labels for roads and buildings.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/irdis/2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We first trained two seperate UNet models on the xView2 Dataset on the task of semantic segmentation, each detecting the segmentation maps for
buildings and roads. Then, we trained a third model conditioned on the type of building and segmentation map to predict the damage level for a building.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/irdis/3.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We then created a Scoring/Ranking system based off the Analytical Hierarchy Process (AHP), which ranked which building was in need of immediate rescue.
The criterions of the AHP took in the type of building, how many buildings were in proximity, how badly damaged the buildings were, the time of day, and what kind of road
lead up to said building.

[Code link](https://github.com/tomtom1103/AISpark_Challenge_IRDIS)

Tools used:
[![](https://img.shields.io/badge/PyTorch-white?style=flat-square&logo=Pytorch)]()
[![](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=Pandas)]()
[![](https://img.shields.io/badge/GeoPandas-139C5A?style=flat-square&logo=GeoPandas)]()
[![](https://img.shields.io/badge/Bash-white?style=flat-square&logo=GNU Bash)]()
[![](https://img.shields.io/badge/CUDA-white?style=flat-square&logo=NVIDIA)]()
[![](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=OpenCV)]()
[![](https://img.shields.io/badge/Git-white?style=flat-square&logo=Git)]()
[![](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=NumPy)]()

<br>

### **2. KUIAI Hackathon - Multi-polygon map based business location recommendation system**

**Korea Electronics Technology Institute (KETI) lead AI Hackathon, 3rd place/20 teams - 2022.01**

<br>

For the hackathon, I lead team **Journey Lee**, where we were tasked to deduct meaningful data collected from commercial stores/buildings
across Seoul, South Korea in 3 days. We decided to build a location recommendation system for the commercial success of businesses.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/journeylee/1.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We trained a simple model with 8 FC layers, tasked to predict the quarterly sales of a business based on multiple variables,
including, but not limited to: geographic coordinates, type of business, sales per day of the week, sales per age group.
We then created an online tool that would take in the type of business and geographic coordinate, and output the predicted monthly sale of said location
and any other viable address within a 500 meter radius.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/journeylee/2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

On the online tool, we've also included a GeoPandas based EDA tool, where data about the various variables used to train the model are visualized.

[Code link](https://github.com/tomtom1103/KUIAI_Hackathon_2022/tree/main), [Web-App link](https://tomtom1103-kuiai-hackathon-2022-jl-app-6kr5kv.streamlit.app/)

Tools used:
[![](https://img.shields.io/badge/PyTorch-white?style=flat-square&logo=Pytorch)]()
[![](https://img.shields.io/badge/Streamlit-white?style=flat-square&logo=Streamlit)]()
[![](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=Pandas)]()
[![](https://img.shields.io/badge/GeoPandas-139C5A?style=flat-square&logo=GeoPandas)]()
[![](https://img.shields.io/badge/Bash-white?style=flat-square&logo=GNU Bash)]()
[![](https://img.shields.io/badge/CUDA-white?style=flat-square&logo=NVIDIA)]()
[![](https://img.shields.io/badge/Git-white?style=flat-square&logo=Git)]()
[![](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=NumPy)]()

<br>

### **3. K-Data Datacampus - Audio-Image Korean phoneme recognition model**

**Korea Data Agency lead AI Tournament, 2nd place/40 teams - 2021.09**

For the tournament, I lead team **Gillajab-i**, where we first developed and trained a transformer based ASR Korean phoneme recognition model,
and serviced a web-app for Korean word pronunciation error, measured by a Levenshtein distance based character-error rate (CER) metric.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/gillajabi/1.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We collected 1000 hours of raw Korean speaking data, re-tokenized the ground truth speech to phonemes, and trained a ViT.
The model took in audio, converted it to Mel-Spectrograms, and predicted the phonemes (instead of words) from said audio clip,
reaching a test accuracy of 93%.


Targeted at foreigners learning to speak korean, the web-app would follow these steps:
1. A user would choose or upload a short video of someone saying the phrase they would want to learn
2. Audio is extracted from the video
3. Model converts the audio into Korean text
4. Converts the Korean text into Romaji (for the user to read)
5. Converts the Korean text into English (for the user to understand)


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/gillajabi/2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The user would record themselves practicing the pronunciation, and our ViT would inference the phonemes from the speech,
and calculate the error rate via Levenshtein distance.

[Code link](https://github.com/tomtom1103/DataYouthCampus-Team4)

Tools used:
[![](https://img.shields.io/badge/PyTorch-white?style=flat-square&logo=Pytorch)]()
[![](https://img.shields.io/badge/Streamlit-white?style=flat-square&logo=Streamlit)]()
[![](https://img.shields.io/badge/Bash-white?style=flat-square&logo=GNU Bash)]()
[![](https://img.shields.io/badge/CUDA-white?style=flat-square&logo=NVIDIA)]()
[![](https://img.shields.io/badge/Git-white?style=flat-square&logo=Git)]()
[![](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=NumPy)]()

<br>

### **4. Smart Campus Datathon - Unsupervised embedding model based scholarship parsing/recommendation system**

**Korea University Datahub lead AI Datathon, 2nd place/15 teams - 2021.09**

<br>

For the datathon, I lead team **Peachtree**, where we we developed a web-app with an embedded unsupervised embedding model
that parsed and recommended scholarships based off a user's age, major, GPA, home address, previous scholarship grants, and scholarship eligibility.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/peachtree/engine.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/portfolio/peachtree/fe.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


We first crawled data from major scholarship hosting platforms targeted to undergraduate students, and preprocessed the unstructured data.
We then used Doc2Vec as an embedding model to embed all the scholarships, and used DBSCAN to cluster said scholarships.
For the actual recommendation of scholarships, 'anchor' students were matched with a cluster based on their previous scholarship grants,
and the cosine similarity between an inference student and all the 'anchor' students were calculated to place the inference student in a cluster.
A final rule based algorithm filters out scholarships that aren't eligible, and students were recommended scholarships that were left in the cluster.


[Code link](https://github.com/tomtom1103/2021Datathon_Peachtree)

Tools used:
[![](https://img.shields.io/badge/sklearn-white?style=flat-square&logo=scikitlearn)]()
[![](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=NumPy)]()
[![](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=Pandas)]()
[![](https://img.shields.io/badge/Django-092E20?style=flat-square&logo=Django)]()
[![](https://img.shields.io/badge/MariaDB-003545?style=flat-square&logo=MariaDB)]()
[![](https://img.shields.io/badge/Bash-white?style=flat-square&logo=GNU Bash)]()
[![](https://img.shields.io/badge/Git-white?style=flat-square&logo=Git)]()

<br>

