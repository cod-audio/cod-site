---
title: Continuous Object Detector
description: Cooper Barth, Hugo Flores Garcia, Jack Wiig
---

## About
Continuous Object Detector (C.O.D.) is a tool designed to help audio engineers with visual impairments to more quickly get to locations within tracks that they are trying to find.

## Interface


## Model
Big picture, our model consists of three main parts: a mel spectrogram, a pre-trained, convolutional audio embedding model, and a neural network classifier. 

### Input Representation

We use a 128-bin Mel Spectrogram of 1 second of audio as our input. Here are some hyperparameters:

- 48kHz audio goes through a log-mel spectrogram. 
    - This means that the highest frequency we can represent is 24kHz (one half of the sample rate). This is well above the range of human hearing (around 18-20kHz)

- Mel Spectrogram hyperparameters:
    - Number of Mel frequency bins: 128
    - Magnitude representation (not power)
    - FFT window size: 2048 samples (42.67ms)
    - FFT hop size: 242 samples (5ms)


### Embedding 
We use the audio subnetwork of the [L3-net](https://github.com/marl/openl3) architecture for our embedding model. The weights are initialied to the mel128, music model variant. 

*Note*: depending on the kernel size of the last maxpool layer, you have two different embedding sizes to choose from: 

- With a maxpool kernel of (16, 24), your output embedding is size 512
- With a maxpool kernel of (4, 8), your output embedding is size 6144

We found during preliminary testing that using an embedding size of 6144 worked best in our case. 

### Classifier
We use fully connected layers with ReLU activations for our classifier. Because PyTorch code is super readable, here's a code snippet with out model architecture. 

```
class MLP6144(pl.LightningModule):

    def __init__(self, dropout, num_output_units):
        super().__init__()

        self.fc = nn.Sequential(
            nn.BatchNorm1d(6144),
            nn.Linear(6144, 512), 
            nn.ReLU(), 
            nn.Dropout(dropout), 

            nn.BatchNorm1d(512),
            nn.Linear(512, 128), 
            nn.ReLU(), 
            nn.Dropout(dropout), 

            nn.BatchNorm1d(128),
            nn.Linear(128, num_output_units))

    def forward(self, x):
        return self.fc(x)
```

where `num_output_units` refers to the number of classes in out dataset.

We use batch normalization, dropout, and ReLU activations, as well as a Softmax in the output (not pictured above because it's included in our loss function).


## Backend
