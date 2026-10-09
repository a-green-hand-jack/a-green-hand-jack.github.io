---
layout: post
title: "Backpropagation"
date: 2024-06-09 00:00:00 +0800
description: "A tutorial on backpropagation covering entropy, cross-entropy loss, and gradient-based optimization."
tags: [deep-learning, machine-learning]
categories: [tutorial, deep-learning]
giscus_comments: true
---

# Preface

"Viewed from the side, a mountain range; viewed from the end, a single peak. Far, near, high, low — each has a different view. I cannot tell the true face of Mount Lu, for I myself am in the mountain."

In the previous post, we discussed solving linear equations, a topic we have studied since elementary school with problems like the "chicken and rabbit in the same cage." Back then, our focus was on "obtaining the solution to the system of equations" — that is, how to deduce the number of chickens and rabbits from the counts of heads and feet:

$$
\begin{bmatrix}
a_{11} & a_{12} \\
a_{21} & a_{22}
\end{bmatrix}
\begin{bmatrix}
x_{1} \\
x_{2}
\end{bmatrix}
=\begin{bmatrix}
b_{1} \\
b_{2}
\end{bmatrix}
$$

The matrix form of the above system is:

$$
\mathbf{A}\mathbf{x} = \mathbf{b}
$$

At that time, given $\mathbf{A}$ and $\mathbf{b}$, we needed to find $\mathbf{x}$. However, when we study linear algebra, we realize that the properties of $\mathbf{A}$ are what truly matter. In other words, the process of mapping one linear space to another is the fundamental question, rather than the properties of individual points in the original or mapped spaces. Compared to the previous section, this section requires a similar shift in perspective.

# Focusing on Parameters

In the previous section, we described the simplest neural network — the multi-layer perceptron (MLP) — using the language of tensors: $\mathbf{\vec{y^0}} \in \mathbb{R}^m = \mathbb{f}(\mathbf{\vec{x^0}} \in \mathbb{R}^n) = sigmoid^i(fc^i(\mathbf{\vec{x^0}} ))=sigmoid^i(\mathbf{W^i}\cdot(\mathbf{\vec{x^0}}) + \mathbf{b^i})$. As previously mentioned, the sigmoid function here is $\sigma(x) = \frac{1}{1 + \mathrm{e}^{-x}}$, and everything about it is fixed. Therefore, the above MLP can be expressed in another form: $\mathbf{\vec{y^0}} \in \mathbb{R}^m = \mathbb{g}(\mathbf{\vec{x^0}}, \mathbf{W, b}), \mathbf{\vec{x^0}} \in \mathbb{R}^n$. Here, $W$ and $b$ are parameters to be learned, collectively denoted as $\theta$.

Thus, the most concise form of the above expression is $\hat{Y} = F(X, \theta), \hat{Y} \in R^{(s,m)}, X \in R^{(s, n)}$. Here, $F$ represents an **implicit function** with no explicit closed-form expression; in the language of neural networks, it embodies human abstract reasoning, and the neural network's goal is to learn such a function. Here, $s$ is the number of data samples in the dataset, $n$ is the input dimension, and $m$ is the output dimension.

For a neural network, both its input and output — $X$ and $\hat{Y}$ — are determined; what it needs to do is learn an **appropriate** $\theta$. Therefore, the focus of this section is precisely this $\theta$!

# A Suitable Metric

We mentioned an **appropriate $\theta$** above, but how do we quantify "appropriateness"?

## Entropy and Information Entropy

"When the law is unpredictable, its authority is immeasurable."

The same holds for information: the more you know, the more ordered the information becomes, and its "power" lies in its predictability; the less you know, the more chaotic the information, becoming an unpredictable "punishment."

Although the concept of "entropy" was introduced by Western scientists, Eastern political thinkers seem to have recognized the power within information entropy long ago...

- Entropy is a concept from physics representing the degree of uncertainty, or disorder, in a system.
- Information entropy: Shannon introduced entropy into information theory to measure the uncertainty of information.
- The two share the same essence applied in different domains.

## Information Entropy

### Properties

- The less likely an event, the greater its information content; conversely, a certain event carries zero information.
- Greater information content implies greater information entropy.
- The information content of several independent events occurring simultaneously equals the sum of each event's information content.

### Definition

First, we define **information content** using the logarithmic function, where $P(x)$ is the probability of the event:

$$
I(x) = -log(P(x))
$$

Let us verify whether $I(x)$ satisfies the properties above:

- When an event is certain, $P(x)=1$, so $I(x)=0$ — no information.
- When an event is impossible, $P(x)=0$, so $I(x)=+\infty$ — infinite information.
- For $n$ **independent events**:
  - Their probabilities are $P(x_i)$, with respective information contents $I(x_i)=-log(P(x_i))$.
  - The joint probability is $\prod_{i=0}^nP(x_i)$.
  - The joint information content is $I(x)=-log(\prod_{i=0}^nP(x_i))=-(\Sigma_{i=0}^nP(x_i))=\Sigma_{i=0}^n I(x_i)$.

Thus, this definition of **information content** satisfies the requirements.

