---
title: "Simple Multi-Layer Perceptron Neural Network"
summary: "A very simple, non-optimised setup of MLP in raw c++ (no external libraries), training using MNIST dataset"
date: 2025-09-25
tags: ["Artifical Intelligence", "Machine Learning", "Neural Networks", "Multi-Layer Perceptron", "C++"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

This blog assumes that the reader has the following pre-requisites:
- Knows C++
- Knows the following concepts/Watched the following videos:
1. (What are neural networks?)[https://www.youtube.com/watch?v=aircAruvnKk]
2. (Gradient Descent)[https://www.youtube.com/watch?v=IHZwWFHWa-w]
3. (Backpropagation Intuitively)[https://www.youtube.com/watch?v=Ilg3gGewQ5U]
4. (Backpropagation Calculus) [https://www.youtube.com/watch?v=tIeHLnjs5U8]

# Setting up
We will first need our dataset: For this project, I used the MNIST dataset which can be found (here)[https://www.kaggle.com/datasets/hojjatk/mnist-dataset?resource=download&select=train-labels.idx1-ubyte].

We will have to download the 4 files with our c++ setup.
![](../../images/AI/data_folder.png)

Next we will read the data:
1. Create a file called ExtractMNISTData.hpp (note: I'm using hpp, you can use .h if you prefer)
I won't go into the specifications of how to unload the data, you can follow (this video)[https://www.youtube.com/watch?v=gbGPBE26Fkc] or refer to (this blog)[https://medium.com/theconsole/do-you-really-know-how-mnist-is-stored-600d69455937]

```c++{linenos=true}
#ifndef EXTRACT_NMIST_DATA
#define EXTRACT_NMIST_DATA

#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <stdint.h>

int32_t ConvertBigEndianToSmallEndian(int32_t big_endian){
    return (
        (big_endian >> 24 & 0x000000FF) | 
        (big_endian >> 8 & 0x0000FF00) | 
        (big_endian << 8 & 0x00FF0000) |
        (big_endian << 24 & 0xFF000000)
        );
}

// list of images of 784 pixels 
std::vector<std::vector<unsigned char>> readImages(const std::string& filename, int32_t& num_image, int32_t& num_rows, int32_t& num_cols)
{
    std::ifstream file(filename, std::ios::in | std::ios::binary);
    if (!file.is_open())
    {
        throw("Unable to open file " + filename);
    }

    int32_t magic_number;
    file.read((char*)&magic_number, 4);
    magic_number = ConvertBigEndianToSmallEndian(magic_number);
    if (magic_number != 2051) {
        throw("MAGIC NUMBER INVALID FOR IMAGE READING");
    }

    file.read((char*)&num_image, 4);
    file.read((char*)&num_rows, 4);
    file.read((char*)&num_cols, 4);

    num_image = ConvertBigEndianToSmallEndian(num_image);
    num_rows = ConvertBigEndianToSmallEndian(num_rows);
    num_cols = ConvertBigEndianToSmallEndian(num_cols);

    size_t image_size = num_rows * num_cols;

    std::vector<std::vector<unsigned char>> ret;
    ret.reserve(num_image);

    for (int i = 0; i < num_image; ++i) {
        std::vector<unsigned char> image(image_size);
        file.read((char*)image.data(), image_size);
        ret.emplace_back(image);
    }

    return ret;
}

std::vector<unsigned char> readLabels(const std::string& filename, int32_t& num_labels) 
{
    std::ifstream file(filename, std::ios::in | std::ios::binary);
    if (!file.is_open())
    {
        throw("Unable to open file " + filename);
    }

    int32_t magic_number;
    file.read((char*)&magic_number, 4);
    magic_number = ConvertBigEndianToSmallEndian(magic_number);
    if (magic_number != 2049) {
        throw("MAGIC NUMBER INVALID FOR IMAGE READING");
    }

    file.read((char*)&num_labels, 4);
    num_labels = ConvertBigEndianToSmallEndian(num_labels);

    std::vector<unsigned char> ret(num_labels);
    file.read((char*)ret.data(), num_labels);

    return ret;
}


#endif //EXTRACT_NMIST_DATA
```

Here is a python file that I wrote as well to display the image using the MNIST dataset values, you'll just have to replace the np array with the image data.

```python{linenos=true}
import numpy as np
import matplotlib.pyplot as plt

data = np.array([[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,3,18,18,18,126,136,175,26,166,255,247,127,0,0,0,0],
[0,0,0,0,0,0,0,0,30,36,94,154,170,253,253,253,253,253,225,172,253,242,195,64,0,0,0,0],
[0,0,0,0,0,0,0,49,238,253,253,253,253,253,253,253,253,251,93,82,82,56,39,0,0,0,0,0],
[0,0,0,0,0,0,0,18,219,253,253,253,253,253,198,182,247,241,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,80,156,107,253,253,205,11,0,43,154,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,14,1,154,253,90,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,139,253,190,2,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,11,190,253,70,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,35,241,225,160,108,1,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,81,240,253,253,119,25,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,45,186,253,253,150,27,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,16,93,252,253,187,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,249,253,249,64,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,46,130,183,253,253,207,2,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,39,148,229,253,253,253,250,182,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,24,114,221,253,253,253,253,201,78,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,23,66,213,253,253,253,253,198,81,2,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,18,171,219,253,253,253,253,195,80,9,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,55,172,226,253,253,253,253,244,133,11,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,136,253,253,253,212,135,132,16,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]])

plt.imshow(data, cmap='gray', vmin=0, vmax=255)
plt.colorbar(label='Pixel Intensity') # Optional: Add a colorbar to show value mapping
plt.title('Grayscale Image from Array Values')
plt.axis('off') # Optional: Hide axes
plt.show()
```

# Raw C++ Linear Algebra
I will be writing my own vector and matrix classes to do calculations, You will notice that I am using VectorF and Matrix class. If you intend to use any third party linear algebra libraries eg. eigen, glm, they are similar and the underlying concept won't change.

Note: I did not write the full functionalities of the vector and matrix class, just those required.

# Layers
First, I'll talk about the layers. In my example, my layers will hold the logits values (pre-activated values), the activated values, the derived values ,the delta (I will talk about the delta later), the weights to the next layer, and the bias of the next layer.

```c++{linenos=true}
class Layer {
public:
    //...

    // Calculate the logit value for the next layer
    void FeedForward(Layer* next_layer);

    //Calculate output layer delta
    void CalculateOutputDelta(VectorF& expectedOutput);
    //Calculate hidden layer delta
    void CalculateDelta(Layer* nextLayer);
    //Update weight and biases during backpropagation
    void UpdateWeightsAndBias(double learning_rate, VectorF& next_layer_delta);

    //...
private:
    VectorF logit_nodes;
    VectorF derived_nodes;
    VectorF activated_nodes;
    VectorF delta;

    ///weights is a m rows x n columns matrix, where m is the number of columns in the next layer, while n is the number of columns in this layer.
    ///Think of it as row i in matrix * all nodes of this layer + bias = next column node j logit values
    Matrix weights;
    VectorF bias;
};
```

# Feed forward
Feed forward in this case just means the process of calculating the next layers of neurons from the inputs, weights and biases. You can see this like the values being calculated layer by layer.

Feed forward calculation:

Lets say we have L1, L2, L3, where L1 is the input, L2 is the hidden and L3 is the output layer.

L2 = L1 * weights in L1 + bias in L1 
L3 = activation(L2) * weights in L2 + bias in L2

As mentioned in the layer section, the weights and bias in L1 is represented by the weights from L1 to L2 and the bias of L2.

This is a very simple operation due to matrix multiplication, but if you are doing this using your own linear algebra classes, ensure that the matrix multiplication results in the correct size outputs.
For example: Layer 1 has 784 nodes, Layer 2 has 64 nodes, the matrix multiplication can only happen you do matrix of size 64x784 * vector of size 784 + vector of size 784. The convention might change depending on the matrix class implementation and you might have to transpose the matrix.


# Backpropagation
If you have watched the backpropagation calculus video on 3Blue1Brown, the calculations are as follows:

Let X, Y and Z be the activated value of a neuron in the 3 different layers, L1, L2, L3. let X', Y', Z' be the logit value of the nueron. Let the activation be σ as I'm using sigmoid as my activation in this example. Let the loss function be Loss(x) and the derivative of the loss function be Loss'(x)

Using gradient descent, calculus, chain rule.

Backpropagating L3->L2

Change in weight of ZY 
= del(Loss) / del(Weight of ZY) 
= [del(Loss) / del(Z)] * [del(Z) / del(Z')] * [del(Z') / del(Weight of ZY)]
= Loss'(Z) * σ'(Z') * Y

Change in bias for Z (the bias vector in L2)
= del(Loss) / del(Bias of Z)
= [del(Loss) / del(Z)] * [del(Z) / del(Z')] * [del(Z') / del(Bias of Z)]
= Loss'(Z) * σ'(Z') * 1

Change in Y in respect of Loss (we will need this later)
= del(Loss) / del(Y)
= [del(Loss) / del(Z)] * [del(Z) / del (Z')] * [del(Z') / del(Y)]
= Loss'(Z) * σ'(Z') * weight of ZY

You would notice that Loss'(Z) * σ'(Z') is a common denominator, this is actually also the value of delta, which is stored so that we can reuse.

Let's continue with the calulations for Backpropagating L2->L1:
Change in weight of YX
= del(Loss) / del(Weight of YX)
= [del(Loss) / del(Y)] * [del(Y) / del(Y')] * [del(Y') / del(Weight of YX)]
= [del(Loss) / del(Y)] * σ'(Y') * X

Change in bias for Y (the bias vector in L1)
= del(Loss) / del(Bias of Y)
= [del(Loss) / del(Y)] * [del(Y) / del(Y')] * [del(Y') / del(Bias of Y)]
= [del(Loss) / del(Y)] * σ'(Y') * 1

Hmm, notice these formula looks familiar?

The derivative of loss is replaced with del(Loss) / del(Y) for all hidden layers, noticed how the first calculation I stored the value of [del(Loss) / del(Y)].

Hence the calculation for output delta will look something like this:
```c++{linenos=true}
    // derivative of loss function (x - x^), using MSE loss function
    // calculate d/dz( 1/n * (z - z1)^2 )
    VectorF loss = (this->activated_nodes - expectedOutput) * (2.0 / (double)m_num_nodes);
    // delta = d/dz ( f'(z) ) * d/dz (loss)
    this->delta = GetDerivedNodes().PiecewiseProduct(loss);
```
Note: In the example, I am using Mean Loss Squared as the loss function, which the derivative will be (Z - Ẑ), where Ẑ is the expected value of the output layer

The calculation of the hidden layer delta will look like this:
```c++{linenos=true}
    VectorF loss = weights.Transpose() * nextLayer->delta;
    this->delta = GetDerivedNodes().PiecewiseProduct(loss);
```

Updating the weights and bias will then look like this:
```c++{linenos=true}
    VectorF adjustment_delta = next_layer_delta * learning_rate;
    weights -= Matrix::VectorXVectorTranspose(adjustment_delta, this->activated_nodes);
    bias -= adjustment_delta;
```
Note: There is a learning rate value, this is to either increase or decrease the rate of the gradient descent, the learning rate is a hyper-parameter,or basically a value that can be controlled outside the neural network.

If you reached this point, thanks for reading. Hope you enjoyed. My repo can be found on my github when I'm satisfied with optimising the code. You can send me an email of DM me to find out more.