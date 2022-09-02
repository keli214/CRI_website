---
layout              : page
title               : "CRI_Simulations Documentation"
meta_title          : "CRI_Simulations Documentation"
header              : no
sidebar: right
layout              : page-fullwidth
permalink           : "/cri/"
---

<div class="row">
<div class="medium-4 medium-push-8 columns" markdown="1">
<div class="panel radius" markdown="1">
**Table of Contents**
{: #toc }
*  TOC
{:toc}
</div>
</div><!-- /.medium-4.columns -->



<div class="medium-8 medium-pull-4 columns" markdown="1">
  
{% include alert text='Note! This project is under active development' %}
  
  
CRI_Simulations is a Python library for interfacing with the INC CRI Neuromorphic comuputing system. 

Check out the [Usage]({{ site.url }}{{ site.baseurl }}/cri/#usage) section for further information, including how to [install]({{ site.url }}{{ site.baseurl }}/cri/#installation) the project.
## Usage
  
### Installation
  
  To use CRI_Simulation, first clone the repository from git:
  
  {% include alert terminal='git clone git@github.com:Integrated-Systems-Neuroengineering/CRI_Simulations.git' %}
  
  Next create a python environment with the necessary dependencies using conda:
  
  {% include alert terminal='cd CRI_Simulations.git
  conda env create -f condaenv.yaml
  conda activate CRI_Simulations' %}
  
  Finally, make sure to copy the adxdma_dump binary into the top directory of the CRI_Simulations repository.
  
### Basic Usage
  
  A few steps are required in order to run a baisc network.

  First configure the parameters in config.yaml and FPGA_Execution/config.yaml.

  Second specify your input and connections in inputs.txt and connections.txt .

  Next within `justin_test.py` edit the second argument of the initialization of fpga_compiler to match the number of neurons in the network and edit the arguments to `write_parameters()` to match with the desired neuron model, neuron threshold, number of outputs (number of neurons) in the network, and number of inputs (number of axons) in the network. Finally edit `num_time_steps` in `justin_test.py` to reflect the desired number of timesteps you wish to run the network for.

  Once the proper parameters are set in the two yaml files and justin_test.py a full execution of the network can be run by `python justin_test.py`. This will read in the connections in [connections.txt]({{ site.url }}{{ site.baseurl }}/cri/#inputs), correctly program HBM on the FPGA, and run the network stepwise providing the appropriate input specified in [inputs.txt]({{ site.url }}{{ site.baseurl }}/cri/#inputs) before each timestep. After each timestep of execution the membrane potentials for all neurons in the network will be printed to terminal.
  
## Intermediate Representation Format {Intermediate-Representation}
  
### Connections
  
### Inputs
  
## Architectural Details {Architectural-Details}
  
### PCI-e Command Specifications
  
### HBM Specifications
  
## API {API}
  
### compile_network module
  
### FPGA_Execution.fpga_compiler module
  
### FPGA_Execution.fpga_controller module
  
## YAML Specifications {YAML-Specifications}
  
### config.yaml
  
### FPGA_Execution/config.yaml
  
</div><!-- /.medium-8.columns -->
</div><!-- /.row -->

bottom text


