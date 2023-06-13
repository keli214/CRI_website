---
layout: page-fullwidth
title: "Mnist"
meta_title: "Mnist"
permalink: "/tutorials/mnist"
header : no
---

### **Feedforward Fully Connected SNN**

This tutorial goes over how to train a simple feedforward SNN and deploy on HiAER Spike using our 
conversion pipline.

### **Define a Feedforward SNN**
To build a simple feedforward spiking neural network with PyTorch, we can use snnTorch, SpikingJelly or other deep learning frameworks that are based on PyTorch. Currently, our conversion pipline supports snnTorch and SpikingJelly. In this tutorial, we will be using SpikingJelly.

Install the PyPi distribution of SpikingJelly

```python
pip install spikingjelly
```

Import necessary libraries from SpikingJelly and PyTorch

```python
from spikingjelly.activation_based import neuron, functional 
import torch 
import torch.nn as nn
```

Using SpikingJelly, we can define a simple 2-layer feedforward SNN model with 1000 hidden neurons. The PyTorch layer will act as synapses between the spiking neuron layers. 

```python
class model(nn.Module): <br>
    def __init__(self, features = 1000): <br>
        super().__init__() <br>
        self.linear1 = nn.Linear(28 * 28, features, bias=False) <br>
        self.lif1 = neuron.LIFNode() <br>
        self.linear2 = nn.Linear(features, 10, bias=False) <br>
        self.lif2 = neuron.LIFNode() <br>
    def forward(self, x): <br>
        x = self.linear1(x) <br>
        x = self.lif1(x) <br>
        x = self.linear2(x) <br>
        x = self.lif2(x) <br>
        return x
```

Initiate the Network
```python
net = model()
```

### **Setting up the MNIST Dataset**
```python
from torchvision import datasets, transforms

#Download MNIST data from torch 
mnist_train = datasets.MNIST('data/mnist', train=True, download=True, transform=transforms.Compose(
    [transforms.ToTensor()]))
mnist_test = datasets.MNIST('data/mnist', train=False, download=True, transform=transforms.Compose(
    [transforms.ToTensor()]))

# Create DataLoaders
train_loader = DataLoader(mnist_train, batch_size=128, shuffle=True, drop_last=True)
test_loader = DataLoader(mnist_test, batch_size=128, shuffle=True, drop_last=True)


```

### **Training the SNN**
Since we are using a static image dataset, we have to encode the image into spikes using the encoding function from spikingjelly. 

```python
#Setting up the encoder
encoder = encoding.PoissonEncoder
#Using Adam optimizer with a learning rate of 0.1
optimizer = torch.optim.Adam(net.parameters(), lr=0.1)

#Define training parameters
epochs = 10

for epoch in range(epochs):
        start_time = time.time()
        net.train()
        train_loss = 0
        train_acc = 0
        train_samples = 0
        for img, label in train_loader:
            optimizer.zero_grad()
            img = img.to(device)
            label = label.to(device)
            label_onehot = F.one_hot(label, 10).float()
            for t in range(args.num_steps):
                encoded_img = encoder(img)
                out_fr += net(encoded_img)
            out_fr = out_fr/args.num_steps  
            loss = loss_fun(out_fr, label_onehot)
            loss.backward()
            optimizer.step()

            train_samples += label.numel()
            train_loss += loss.item() * label.numel()
            train_acc += (out_fr.argmax(1) == label).float().sum().item()

            functional.reset_net(net)

        train_time = time.time()
        train_speed = train_samples / (train_time - start_time)
        train_loss /= train_samples
        train_acc /= train_samples
```

### **Converting the SNN to HiAER Spike Format**
```python
from converter import *
from l2s.api import CRI_network

#Fold the BN layer 
bn = BN_Folder() 
net_bn = bn.fold(net)

#Weight, Bias Quantization 
qn = Quantize_Network() 
net_quan = qn.quantize(net_bn)

#Convert to HiAER-Spike Dictionaries
num_steps = 4
input_layer = 0
output_layer = 11
input_size = (3, 32, 32)
backend = 'snnTorch'
threshold = qn.v_threshold

cn = CRI_Converter(num_steps = num_steps, 
                   input_layer = input_layer, 
                   output_layer = output_layer, 
                   input_shape = input_shape,
                   backend=backend,
                   v_threshold = v_threshold)
cn.layer_converter(net_quan)
```

### **Initiate the HiAER Spike SNN**
```python
config = {}
config['neuron_type'] = "I&F"
config['global_neuron_params'] = {}
config['global_neuron_params']['v_thr'] = int(quan_fun.v_threshold)
    

hardwareNetwork = CRI_network(dict(cri_convert.axon_dict),
                              connections=dict(cri_convert.neuron_dict),
                              config=config,target='CRI', 
                              outputs = cri_convert.output_neurons,
                              coreID=1)
softwareNetwork = CRI_network(dict(cri_convert.axon_dict),
                              connections=dict(cri_convert.neuron_dict),
                              config=config,target='simpleSim', 
                              outputs = cri_convert.output_neurons,
                              coreID=1)
```

### **Deploying the SNN on HiAER Spike**
```python
cri_convert.bias_start_idx = int(cri_convert.output_neurons[0])
loss_fun = nn.MSELoss()
start_time = time.time()
test_loss = 0
test_acc = 0
test_samples = 0
num_batches = 0
for img, label in tqdm(test_loader):
    cri_input = cri_convert.input_converter(img)
    output = torch.tensor(cri_convert.run_CRI_hw(cri_input,hardwareNetwork), dtype=float)
    loss = loss_fun(output, label)
    test_samples += label.numel()
    test_loss += loss.item() * label.numel()
    test_acc += (output == label).float().sum().item()
    num_batches += 1
test_time = time.time()
test_speed = test_samples / (test_time - start_time)
test_loss /= test_samples
test_acc /= test_samples

print(f'test_loss ={test_loss: .4f}, test_acc ={test_acc: .4f}')
print(f'test speed ={test_speed: .4f} images/s')
```