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
  
  {% include alert terminal='cd CRI_Simulations.git <br> conda env create -f condaenv.yaml <br>   conda activate CRI_Simulations' %}
  
  Finally, make sure to copy the adxdma_dump binary into the top directory of the CRI_Simulations repository.
  
### Basic Usage
  
  A few steps are required in order to run a baisc network.

  First configure the parameters in [config.yaml]({{ site.url }}{{ site.baseurl }}/cri/#configyaml) and [FPGA_Execution/config.yaml]({{ site.url }}{{ site.baseurl }}/cri/#fpgaexecutionconfigyaml).

  Second specify your input and connections in inputs.txt and connections.txt .

  Next within `justin_test.py` edit the second argument of the initialization of fpga_compiler to match the number of neurons in the network and edit the arguments to `write_parameters()` to match with the desired neuron model, neuron threshold, number of outputs (number of neurons) in the network, and number of inputs (number of axons) in the network. Finally edit `num_time_steps` in `justin_test.py` to reflect the desired number of timesteps you wish to run the network for.

  Once the proper parameters are set in the two yaml files and justin_test.py a full execution of the network can be run by `python justin_test.py`. This will read in the connections in [connections.txt]({{ site.url }}{{ site.baseurl }}/cri/#inputs), correctly program HBM on the FPGA, and run the network stepwise providing the appropriate input specified in [inputs.txt]({{ site.url }}{{ site.baseurl }}/cri/#inputs) before each timestep. After each timestep of execution the membrane potentials for all neurons in the network will be printed to terminal.
  
## Intermediate Representation Format {Intermediate-Representation}
  
### Connections
  
  specify the connection representation
  
### Inputs
  
  specify the connection representation
  
## Architectural Details
  
### PCI-e Command Specifications
  
 ** Write Network Parameters**
  
  | Bits  | 0:16 | 17:33 |  34:69 |  70:71 | 72:503 |  504:511 |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Number of Inputs  |  Number of Outputs  |  Threshold  |  Neuron Model  |  Zeros  |  Command: 0x04  |
  
  **Send Input**
  
  | Bits  | 0:510 | 504 |  404:511 |  512:1023 |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Zeros  |  1  |  Zeros  |  One Hot Encoding of Active Axons  |
  
**  Request Read From HBM**
  
   | Bits  | 0:255 | 256:278 |  279 |  280:503 |  504:511 |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Zeros  |  Row Address  |  0  |  Zeros  |  0x02  |
  
  
  **Flush Read From HBM**
  
    | Bits  | 0:8 |
| ------------- | ------------- |
| Content  | 0x04  |

  Format of the read flushed from HBM is as follows: 
  
    | Bits  | 0:255 | 256:495 |  496:511 |
| ------------- | ------------- | ------------- | ------------- |
| Content  |  Data  |  Zeros  |  0xBBBB  |

  **Write HBM**
  
     | Bits  | 0:255 | 256:278 |  279 |  280:503 |  504:511 |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Data  |  Row Address  |  1  |  Zeros  |  0x02  |
  
  **Request Read from URAM**
  
       | Bits  | 0:35 | 36:48 |  49:52 |  53 |  504:511 |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Zeros  |  Neuron Row Address  |  Neuron Column Address  |  0  |  0x03  |
  
  **Flush Read From URAM**
  
      | Bits  | 0:8 |
| ------------- | ------------- |
| Content  | 0x04  |
  
  Format of the read flushed from URAM is as follows:
  
         | Bits  | 0:35 | 36:48 |  49:52 |  53:495 |  496:511 |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Membrane Potential  |  Neuron Row  |  Neuron Column  |  Zeros  |  0xCCCC  |
  
  **Execute Time Step**
  
        | Bits  | 0:503 | 504:511 |
| ------------- | ------------- | ------------- |
| Content  | Zeros  | 0x01  |
  
### HBM Specifications
  
  HBM is segmented into three sections, one section to hold axon pointers, a section for neuron pointers, and a section for synapses. The different sections in HBM start at different addresses in the HBM. They are as below:
  
  Axon Base Address: 0 Neuron Base Address: 2<sup>14</sup> Synapse Base Address: 2<sup>15</sup>
  
  HBM is segmented into rows holding eight axons/neurons/synapses each. Since we arrange axons/neurons/synapses into groups of 16 each group of 16 axons/neurons/synapses occupies two adjacent rows in HBM. Within those 16 neuron groups axon and neuron pointers are arranged from zero to 15 where as synapses are arranged from 15 to zero. Axon and neuron pointers contain a starting address that refers to a row in the synapse space of HBM and a length value that determines the number of rows in the synapse section that contain the synapses for that neuron. Within the rows pointed to by an axon/neuron pointer synapses are arranged based on the index of their destination neuron. That is within a two row 16 synapse group synapses are placed at an index based of of their destination neuron modulo 16. So for example if the axon zero pointer points to Rows 0 and 1 of the synapse section and axon 0 has a single synapse to neuron 18 the synapse would be stored in the synapse two slot of the first two rows of the synapse portion of HBM.
  
## API {API}
  
### compile_network module
  
### FPGA_Execution.fpga_compiler module
  
### FPGA_Execution.fpga_controller module
  
## YAML Specifications
  
### config.yaml
  
### FPGA_Execution/config.yaml
  
</div><!-- /.medium-8.columns -->
</div><!-- /.row -->

bottom text


