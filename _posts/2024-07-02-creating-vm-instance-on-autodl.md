---
layout: post
title: "Creating a VM Instance on AutoDL"
date: 2024-07-02 12:00:00 +0800
description: "A step-by-step guide to creating and connecting to a virtual machine instance on AutoDL for deep learning."
tags: [Git, Virtual Machine, Deep Learning, GitHub, ssh]
categories: [tools, deep-learning]
giscus_comments: true
---

# Creating a Virtual Machine

For first-time AutoDL users, after entering the [AutoDL website](https://www.autodl.com/), you first need to register an account. If you are a student, remember to complete **student verification**, which can save some money. These operations are relatively simple, so I won't go into detail here.

## Creating an Instance

After registration, click the **Console** button in the upper right corner to enter the console interface.

Then, enter the instance container.

Here, all rented instances are displayed. Click **Rent New Instance** to enter the instance creation interface.

You can see that you can choose the appropriate billing method, region, GPU type, number of GPUs, data disk size, image to use, etc.

Here, we choose **Pay-as-you-go**, **Chongqing Zone A**, **4090D×1**, **500GB data disk**, and the **image as shown**.

Then, click **Create Instance**, wait for a while, and the instance will be created successfully.

# Connecting to the Virtual Machine

Although AutoDL provides several different connection methods, I recommend using the **SSH** connection method, relying on **key-based authentication**.

## Setting up config

First, copy the **login command**, and you will get a command in the form of `ssh -p 12345 root@xxx.mmm.nnn.com`. Then, enter the `.ssh` folder under your user directory.

Enter the `config` file and write in the following format:

```
Host autodl
    HostName xxx.mmm.nnn.com
    Port 12345
    User root
    IdentityFile ~/.ssh/id_rsa_autodl
```

Then you can connect using `ssh autodl`.

# Environment Configuration

After connecting, you can configure the conda environment as needed.

For example, create a new environment:

```bash
conda create -n gene python=3.10
```

Activate the environment:

```bash
conda activate gene
```

You can also modify the environment prompt:

```bash
conda config --set env_prompt '({default_env})'
```

Then use `conda info --envs` to see that the `gene` environment has been activated.
