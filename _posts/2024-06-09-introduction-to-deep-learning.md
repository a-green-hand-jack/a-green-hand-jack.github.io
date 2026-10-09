---
layout: post
title: "Introduction to Deep Learning"
date: 2024-06-09 12:00:00 +0800
description: "An intuitive introduction to neural networks, using curve fitting as a motivating example to explain forward propagation."
tags: [Machine Learning, Deep Learning]
categories: [deep-learning, fundamentals]
giscus_comments: true
---

# Preface

## Problem Introduction

Imagine you are researching a new drug; let's call it "Neo-Metformin." During drug development, you want to study the relationship between drug dosage and its efficacy. For simplicity, you use the mass-to-body-weight ratio (g/kg) as the dosage, and blood glucose level (mmol/L) one hour after administration as the efficacy measure, considering 0.45-0.50 mmol/L as the normal range.

After conducting a series of experiments, you obtain the following figure:

Naturally, you would hope to use some analytical expression to describe this curve; and furthermore, you would hope to determine this curve with the fewest possible parameters. A simple idea is to use **piecewise linear fitting**, which would look something like this:

Obviously, although this method uses **fewer parameters**, it requires determining **kink points**, that is, switching between different linear functions, which does not seem to be an easy task.

Of course, we could also use the great **polynomial fitting**, but this method is not very good either. If you have done *fog experiments*, you know this method tends to overfit, meaning it doesn't fit well at certain points. If this problem occurs in low dimensions, we certainly cannot expect good results in high dimensions either.

But what if there were a method that could automatically help **bend linear functions**? Then using linear function fitting would be a great idea. Fortunately, this is not difficult!

The term "bend" here is just a visual description; a more appropriate statement is adjusting the contribution of different linear functions to the "sum" in different ranges.

## Fitting Curves

In fact, with just some simple linear and nonlinear functions, we can easily fit this curve, as shown below:

Here, the blue and red represent a linear change and a nonlinear change respectively. We achieve these changes through corresponding linear and nonlinear functions. Notably, the nonlinear function used here is a **sigmoid** function. I haven't labeled the parameters for each linear change because the specific parameters don't matter here; we just need to see from the changes in the y-axis that the corresponding linear changes are occurring.

It should be pointed out that the sigmoid function has the form: $\sigma(x) = \frac{1}{1 + \mathrm{e}^{-x}}$, meaning its parameters are fixed and known. The linear function $y = \mathrm{w} \cdot x + \mathrm{b}$ has unknown $\mathrm{w}$ and $\mathrm{b}$ that need to be determined.

For now, let's not consider how $\mathrm{w}$ and $\mathrm{b}$ are determined, but instead deeply examine what the above figure is actually doing and how to understand such a fitting process.

# Abstraction

This section prepares for the methods discussed later, mainly to introduce **tensors**.

## Neural Networks and Curve Fitting

As discussed above, fitting a curve can in fact be achieved through the superposition of some linear and nonlinear operations. And what happens in neural networks is exactly such operations, as shown below:

Here we emphasize what happens during **forward** propagation; **backward** propagation can wait ＞︿＜

This shows a simple neural network **SimpleMLP**, whose internal structure is in fact two linear layers, `fc1` and `fc2`, and two nonlinear layers, `sigmoid1` and `softmax`, working together. Input data passes through sequentially: `fc1` > `sigmoid1` > `fc2` > `softmax`, to get the final output.

This is the basic structure of a Multi-Layer Perceptron (**MLP**), and MLP is the most fundamental form of modern neural networks.

Here, let's first treat the `fc1` > `sigmoid1` > `fc2` > `softmax` process as "some transformation," that is, "a black box." We just input a series of data and get a series of outputs, and the outputs match the real-world results!

So by now we know that we can in fact view a neural network as the result of a series of linear and nonlinear functions acting successively. But always using such language to describe a neural network is inconvenient, especially when studying network optimization and complex networks. So below we hope to introduce a more abstract, universal method to describe what is happening.

## Neural Networks and Tensors

A scalar is a 0-dimensional tensor, a vector is a 1-dimensional tensor, and a matrix is a 2-dimensional tensor; this is pointed out to avoid ambiguity in what follows.

An important thing linear algebra teaches us is: **a linear transformation can be represented by a matrix operation**. We often do this by using input and output vectors to determine the intermediate transformation matrix. So if we ignore the nonlinear parts in neural networks first, we realize they are the same thing. That is, we can use operations between tensors to replace the operations in the linear parts of neural networks.

We use tensors because although we always see operations on vectors and matrices when learning linear algebra, real-world data is often high-dimensional, such as images, which are difficult to describe without tensors. And the rules of tensor operations are the same.

So how do we handle the nonlinear parts? It's actually simple: we use element-wise operations. So nonlinear operations are element-wise operations on tensors; specifically, the elements should be injective, meaning one input has only one unique output.

Here, we replace the original linear operations with dot products between tensors; and replace the original nonlinear operations with element-wise operations on tensors. This explains how to obtain the corresponding prediction $\mathbf{\vec{y^0}} \in \mathbb{R}^m$ from input $\mathbf{\vec{x^0}} \in \mathbb{R}^n$.

Notably, a neural network can have multiple inputs and multiple outputs, and each input or output can be high-dimensional information (which often means a very long vector).

# Summary

In summary, we have learned that neural networks are to some extent the same thing as curve fitting; and to better describe the process, and to meet the challenges of high dimensions and complex networks, we use the language of tensors to describe operations in neural networks. This way, we have described the **forward** propagation process of a neural network.

However, we have not yet explained how the unknown parameters of the linear operations in neural networks, that is, those tensors $\mathbf{W}$ and $\mathbf{b}$, are determined. Because they are obviously carefully chosen; otherwise, how could the predicted curve match the real curve? And these are the processes of backpropagation (backward).

Moreover, we should recognize that although the "introduction" input I gave here is a scalar, the actual input is a **tensor**. That is, it is high-dimensional information, such as a 512×512 RGB image, which has 262,144 pixels, and each pixel has 3 numbers controlling the corresponding three-channel values, meaning that to completely describe this image requires 786,432 parameters. And this is just one image; real-world images number in the hundreds of millions.

Considering this, and considering the requirements of linear equation solving, our network will also need a large number of parameters, those $\mathbf{W}$ and $\mathbf{b}$. Thus, how to efficiently optimize these parameters so that they can well capture the relationship between input and output is of critical importance.

And **backward** is the answer to this question!