Next, we define **information entropy** as the **expected information content** produced by events following distribution $P$: $H(P)=E_{X\sim P}[I(x)]=-E_{X\sim P}[logP(x)]=-\Sigma_iP(x_i)log(P(x_i))$. Clearly, $H(P)$ and $I(x)$ satisfy property 2.

In summary, we have obtained the definition of information entropy and recognize that it can serve as a metric for **distributions**.

## KL Divergence

A neural network can be viewed as a mapping — albeit a complex one 🧠 — that transforms an **input space** into a **prediction space**. Corresponding to this input space, we also have a **label space**, i.e., the **ground-truth space**.

Obviously, we want our **prediction space** to be as similar as possible to the **ground-truth space**. Is there any way to measure whether these two spaces are "similar"? The answer is clear: from the above discussion on **entropy**, we realize that a space is essentially defined by $X\sim P(X)$, so we can indeed describe a space's distribution using probability distributions. Since **entropy** describes the distribution of a single space, it is natural to use a similar concept to describe the **difference between distributions** of two spaces.

Relative entropy, or KL divergence, measures the degree of approximation between two probability distributions. Like entropy, KL divergence takes a small value when the two distributions are very close.

Below is the definition of KL divergence. Suppose we have two probability distributions: the true distribution $P$ and the approximate distribution $Q$:

$$
\begin{cases}
KL(P||Q)=\Sigma_i P(x_i)log(\frac{P(x_i)}{Q(x_i)}) —— \small{离散状态} \\
KL(P||Q)=\int p(x)\frac{p(x)}{q(x)}——连续状态
\end{cases}
$$

It can also be written in expectation form: $KL(P||Q)=\mathbb{E}[log(p(x))-log(q(x))]$

From this definition, when the true and approximate distributions are identical, the KL divergence is zero. Its expectation form can be interpreted as the **expected difference of the logarithms between the true and approximate distributions**.

## Cross-Entropy

Clearly, KL divergence is already a good metric for measuring the difference between two distributions, but why not simplify it further if possible?

$$
\begin{align*}
KL(P||Q) &= \sum_i P(x_i) \log\left(\frac{P(x_i)}{Q(x_i)}\right) \\
&= \sum_i \left(P(x_i) \log(P(x_i)) - P(x_i) \log(Q(x_i))\right) \\
&= -H(P) - \sum_i P(x_i) \log(Q(x_i))
\end{align*}
$$

Here, we find that $-H(P)$, determined by the **true distribution**, is a constant in deep learning since it is fixed by the provided dataset. The latter term is what actually determines the KL divergence. Therefore, we can compute only the latter term, $-\Sigma_iP(x_i)log(Q(x_i))$, which we call **cross-entropy**.

Here is the definition of cross-entropy: for two distributions $P$ and $Q$, where $P$ is the true distribution and $Q$ is the predicted/approximate distribution:

$$
H(P,Q)=-E_{X\sim P}[logQ(x)]=-\Sigma_iP(x_i)log(Q(x_i))
$$

Thus, we establish the relationship between information entropy ($H(P)$), relative entropy / KL divergence ($KL(P||Q)$), and cross-entropy ($H(P,Q)$):

$$
KL(P||Q) = -H(P) + H(P,Q)
$$

When the true distribution $P$ remains fixed, minimizing KL divergence is equivalent to minimizing cross-entropy, and cross-entropy is commonly used as the loss function in machine learning!

## Cross-Entropy and Model Parameters

Alright, we discussed some mathematical concepts above and introduced the important notion of **cross-entropy**, but what does cross-entropy have to do with our model?

$$
H(P,Q)=-E_{X\sim P}[logQ(x)]=-\Sigma_iP(x_i)log(Q(x_i))
$$

Here, $Q$ is the approximate distribution. As mentioned earlier, the most concise form of a neural network is: $\hat{Y} = F(X, \theta), \hat{Y} \in R^{(s,m)}, X \in R^{(s, n)}$. Thus, the connection becomes clear: $\hat Y$ and $Q$ are equivalent, both representing the distribution predicted by the model (the neural network). Therefore, $H(P,Q)=H(P,\hat{Y})=\mathbb{H}(P, \theta, X)$, where $X$ is entirely determined by the input dataset itself, just like $P$. This can be further simplified to $H(P,Q)=\mathfrak{H}(\theta)$.

In other words, cross-entropy depends solely on the model parameters!!

For simplicity, we will write $\mathfrak{H}(\theta)$ as $loss(\theta)$, i.e., the loss function. In deep learning, different tasks may use different forms of **loss functions**, but cross-entropy is the most common one, so we use it here as an example.

# Optimizing the Loss Function

## The Objective of Optimization

After the derivations above, we have obtained a metric, $loss$ — the loss function. This metric depends only on the model parameters $\theta$, yet effectively reflects the discrepancy between the prediction space and the ground-truth space. There is a clear relationship: the smaller the $loss$, the smaller the discrepancy between the two spaces. Thus, our original question becomes concrete: **find a $\theta^*$ that makes $loss(\theta)$ as small as possible**.

