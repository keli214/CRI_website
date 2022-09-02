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

```
pip install l2s
```
### **Development Installation**

- First install [Poetry](https://python-poetry.org/)
  - If Poetry installs in may be necessary to install an alternative Python distribution such as [Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
- Then clone this repository
   ```
   git clone https://github.com/Integrated-Systems-Neuroengineering/L2S.git
   ```
- Next you will need to clone the cri-simulations repository into the same directory you just cloned this repository into
    ```
    git clone https://github.com/Integrated-Systems-Neuroengineering/CRI_Simulations_Public.git
    ```
- cd into the L2S repo you cloned and install the needed dependencies. Resolving dependencies may take a while.
    ```
    cd L2S
    poetry install
    ```
   - Some Python dependencies may require a compiler supporting C++11 to be installed on your system, such as a recent version of GCC

- finally activate the development environment
    ```
    poetry shell
    ```
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
  ```
  configuration = {}
  configuration['neuron_type'] = "I&F"
  configuration['global_neuron_params'] = {}
  configuration['global_neuron_params']['v_thr'] = 4
  ```
- Defining the Axons Dictionary

    The axons dictionary configures inputs to the network. Axons are synapses connected to neurons in the network that the user can manually send spikes over at a given timestep. Each key in the dictionary is the name of an axon. Each value is a list of two element tuples. Each tuple defines an in-going synapse to a neuron. The first element is the name of a neuron in the network and the second element is the weight of the synaptic connection. Synapse weights must be integers, but they may be positive or negative.
    ```python
    axons = {'alpha': [('a', 3)],
             'beta': [('d', 3)]}
    ```

- Defining the Connections Dictionary
    
    The connections dictionary defines the neurons in the network and the connections between them. Each key in the dictionary is the name of a neuron. Of note the names of neurons in the connections dictionary and the names of axons in the axons dictionary must be mutually exclusive. Each value is a list of two element tuples. Each tuple defines a synapse between neurons in the network. The first element is the name of the postsynaptic neuron and the the second element is the weight of the synapse. Synapse weights must be integers but they may be positive or negative. If a neuron has no outgoing synapses it’s synapse list may be left empty.
    ```python
    connections = {'a': [('b', 1)],
                   'b': [],
                   'c': [],
                   'd': [('c', 1)]}
    ```
- Defining the Outputs List
  The outputs list defines the neurons in the network the user wishes to receive spikes from. Each element in the list is the key of a neuron in the connections dictionary.
  ```python
  outputs = ['a', 'b']
  ```
#### **Initializing a network**
Once we’ve defined the above dictionaries and list we must pass them to the CRI_network constructor to create a CRI_network object.

```python
network = CRI_network(axons=axons,connections=connections,config=config, outputs=outputs)
```
#### **Running a Timestep**
Once we’ve constructed an CRI_network object we can run a timestep. We do so by calling the step() method of CRI_network. This method expects a single input called inputs. Inputs defines the inputs to the network at the current timestep, in particular it is a list of names of axons that you wish to carry spikes into the network at the current timestep. Normally network.step() returns a list of the keys that correspond to neurons that spiked during the given timestep, however the membranePotential parameter can be set to True to additionally output the membranePotentials for all neurons in the network.
```python
inputs = ['alpha','beta']
spikes = network.step(inputs)
#Alternative
potentials, spikes = network.step(inputs, membranePotential=True)
```

This method will return a list of membrane potentials for all neurons in the network after the current timestep has elapsed.

#### **Updating Synapse Weights**
Once the CRI_network class the topology of the network is fixed, that is what axon and neurons are in the network and how they are connected via synapses may not be changed. However it is possible to update the weight of preexisting synapses in the network. This can be done by calling the write_synapse() method of CRI_network. write_synapse() takes three arguments, the presynaptic neuron name, the postsynaptic neuron name, and the new synapse weight.
```python
network.write_synapse('a', 'b', 2)
```


## **Submitting Jobs to Run on the Hardware**
The same Python scripts you’ve developed and run on your local machine can be deployed to the CRI servers to run on the actual CRI hardware. Just make sure all the libraries you import in your script are available on the [CRI servers](https://github.com/Integrated-Systems-Neuroengineering/L2S/blob/main/Python%20libraries%20present%20on%20the%20CRI%20servers). The CRI hardware is hosted in the San Diego Supercomputing Center and jobs may be submitted to run on the hardware via the [Neuroscience Gateway](https://www.nsgportal.org/index.html). First you must register an account with Neuroscience Gateway in order to submit jobs. Perform the following steps to submit a task to NSG:
- Put your CRI Python script in a folder of any name, then zip the folder
- Log into NSG.
- Create a task folder if there is none listed on the upper left.  It's a place to hold related jobs.
- Click on data, and save the previously created zip file as the data.  Here 'data' is ambiguous - it is the job and its data.
- Click on task.
- Create a new task if needed (or clone an old one).
- Assign the zip you just uploaded as data as the input to the task.
- Select *Python for CRI* as the software to run.
- Set parameters for the task:
    - Set execution 'wall time', cores, and GB of DRAM if you wish. Please be consideret to others and only request the hardware you need.
    - Enter the name of your.py python scrip as the "input" using the same name as is in the zip folder.
    - Enter a name for the "output" (optional)
- Click save parameters
-  Click *save and run* to run the task.
- Click *OK* on the popup or the job will not start.
- Click on task again in your folder at the upper left if the task list is not present.
- View status if desired, refresh as needed, or just watch for the task done email.
- When it is done select the 'view output' for that task on the task list.
- Download outputs and decompress.  Job 'inputs' is displayed as garbage.

* Python libraries present on the CRI servers
  
    | absl-py                |     1.1.0 |
    | bidict                 |    0.22.0 |
    | brotlipy               |     0.7.0 |
    | certifi                | 2021.10.8 |
    | cffi                   |    1.15.0 |
    | charset-normalizer     |     2.0.4 |
    | click                  |     8.1.3 |
    | colorama               |     0.4.4 |
    | conda                  |    4.12.0 |
    | conda-content-trust    | 0+unknown |
    | conda-package-handling |     1.8.1 |
    | confuse                |     1.7.0 |
    | cri-simulations        |     0.1.2 |
    | cryptography           |    36.0.0 |
    | cycler                 |    0.11.0 |
    | fbpca                  |       1.0 |
    | fonttools              |    4.33.3 |
    | idna                   |       3.3 |
    | joblib                 |     1.1.0 |
    | k-means-constrained    |     0.7.1 |
    | kiwisolver             |     1.4.3 |
    | l2s                    |     0.1.3 |
    | llvmlite               |    0.38.1 |
    | matplotlib             |     3.5.2 |
    | metis                  |     0.2a5 |
    | networkx               |     2.8.4 |
    | numba                  |    0.55.2 |
    | numpy                  |    1.22.4 |
    | ortools                | 9.3.10497 |
    | packaging              |      21.3 |
    | Pillow                 |     9.1.1 |
    | pip                    |    21.2.4 |
    | protobuf               |    4.21.1 |
    | pycosat                |     0.6.3 |
    | pycparser              |      2.21 |
    | PyMetis                |    2020.1 |
    | pyOpenSSL              |    22.0.0 |
    | pyparsing              |     3.0.9 |
    | PySocks                |     1.7.1 |
    | python-dateutil        |     2.8.2 |
    | PyYAML                 |       6.0 |
    | requests               |    2.27.1 |
    | ruamel-yaml-conda      |  0.15.100 |
    | scikit-learn           |     1.1.1 |
    | scipy                  |     1.8.1 |
    | setuptools             |    61.2.0 |
    | six                    |    1.16.0 |
    | sklearn                |       0.0 |
    | threadpoolctl          |     3.1.0 |
    | tqdm                   |    4.63.0 |
    | urllib3                |    1.26.8 |
    | wheel                  |    0.37.1 |