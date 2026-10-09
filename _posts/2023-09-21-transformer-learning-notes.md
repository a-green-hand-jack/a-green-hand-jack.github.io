---
layout: post
title: "Transformer Learning Notes"
date: 2023-09-21 12:00:00 +0800
description: "A beginner-friendly walkthrough of the Transformer architecture, covering self-attention, multi-head attention, encoder-decoder structure, and positional encoding."
tags: [Machine Learning, Transformer]
categories: [deep-learning, fundamentals]
giscus_comments: true
---

## Preface

[Attention is All You Need](http://arxiv.org/abs/1706.03762) is the seminal work that introduced the Transformer. The TensorFlow code related to the paper can be obtained from [GitHub](https://github.com/tensorflow/tensor2tensor), and of course many people have made [secondary developments](https://github.com/huggingface/transformers) of it.

In this article, I try to simplify the model a bit and introduce the core concepts one by one, hoping to make it easy for beginners to understand.

## Machine Translation

![Machine Translation](%E6%9C%BA%E5%99%A8%E7%BF%BB%E8%AF%91.png)

The Transformer was originally proposed mainly for translation tasks, so let's start by understanding it from the principle of translation.

First, we can observe that translation consists of two parts: **encoder** and **decoder**; here the **encoder** absorbs the sentence to be translated, such as *Machine Learning*, while the **decoder** outputs the corresponding *机器学习* (machine learning in Chinese).

Specifically, the entire translation process is divided into the following steps:

### Sentence Processing

For a passage, both the meaning of words themselves and their relative positions in the sentence are equally important. So the translator first obtains the vector corresponding to each word's own meaning and the vector corresponding to the word's position in the sentence through **token embeddings**; and adds them together. This way each word can be represented by a vector that simultaneously encodes meaning and position. Correspondingly, a sentence can be expressed by a matrix.

### Processing the Sentence Matrix

Then, we pass the sentence vector matrix $X$ we just obtained into the **encoder**, which outputs an encoded sentence $C$ containing all the sentence's information. $X$ and $C$ have exactly the same shape, both $n \times d$, where $n$ represents the number of words in the sentence, and $d$ represents the vector dimension of a word (the paper uses $d=512$).

### Translation

Now we pass the **encoder** output $C$ into the **decoder**, that is, we pass the information of the sentence to be translated. At the same time, we pass in a special symbol "$", which represents the beginning of a sentence. Then the **decoder** will output the first Chinese character "**机**" based on $C$ and "$".

Then, "**$+机**" replaces the original $ position as input, and together with $C$ passes through the **decoder** to output "**器**".

This cycle repeats until another special symbol representing the end of a sentence, "#", is output.

Below, we specifically introduce how the Transformer implements this process.

### Transformer Overall Architecture

![Transformer Architecture](transformer%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84.png)

As can be seen, the Transformer is similar in form to the translator we just discussed. Let's now expand on what each part is doing.

## Transformer Input

### Input Embedding

Inputs, that is, the text we input to be translated, after passing through the embedding vector, will be mapped to a high-dimensional space. That is, the "features" of words are extracted and placed in a matrix, which we might call the **semantic matrix**.

### Positional Encoding

Obtain the position information of each word in the sentence, also forming a matrix, which we might call the **position matrix**. Then add the semantic matrix and the position matrix to get the encoder's input.

There are many algorithms for the specific position matrix; here we introduce a relatively simple method:

$$
PE_{(pos,2i)}=\sin(\frac{pos}{10000^{\frac{2i}{d}}}), \quad
PE_{(pos,2i+1)}=\cos(\frac{pos}{10000^{\frac{2i}{d}}})
$$

Here $PE$ represents the position information of words; $pos$ represents the word's position in the sentence; $d$ represents the word's dimension, that is, the number of measures needed for the word's "feature"; $2i$ is the even dimension, and $2i+1$ is the odd dimension. Using this calculation method is simple enough and well satisfies several conditions:

- Ensures PE has the same size as words, that is, the semantic matrix and position matrix have the same size
- PE can adapt to situations where sentences in the validation set are longer than those in the training set
- Facilitates obtaining relative position information; for example, if two words are distance $k$ apart, knowing one allows quickly obtaining the other: $\sin(pos+k)=\sin(pos)\cos(k)+\cos(pos)\sin(k)$, $\cos(pos+k)=\cos(pos)\cos(k)-\sin(pos)\sin(k)$

### Entering the Encoder

Adding input embedding and positional embedding gives the matrix $X$ representing the sentence, which then enters the encoder.

## Self-Attention

Before entering the encoder, it is necessary to first introduce the self-attention mechanism, which is the essence of the Transformer. Let's take another look at the overall architecture.

![Transformer Architecture](transformer%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84.png)

Please note those **light orange** blocks. The **Self-Attention** here is what we will focus on next, and this structure is the same in both the encoder and decoder. As for "Multi-Head Attention" and "Masked Multi-Head Attention," they are essentially the same, just with some additional modifications.

### Self-Attention Structure

First, let's look at a simple example to understand what **attention** is. As shown in the figure below, attention is the degree of closeness between one word and another. Here **the green "The"** is the object we want to examine. We want to know how closely it is related to other words in the sentence, so we take the inner product of **The**'s representation vector $\vec x_{the}$ and the representation vectors of other words in the sentence $\vec x_{others}$. Then we use softmax for normalization to get the score of the relationship between **The** and other words.

Then, we use this "scoring system" to score each word in the sentence and sum them up to get the **attention** that **The** occupies in the entire sentence.

In fact, we can see that since every word in "The weather is nice today" needs the same operation, the entire process can be viewed as matrix multiplication:

$$
Attention = \text{softmax}(X \cdot X^{T}) \cdot X
$$

Here, the input and output matrix shapes are exactly the same; it's just that the "semantic matrix" of the entire sentence adjusts the "representation vector" of each word in the sentence. Moreover, since it is multiplying itself by itself, this process is **self**-attention.

At the same time, this process is also consistent with human language habits. The same word necessarily needs to be given different meanings depending on different sentences, such as "Apple phone" and "picking apples".

![Self-Attention Example](%E8%87%AA%E6%B3%A8%E6%84%8F%E5%8A%9B%E4%BE%8B%E5%AD%90.png)

### Self-Attention in Transformer

Then let's see how the Transformer applies self-attention.

![Self-Attention](%E8%87%AA%E6%B3%A8%E6%84%8F%E5%8A%9B%E6%9C%BA%E5%88%B6.png)

To enhance the effect of self-attention, before formal calculation, matrices $W_Q$ (query matrix), $W_K$ (key matrix), and $W_V$ (value matrix) are first used to correct the input ($X$).

$$
Q = X \cdot W_Q \\
K = X \cdot W_K \\
V = X \cdot W_V
$$

This gives three different matrices: Q (query), K (key), and V (value). Compared to $X$, $Q, K, V$ have unchanged row numbers but changed column numbers; equivalently, the number of words remains unchanged but the number of features measuring each word has changed.

Then, these three are passed into the mechanism shown in the figure above to calculate Self-Attention:

$$
Z = Attention(Q,K,V) = \text{softmax}(\frac{Q \cdot K^{T}}{\sqrt{d_k}}) \cdot V
$$

The following figure may better express the entire process.

![Self-Attention](self-attention.png)

Here, to prevent the inner product from becoming too large, it is divided by $d_k$, which is the number of columns of $K$. Similarly, the entire process does not change the row number of $X$, only the column number. Here $Z$ is the output matrix of self-attention.

### Multi-Head Attention

In the previous step, we obtained the output matrix $Z$ through Self-Attention, but this is not enough. One reason is that the sentence information obtained this way is insufficient, leading to insufficient accuracy. The essence of solving this problem is to extract more information from the sentence. A naive idea here is to "read it multiple times." Multi-Head Attention is such an idea.

![Multi-Head Attention](Multi-Head%20Attention.png)

From the figure above, we can see that Multi-Head Attention contains multiple Self-Attention layers. First, the input $X$ is passed to $h$ different Self-Attention modules, calculating $h$ output matrices $Z$. Here is the case when $h=8$, meaning 8 output matrices $Z$ will be obtained.

Then the output $Z_i$ are concatenated together, and a linear transformation is applied to keep the matrix size unchanged before and after multi-head attention. Analogously, we can naturally think of this linear transformation as fusing information obtained from multiple readings.

The following figure has a small problem: the $X$ and $D_X$ on the leftmost matrix are written in reverse; the small matrices in the middle are written correctly. However, this figure looks more intuitive, so it's included here.

![MHA Illustration](Multi-Head%20Attention-%E7%A4%BA%E6%84%8F.png)

### Multi-Layer Self-Attention

This idea seems so natural. Just consider our MLP and you can clearly see the benefits of increasing network depth. So here we don't want to discuss such mechanisms, but rather find this figure interesting. I didn't understand it the first time, and I think it's worth discussing.

![Multi-Layer Self-Attention](Multi-Layer%20Self-Attention.png)

In the figure above, each large rectangle wraps a Self-Attention in Transformer. **A hexagon represents the attention between one word and other words**. Looking carefully, you can see that the red dots in each hexagon are different. After one Self-Attention in Transformer is implemented, some nonlinear function $f(\cdot)$ is used to pass the output to the next Self-Attention in Transformer, and this is repeated several times.

## Encoder

After discussing Self-Attention, we can finally formally enter the Transformer.

![Transformer Architecture](transformer%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84.png)

The light orange part on the left side of the figure above is our encoder, or encoder block, because we need to use this structure repeatedly $N$ times.

We have already discussed Multi-Head Attention in detail, so here we focus on Add & Norm and Feed Forward.

### Add

Add is the residual connection, expressed by the formula $X + \text{MultiHeadAttention}(X)$. Such operations were already seen in RNNs; the benefit is avoiding gradient vanishing in backpropagation.

Here we can see the benefit of keeping $Z$ and $X$ the same shape.

### Norm

Normalization needs little explanation; it's just Gaussian/normalization. $\text{LayerNorm}(X + \text{MultiHeadAttention}(X))$

Layer Normalization converts the input of each layer of neurons to have the same mean and variance, which can accelerate convergence.

### Feed Forward

The Feed Forward layer is relatively simple, a two-layer fully connected layer with ReLU as the activation function for the first layer and no activation function for the second layer: $\text{ReLU}(XW_1+b_1)W_2+b_2$

### Assembling the Head

Multi-Head Attention, Feed Forward, and Add & Norm can construct an Encoder block. The Encoder block receives the input matrix $X$ and outputs a matrix $C$. Multiple Encoder blocks are stacked to form the Encoder.

The first Encoder block's input is the word representation vector matrix of the sentence. Subsequent Encoder blocks' inputs are the outputs of the previous Encoder block. The last Encoder block's output matrix is the encoded information matrix $C$, which will be used in the Decoder later.

## Decoder

![Transformer Architecture](transformer%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84.png)

The light orange part on the right side of the figure above is our decoder, or decoder block, because we also need to use this structure repeatedly $N$ times.

Although there are two here, they are essentially the same, both being our Multi-Head Attention; it's just that the input $X$ is different from before.

In summary:

- The first Multi-Head Attention layer uses Masked
- The second Multi-Head Attention layer uses the Encoder's encoded information matrix $C$ to calculate K, V matrices, while Q uses the previous Decoder block's output or input $X$ to calculate

Here, let's think about why there is this arrangement of "the K, V matrices of the Multi-Head Attention layer use the Encoder's encoded information matrix $C$ to calculate."

### Masked Multi-Head Attention

Mask, occlusion — I feel this is very intuitive, but something I couldn't think of myself. It's the kind of thing that seems obvious after hearing someone else's explanation, but you would absolutely never think of it yourself.

It comes from a reality: when we do translation, we only know the preceding context of the translated content, not the following context. Imagine you are reading an English book; the English content is naturally complete, but the Chinese meaning appears one by one in sequence. So when calculating **attention**, we need to shield the connection between this word and words that come after it in the sentence. This part feels a bit convoluted.

I wonder if you still remember the concept of **attention**. It's actually the degree of closeness between one word and all other words in a sentence. The general measure is the size of the inner product of the word's **representation vector**. Since every word in the sentence needs to obtain its own attention, the actual calculation is the dot product of the sentence's **semantic matrix** with its own transpose.

But as mentioned earlier, in the Transformer, for better performance, we use linear transformations to convert $X$ into $Q, K, V$, and we also said $Z = Attention(Q,K,V) = \text{softmax}(\frac{Q \cdot K^{T}}{\sqrt{d_k}}) \cdot V$. I wonder if you can see which part is the **attention matrix** (a term I just made up).

It's actually $Q \cdot K^{T}$. We just need to multiply it by a mask matrix, that is, $Q \cdot K^T \cdot \text{mask}$.

So, what kind of mask matrix can satisfy such requirements? As shown below:

$$
\begin{bmatrix}
{1} & {0} & {\cdots} & {0} \\
{1} & {1} & {\cdots} & {0} \\
{\vdots} & {\vdots} & {\ddots} & {\vdots} \\
{1} & {1} & {\cdots} & {0}
\end{bmatrix}
$$

This way, our attention becomes: $Attention(Q,K,V) = \text{softmax}(\frac{Q \cdot K^{T} \cdot \text{Mask}}{\sqrt{d_k}}) \cdot V$

We obtain a Mask Self-Attention output matrix. Similar to the Encoder, through Multi-Head Attention, multiple outputs are concatenated and then linearly transformed to get the first Multi-Head Attention output $Z$, which has the same dimensions as input $X$.

In fact, when I was about to write Multi-Head Attention-2, I suddenly realized there might be some ambiguity here regarding the input of Masked Multi-Head Attention.

What I want to say is that the input of Masked Multi-Head Attention has no explicit relationship with the encoder; subsequent decoder blocks will naturally have implicit connections! The $Q, K, V$ in the input of Masked Multi-Head Attention are obtained from a matrix $X$ transformation!

### Multi-Head Attention-2

Well, Multi-Head Attention-2 is also easy to understand. The only thing worth noting is that the Self-Attention K, V matrices are not calculated using the previous Decoder block's output, but using the Encoder's encoded information matrix $C$.

I wonder if you still remember the previous question.

Here, it's not hard to answer. Think about how our current $Q, K, V$ are obtained:

$$
Q = X \cdot W_Q \\
K = C \cdot W_K \\
V = C \cdot W_V
$$

Then think about what attention is: $Attention(Q,K,V) = \text{softmax}(\frac{Q \cdot K^{T}}{\sqrt{d_k}}) \cdot V$. We are essentially examining the relationship between the input $X$, such as a start symbol, and the content of the sentence to be translated; then adjusting the content of the sentence to be translated based on this relationship.

The advantage of doing this is that in the Decoder, every output word can utilize information from all words in the Encoder.

## Softmax

After finishing the Decoder, because it's also a "Multi-Head," a linear transformation is first applied to keep the size the same as at the beginning. Then softmax is used to complete scoring.

After that, it's no different from the subsequent steps of MLP: comparing with the decoder input $X$ to get loss and accuracy, choosing an appropriate optimizer to compute gradients, etc.

## Summary

- The Transformer does not utilize word order information, so positional embeddings need to be added to the input.
- The Self-Attention structure is the focus of the Transformer, where the Q, K, V matrices are obtained through linear transformation of $X$.
- Multi-Head Attention contains multiple Self-Attention modules, aiming to capture more relationships between words.
