---
#
# Use the widgets beneath and the content will be
# inserted automagically in the webpage. To make
# this work, you have to use › layout: frontpage
#
layout: frontpage
header: false

widget1:
  title: "Cinnamoroll"
  url: 'https://stang126.github.io/CRI/'
  image: tumblr_f40443bf9aa38d7eaf4a6940521fe55d_e3aeaff6_640.jpg
  text: '...'
widget2:
  title: "Gert"
  url: 'https://stang126.github.io/CRI/'
  image: gertcauwenberghs.png
  text: '...'
widget3:
  title: "Kuromi"
  url: 'https://stang126.github.io/CRI/'
  image: download.jpg
  text: '...'

#
# Use the call for action to show a button on the frontpage
#
# To make internal links, just use a permalink like this
# url: /getting-started/
#
# To style the button in different colors, use no value
# to use the main color or success, alert or secondary.
# To change colors see sass/_01_settings_colors.scss
#
permalink: /index.html
#
# This is a nasty hack to make the navigation highlight
# this page as active in the topbar navigation
#
homepage: true
---

## **Introduction**
---
L2S is a Python library for interacting with the ISN CRI project hosted at SDSC. This project aims to make massive scale simulations of spiking neural networks easily accessible to the research community, and in particular researches interested in neuromorphic computing for artificial intelligence and neuroscience researchers. This library allows a user to define a spiking neural network and execute it on one of two backends: the CRI neuromorphic hardware or if the hardware is not available a python simulation of the hardware.

## **Installation**
---
### **Simple Installation**

{% include alert terminal='pip install l2s' %}
    
### **Development Installation**

- First install [Poetry](https://python-poetry.org/)
  - If Poetry installs in may be necessary to install an alternative Python distribution such as [Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
- Then clone this repository: 

{% include alert terminal='git clone https://github.com/Integrated-Systems-Neuroengineering/L2S.git' %}

- Next you will need to clone the cri-simulations repository into the same directory you just cloned this repository into:

{% include alert terminal='git clone https://github.com/Integrated-Systems-Neuroengineering/CRI_Simulations_Public.git' %}

- cd into the L2S repo you cloned and install the needed dependencies. Resolving dependencies may take a while.

{% include alert terminal='cd L2S <br> poetry install' %}

   - Some Python dependencies may require a compiler supporting C++11 to be installed on your system, such as a recent version of GCC

- finally activate the development environment

{% include alert terminal='poetry shell' %}
    
## **Usage**
---
### **Running on the Simulator**

On your local machine you can run networks using the Python based simulator of the CRI hardware.

### **Defining a Network**
Users are expected to provide three data structures in order to define a network

#### **Defining the Configuration Dictionary**

The configuration dictionary specifies a few properties that are shared by every neuron in the network

- neuron_type specifies the type of neuron model used to calculate membrane potentials
- global_neuron_params is a sub-dictionary of the configuration dictionary
  - v_thr is an entry in the global_neuron_params dictionary, it sets the membrane potential threshold for all neurons in the network

// {% include alert terminal="configuration = {}  <br> configuration &#91;'neuron_type'&#93; = <q>I&F</q>  <br> configuration &#91; 'global_neuron_params' &#93; = {}  <br> configuration &#91; 'global_neuron_params'&#93; &#91;'v_thr' &#93; = 4" %}