I am not quite sure why it is called a "loss function" rather than "loss entropy." Perhaps it is because this "entropy" behaves like a function as $\theta$ changes, hence the name "loss function."

Note that although we hope for a unique $\theta^*$ that achieves this, it is not entirely clear whether such a unique solution exists. However, this does not significantly affect our operations; we only need to find a good $\hat{\theta}$ under a fixed input distribution (corresponding to a specific dataset)!

As mentioned in the previous section, a model has many parameters, and they are randomly initialized to $\theta_{init}$. Therefore, obtaining our desired $\hat{\theta}$ is no easy task. Fortunately, **gradient optimization** can help us achieve this.

Although there are methods for parameter initialization, we will not discuss them here.

## Gradient Optimization

In short, we gradually obtain the desired $\hat \theta$ through gradient optimization:

$$
\theta_t = \theta_{t-1} - \mu \cdot \nabla loss(\theta_{t-1})
$$

This should be easy to understand. In multivariable calculus, it is proven that the gradient of a function points in the direction of steepest descent. Therefore, optimizing $\theta$ according to the gradient $\nabla loss(\theta)$ of the loss function is a natural choice. Here, $\mu$ is the learning rate, i.e., the step size of each parameter update.

To intuitively illustrate how gradient descent moves from the initialized $\theta_{init}$ to the desired $\hat \theta$, let us first consider an "ideal model" with only two parameters, $\theta_1$ and $\theta_2$. The corresponding $loss$ is a binary function, as shown below:
*
Clearly, the $loss$ here is entirely determined by the parameters $\theta_1$ and $\theta_2$.

Several different gradient optimization methods are shown below, but we can ignore their specific operations for now; it suffices to know that they all depend on $\nabla loss(\theta)$.
![梯度优化](%E6%A2%AF%E5%BA%A6-%E5%8F%82%E6%95%B0%E4%BC%98%E5%8C%96.gif)

## Chain Rule

![链式法则](cover.png)

In the previous section, we explained that gradient optimization is the method for parameter optimization, and an essential operation in gradient optimization is computing the gradient of the function, i.e., $\nabla loss(\theta)$.

However, this computation is not trivial. In a multi-layer perceptron (MLP), the loss function is typically computed at the output layer, and then the error is propagated backward layer by layer through the backpropagation algorithm. Suppose we have a three-layer MLP. The loss function $ L $ is usually the difference between the predicted value $\hat{y}$ at the output layer and the true value $y$. A common form is the mean squared error (MSE):
$ L = \frac{1}{2}(\hat{y} - y)^2 $

To demonstrate backpropagation and the chain rule, we need to compute the partial derivatives of the loss function $ L $ with respect to each weight $w$ and bias $b$. Below are the general forms of the partial derivatives of the loss with respect to the first-layer weight $w^{(1)}$ and bias $b^{(1)}$:

$ \frac{\partial L}{\partial w^{(1)}} = \frac{\partial L}{\partial z^{(2)}} \cdot \frac{\partial z^{(2)}}{\partial a^{(2)}} \cdot \frac{\partial a^{(2)}}{\partial z^{(1)}} \cdot \frac{\partial z^{(1)}}{\partial w^{(1)}} $
$ \frac{\partial L}{\partial b^{(1)}} = \frac{\partial L}{\partial z^{(2)}} \cdot \frac{\partial z^{(2)}}{\partial a^{(2)}} \cdot \frac{\partial a^{(2)}}{\partial z^{(1)}} \cdot \frac{\partial z^{(1)}}{\partial b^{(1)}} $

where:

- $ z^{(1)} = w^{(1)} \cdot x + b^{(1)} $ is the weighted input plus bias of the first layer.
- $ a^{(1)} = \sigma(z^{(1)}) $ is the activation output of the first layer.
- $ z^{(2)} = w^{(2)} \cdot a^{(1)} + b^{(2)} $ is the input to the second layer.
- $ a^{(2)} = \sigma(z^{(2)}) $ is the activation output of the second layer.
- $ \hat{y} = w^{(3)} \cdot a^{(2)} + b^{(3)} $ is the final prediction.
- $ \sigma $ is the activation function, such as sigmoid or ReLU.

PyTorch is currently the most popular deep learning framework. In PyTorch, the gradient $\nabla loss(\theta^i_t)$ of a parameter at a given time step is computed through this backpropagation method and stored in the form $(\theta_t^i, \nabla loss(\theta^i_t))$. This increases memory overhead but reduces computation time.

# Summary

In this section, we analyzed how to obtain "appropriate parameters" for a model. We first quantified the "appropriateness" of model parameters using cross-entropy, then used gradient optimization to continuously adjust $\theta_{init}$ so that it gradually approaches the target $\hat \theta$. To achieve such gradient optimization, we need to efficiently compute $\nabla loss(\theta^i_t)$, a task guaranteed by the chain rule.

We have not yet discussed the specific algorithms for gradient optimization, nor how to adjust the learning rate. These topics would take considerable time, so we will set them aside for now.

In summary, through these two sections, we have roughly clarified the operational ideas behind forward and backward passes in a neural network. The remaining details will be filled in gradually — stay tuned! 🥳
