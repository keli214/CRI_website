---
#
# Use the widgets beneath and the content will be
# inserted automagically in the webpage. To make
# this work, you have to use › layout: frontpage
#
layout: frontpage

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

## Introduction
---

The ISN CRI project is hosted at SDDC and aims to make massive scale simulations of spiking neural networks easily accessible to the research community, in particular researches interested in neuromorphic computing for artificial intelligence and neuroscience researchers. 

## Install
---

### Simple Installation
{% include alert terminal='pip install l2s' %}

### Development Installation

- First install [Poetry](https://python-poetry.org/)
  - If Poetry installs in may be necessary to install an alternative Python distribution such as [Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
- Then clone this repository: 

{% include alert terminal='git clone https://github.com/Integrated-Systems-Neuroengineering/L2S.git' %}

- Next you will need to clone the cri-simulations repository into the same directory you just cloned the L2S repository into:

{% include alert terminal='git clone https://github.com/Integrated-Systems-Neuroengineering/CRI_Simulations_Public.git' %}

- cd into the L2S repo you cloned and install the needed dependencies. Resolving dependencies may take a while.

{% include alert terminal='cd L2S <br> poetry install' %}

   - Some Python dependencies may require a compiler supporting C++11 to be installed on your system, such as a recent version of GCC

- finally activate the development environment

{% include alert terminal='poetry shell' %}

## Quick Start
---

getting started

### Using NeuroSci Gateway
