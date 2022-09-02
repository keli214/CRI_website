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
  
## Intermediate Representation Format
  
### Connections
  
  specify the connection representation
  
### Inputs
  
  specify the connection representation
  
## Architectural Details
  
### PCI-e Command Specifications
  
 **Write Network Parameters**
  
  | Bits  | 0:16 | 17:33 |  34:69 |  70:71 | 72:503 |  504:511 |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Number of Inputs  |  Number of Outputs  |  Threshold  |  Neuron Model  |  Zeros  |  Command: 0x04  |
  
  **Send Input**
  
  | Bits  | 0:510 | 504 |  404:511 |  512:1023 |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| Content  | Zeros  |  1  |  Zeros  |  One Hot Encoding of Active Axons  |
  
**Request Read From HBM**
  
  | Bits  | 0:255 | 256:278 |  279  |  280:503 |  504:511 |
  | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
  |  Content  | Zeros  |  Row Address  |  0  |  Zeros  |  0x02  |
  
  **Flush Read From HBM**
  
  |  Bits  |  0:8  |
  | --- | --- |
  |  Content  |  0x04  |

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
  
  | **Axon Pointers**  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Axon 0 Pointer  | Axon 1 Pointer  |  Axon 2 Pointer  |  Axon 3 Pointer  |  Axon 4 Pointer  |  Axon 5 Pointer  |  Axon 6 Pointer  |  Axon 7 Pointer  |
  | Axon 8 Pointer  | Axon 9 Pointer  |  Axon 10 Pointer  |  Axon 11 Pointer  |  Axon 12 Pointer  |  Axon 13 Pointer  |  Axon 14 Pointer  |  Axon 15 Pointer  |
  | Axon 16 Pointer  | Axon 17 Pointer  |  Axon 18 Pointer  |  Axon 19 Pointer  |  Axon 20 Pointer  |  Axon 21 Pointer  |  Axon 22 Pointer  |  Axon 23 Pointer  |
  | Axon 24 Pointer  | Axon 25 Pointer  |  Axon 26 Pointer  |  Axon 27 Pointer  |  Axon 28 Pointer  |  Axon 29 Pointer  |  Axon 30 Pointer  |  Axon 31 Pointer  |
  | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ |
  | **Neuron Pointers**  |  |  |  |  |  |  |  |
  | Neuron 0 Pointer  | Neuron 1 Pointer  |  Neuron 2 Pointer  |  Neuron 3 Pointer  |  Neuron 4 Pointer  |  Neuron 5 Pointer  |  Neuron 6 Pointer  |  Neuron 7 Pointer  |
  | Neuron 8 Pointer  | Neuron 9 Pointer  |  Neuron 10 Pointer  |  Neuron 11 Pointer  |  Neuron 12 Pointer  |  Neuron 13 Pointer  |  Neuron 14 Pointer  |  Neuron 15 Pointer  |
  | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ | ⋮ |
  | **Synapses**  |  |  |  |  |  |  |  |
  | Synapse 15  | Synapse 14  |  Synapse 13  |  Synapse 12  |  Synapse 11  |  Synapse 10  |  Synapse 9  |  Synapse 8  |
  | Synapse 7  | Synapse 6  |  Synapse 5  |  Synapse 4  |  Synapse 3  |  Synapse 2  |  Synapse 1  |  Synapse 0  |
  | Synapse 15  | Synapse 14  |  Synapse 13  |  Synapse 12  |  Synapse 11  |  Synapse 10  |  Synapse 9  |  Synapse 8  |
  | Synapse 7  | Synapse 6  |  Synapse 5  |  Synapse 4  |  Synapse 3  |  Synapse 2  |  Synapse 1  |  Synapse 0  |
  
  Within the overall HBM layout axon pointers, neuron pointers, and synapses are represented as 32 bits of data arranged as follows:
  
  **Axon and Neuron Pointers**
  
  | Bits  | 0:22 | 23:31 |
| ------------- | ------------- | ------------- |
| Content  | Pointer Address  | Pointer Length  |

  **Synapse Format**
  
  | Bits  | 0:15 | 16:28 |  29:31 |
| ------------- | ------------- | ------------- | ------------- |
| Content  | Weight | Address  | Opcode  |
  
  **Spike Format**
  
  | Bits  | 0:15 | 16:28 |  29:31 |
| ------------- | ------------- | ------------- | ------------- |
| Content  | Spike Data  |  Address  | Opcode  |
  
  
## API
  
