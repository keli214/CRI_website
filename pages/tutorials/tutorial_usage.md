---
layout: page-fullwidth
title: "Usage"
meta_title: "Usage"
permalink: "/examples/usage"
header : no
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

{% include alert terminal="configuration = {}  <br> configuration&#91;neuron_type'&#93; = <q>I&F</q>  <br> configuration&#91;'global_neuron_params'&#93; = {}  <br> configuration&#91;'global_neuron_params'&#93;&#91;'v_thr'&#93; = 4" %}

- Defining the Axons Dictionary

    The axons dictionary configures inputs to the network. Axons are synapses connected to neurons in the network that the user can manually send spikes over at a given timestep. Each key in the dictionary is the name of an axon. Each value is a list of two element tuples. Each tuple defines an in-going synapse to a neuron. The first element is the name of a neuron in the network and the second element is the weight of the synaptic connection. Synapse weights must be integers, but they may be positive or negative.

{% include alert terminal="axons = {'alpha': &#91;('a', 3)&#93;, <br> 'beta': &#91;('d', 3)&#93;}" %}

- Defining the Connections Dictionary
    
    The connections dictionary defines the neurons in the network and the connections between them. Each key in the dictionary is the name of a neuron. Of note the names of neurons in the connections dictionary and the names of axons in the axons dictionary must be mutually exclusive. Each value is a list of two element tuples. Each tuple defines a synapse between neurons in the network. The first element is the name of the postsynaptic neuron and the the second element is the weight of the synapse. Synapse weights must be integers but they may be positive or negative. If a neuron has no outgoing synapses it’s synapse list may be left empty.

{% include alert terminal="connections = {'a':  &#91;('b', 1)&#93;, <br> 'b':  &#91;&#93;, <br> 'c':  &#91;&#93;, <br> 'd':  &#91;('c', 1)&#93;}" %}

- Defining the Outputs List
  The outputs list defines the neurons in the network the user wishes to receive spikes from. Each element in the list is the key of a neuron in the connections dictionary.

{% include alert terminal="outputs = &#91;'a', 'b'&#93;" %}

#### **Defining Stochastic Behavior**
HiAER-Spike supports randomly perturbing the membrane potential of neurons at each timestep of execution. To enable this perturbation the perturb variable is set to True. The amplitude of this perturbation can also be scaled by setting the perturbMag variable (default = 0). perturbMag takes integer values between 0 and 18 and multiplies the random noise to be added to the membrane potential by 2<sup>perturbMag</sup>.

{% include alert terminal="perturb = True <br> perturbMag = 2" %}

#### **Initializing a network**
Once we’ve defined the above dictionaries and list we must pass them to the CRI_network constructor to create a CRI_network object.

{% include alert terminal='network = CRI_network(axons=axons,connections=connections,config=config, outputs=outputs, perturb=Perturb, perturbMag = perturbMag)' %}

#### **Running a Timestep**
Once we’ve constructed an CRI_network object we can run a timestep. We do so by calling the step() method of CRI_network. This method expects a single input called inputs. Inputs defines the inputs to the network at the current timestep, in particular it is a list of names of axons that you wish to carry spikes into the network at the current timestep. Normally network.step() returns a list of the keys that correspond to neurons that spiked during the given timestep, however the membranePotential parameter can be set to True to additionally output the membranePotentials for all neurons in the network.

{% include alert terminal="inputs = &#91;'alpha','beta'&#93; <br> spikes = network.step(inputs) <br> #Alternative <br> potentials, spikes = network.step(inputs, membranePotential=True)" %}

This method will return a list of membrane potentials for all neurons in the network after the current timestep has elapsed.

#### **Updating Synapse Weights**
Once the CRI_network class the topology of the network is fixed, that is what axon and neurons are in the network and how they are connected via synapses may not be changed. However it is possible to update the weight of preexisting synapses in the network. This can be done by calling the write_synapse() method of CRI_network. write_synapse() takes three arguments, the presynaptic neuron name, the postsynaptic neuron name, and the new synapse weight.

{% include alert terminal="network.write_synapse('a', 'b', 2)" %}
