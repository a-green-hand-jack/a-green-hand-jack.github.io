---
layout: post
title: "Introduction to Diffusion Models"
date: 2024-05-22 12:00:00 +0800
description: "A brief overview of diffusion models for generative modeling. (Original content was lost during migration.)"
tags: [Deep Learning, Diffusion Models]
categories: [deep-learning, fundamentals]
giscus_comments: true
---

> **Note**: The original content of this article was lost during the blog migration from Hexo to Jekyll. This is a placeholder.

Diffusion models are a class of generative models that learn to reverse a gradual noising process. They have become the dominant approach for high-quality image generation, with notable examples including DALL-E, Stable Diffusion, and Imagen.

Key concepts include:
- **Forward process**: Gradually adding Gaussian noise to data over timesteps
- **Reverse process**: Learning to denoise and recover the original data
- **Score matching**: Estimating the gradient of the data distribution

For a comprehensive introduction, see [Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239) and [What are Diffusion Models?](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/).