### compile_network module
  
  {% include alert text='compile_network.compileNetwork()' %}
  
  Creates simulation and FPGA data structures:
  
  Creates a representation of the axon pointers, neuron pointers, and synapse weights in HBM memory both in the format used to produce the commands to program the actual FPGA and in the format expected by the hardware simulator.
  
  `inputdict`
  Dictionary specifying inputs to the network. Key, Time Step Value, TODO are these axons or neurons
  
  `hbmdict`
  Dictionary specifying the hbm structure for each core expected by the hardware simulator. Key: core number Value: tuple of (pointer,data) where pointer is a numpy array and data is a list of lists of tuples.
  
  `outputsdict`
  TODO: I’m not sure what the outputs are for
  
  `axonLengthint`
  number of axons specified in the network
  
  {% include alert text='compile_network.external_input_optimization()' %}
  
  {% include alert text="compile_network.load_network(<em>input='test_inputs.txt', connex='test_connectivity.txt', output='out.txt'</em>)" %}
  
  Loads the network specification.
  
  This function loads the inputs and connections specified for the network. Also determines the number of FPGA cores to be used.
  
  `Parameters`
  * **input** *(str, optional)* – Path to file specifying network inputs. (the default is the path in config.yaml)
  * **connex** *(str, optional)* – Path to file specifying network connections. (the default is the path in config.yaml)
  
  `Returns`
  * **axons** *(dict)* – Dictionary specifying axons in the network. Key: axon number Value: Synapse Weights
  * **connections** *(dict)* – Dictionary specifying neurons in the network. Key: Neuron Number Value: Synapse Weights
  * **inputs** *(dict)* – Dictionary specifying inputs to the network. Key, Time Step Value, axon
  * **outputs** *(dict)* – TODO: I’m not sure what the outputs are for. I belive it’s unused
  * **ncores** *(int)* – The number of cores peresent in the CRI system
  
  {% include alert text='compile_network.main()' %}
  
  {% include alert text='compile_network.map_to_hbm(<em>axons, network, input, assignment, n_cores</em>)' %}
  
  {% include alert text='compile_network.map_to_hbm_fpga(<em>axons, network, input, assignment, n_cores, to_fpga=True</em>)' %}
  Creates HBM Data Structure
  
  Creates a representation of the axon pointers, neuron pointers, and synapse weights in HBM memory
  
  `Parameters`
  * **axons** *(dict)* – Dictionary specifying axons in the network. Key: axon number Value: Synapse Weights
  *  **network** *(dict)* – Dictionary specifying neurons in the network. Key: Neuron Number Value: Synapse Weights
  *  **inputs** *(dict)* – Dictionary specifying inputs to the network. Key: Time Step Value: TODO are these axons or neurons
  *  **assignment** *(dict)* – Dictionary specifying neurons mapped to each core. Key: core number Value: tuple of (neuron number, core number)
  *  **n_cores** *(int)* – The number of cores peresent in the CRI system
  *  **to_fpga** *(bool, optional)* – This parameter is depracated and has no effect. (the default is True)
  
  `Returns`
  
  **hbm** – Dictionary specifying the structure of data in memory for each core. Key: core number Value: tuple of (pointer,data) where pointer is a numpy array of tuples representing offsets into hbm memory and data is a list of lists tuples representing synapses.
  
  `Return type` dict
  
  {% include alert text='compile_network.partition(<em>network, n_cores</em>)' %}
  Creates adjacency list
  
  Uses the partitioning algorithm to partition the neurons in the network and return core assignments
  
  `Parameters`
  *  **network** *(dict)* – Dictionary specifying neurons in the network. Key: Neuron Number Value: Synapse Weights
  *  **n_cores** *(int)* – The number of cores peresent in the CRI system
  
  `Returns`
  
  Dictionary specifying neurons mapped to each core. Key: core number Value: tuple of (neuron number, core number)
  
  `Return type` dict
  
### FPGA_Execution.fpga_compiler module
  
  {% include alert text='<em>class</em> FPGA_Execution.fpga_compiler.fpga_compiler(<em>data, N_neurons</em>)' %}
  
  Bases: `object`
  
  Produces the needed adxdma dump scripts of a given network to program HBM
  
  `input`
  The inputs to the network for each timestep. Key, timestep value, list of axons <br/>
  Type: dict
  
  `axon_ptrs`
  Array of tuples pointing to the rows containing the synapses for the corresponding Axon.<br/>
  Type: dict
  
  `Each tuple is (start row, end row).`
  
  `neuron_ptrs`
  Array of tuples pointing to the rows containing the synapses for the corresponding Axon. <br/>
  Type: numpy array
  
  `Each tuple is (start row, end row).`
  
  `synapses`
  List of tuples corresponding to synapses. Each tuple is (oncore/offcore bit, synapse address (row index of destination neuron pointer in HBM calculated as floor(destination neuron index / number of neuron groups)), weight) <br/>
  Type: list
  
  `HBM_WRITE_CMD`<br/>
  Type: str
  
  `HBM_OP_RW
  OP code to read/write to hbm vie PCie<br/>
  Type: str
  
  `NRN_BASE_ADDR`
  Starting address of neuron pointers in HBM <br/>
  Type: int
  
  `SYN_BASE_ADDR`
  Starting address of synapses in HBM <br/>
  Type: int
  
  `PTR_ADDR_BITS`
  Number of bits used to represent pointer starting address </br>
  Type: int
  
  `PTR_LEN_BITS`
  Number of bits used to represent the number of rows of synapses a pointer corresponds to 
  
  `SYN_OP_BITS`
  Number of bits used to represent synapse opcode
  
  `SYN_ADDR_BITS`
  Number of bits used to represent synapse address
  
  `SYN_WEIGHT_BITS`
  Number of bits used to represent synapse weight
  
  `AXN_BASE_ADDR= _0_`
  
  `HBM_OP_RW= '<em>0200000000000000000000000000000000000000000000000000000000</em>'`
  
  `HBM_WRITE_CMD= '*sudo ./adxdma_dmadump wb 0 0*'`
  
  `NRN_BASE_ADDR= *16384*`
  
  `PTR_ADDR_BITS= *23*`
  
  `PTR_LEN_BITS= *9*`
  
  `SYN_ADDR_BITS= *13*`
  
  `SYN_BASE_ADDR= *32768*`
  
  `SYN_OP_BITS= *3*`
  
  `SYN_WEIGHT_BITS= *16*`
  
  `WRITE= *'01000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'*`
  
  `create_axon_ptrs()`
  
  Creates the necessary adxdma_dump commands to program the axon pointers into HBM
  
  Returns <br/>
  * *script* – The bash commands to run to program the axon pointers in HBM
  
  Return type: str
  
  `create_input_script(*num_timesteps, n_inputs, filename*)`
  
  `create_neuron_ptrs()`
  
  Creates the necessary data arguments to pass to the adxdma_dump commands to program the neuron pointers into HBM. Data arguments for multiple adxdma_dump commands are seperated by new line characters
  
  Returns  <br/>
  * **script** (*str*) – The data arguments to provide to a series of adxdma_dump commands. Data arguments for successive adxdma_dump commands
  * *are seperated by newline characters*
  
  `create_script(*fname*)`
  
  Generates the bash file to program HBM for the current network
  
  Generates the bash file needed to program the axon pointers, neuron pointers, and synapses into hbm
  
  Parameters <br/>
  * **fname** (*int*) – The filename to write the script to
  
  `create_synapses()`
  
  Creates the necessary adxdma_dump commands to program the synapses into HBM
  
  Returns <br/> 
  * **script** – The bash commands to run to program the synapses in HBM
  
  Return type: str
  
  `gen_input(*time_step*)`
  
  `gen_input2(*time_step*)`
  
  Generates the input command for a given time step
  
  Generates the necesary bash command to run to provide inputs to the network for a given timestep
  
  Parameters <br/>
  * **time_step** (*int*) – The timestep you wish to generate the input command for
  
  Returns <br/>
  * **command** – The bash command to run to send the input to the FPGA
  
  Return type: str
  
  `txt2script(*cmd_str*)`
  
  Converts a string of hex characters into the correct format to suppyl to the adxdma dump command.
  
  Given a string of hex characters with the left most character containing the MSB create a string of pairs of hex characters representing bytes with the leftmost byte contanining the LSB in the format expected by the adxdma_dmadump binary for the data argument.
  
  Parameters <br/>
  * **cmd_str** (*str*) – The string of hexidecimal characters to format. The first character represents the hex character containing the MSB
  
  Returns <br/>
  * script_txt – The formated string of bytes
  
  Return type: str
  
  {% include alert text='FPGA_Execution.fpga_compiler.main()' %}
  
  {% include alert text='FPGA_Execution.fpga_compiler.text2script(<em>self, cmd_str</em>)' %}
  
  Converts a string of hex characters into the correct format to suppyl to the adxdma dump command.
  
  Given a string of hex characters with the left most character containing the MSB create a string of pairs of hex characters representing bytes with the leftmost byte contanining the LSB in the format expected by the adxdma_dmadump binary for the data argument.
  
  Parameters <br/>
  * **cmd_str** (*str*) – The string of hexidecimal characters to format. The first character represents the hex character containing the MSB
  
  Returns <br/>
  * **script_txt** – The formated string of bytes
  
  Return Type: str
  
  
### FPGA_Execution.fpga_controller module
  
## YAML Specifications
  
### config.yaml
  
### FPGA_Execution/config.yaml
  
</div><!-- /.medium-8.columns -->
</div><!-- /.row -->

bottom text


