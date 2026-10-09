---
layout: post
title: "Creating a VM Instance on GCP"
date: 2024-05-25 12:00:00 +0800
description: "A practical guide to creating GPU-enabled virtual machine instances on Google Cloud Platform."
tags: [Deep Learning, Virtual Machine, ssh, Git, GitHub]
categories: [tools, deep-learning]
giscus_comments: true
---

# Creating an Instance

After entering the [GCP Console](https://console.cloud.google.com/), you will be automatically redirected to the welcome page of the corresponding project.

Then click **Create a VM** to enter the instance creation page.

Here, according to your needs, select the appropriate region, machine configuration, boot disk, set the corresponding identity and API access permissions, firewall, advanced options - network, etc.

After setting up, click **Create**.

It is worth noting that sometimes the **Monthly estimated cost** will show **Quota exceeded**. Just click *Request* below, and the request will generally be approved within 10 minutes.

When selecting a machine, GPU configuration is often the focus. This [table](https://cloud.google.com/compute/docs/gpus/gpu-regions-zones?hl=en) shows the GPU models provided by GCP in different regions.

This way, you can see the VM you created in the **VM instances** interface.

# SSH Connection

## Web Connection

After creating the VM, you can use SSH to connect to it, which is also very simple. Just click **Open in browser window** to enter the VM.

Here, because I need to use a GPU, I chose an **image** with NVIDIA drivers and CUDA pre-installed.

Finally, although there may be some warnings, it does not affect normal use.

```bash
nvidia-smi
```

## Local SSH Connection

You can also connect via local SSH. First, generate an SSH key pair locally, then add the public key to the VM's metadata, and finally connect using the private key.

# Installing Common Tools

After connecting to the VM, you may need to install some common tools:

## Git LFS

For managing large files with Git:

### Ubuntu/Debian

```bash
sudo apt-get update
sudo apt-get install git-lfs
```

Or add the Git LFS repository and install:

```bash
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
```

### CentOS/RHEL

```bash
sudo yum install epel-release
sudo yum install curl
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.rpm.sh | sudo bash
sudo yum install git-lfs
```

### Fedora

```bash
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.rpm.sh | sudo bash
sudo dnf install git-lfs
```

## Verification

After installation, you can verify whether each tool is installed successfully:

```bash
git lfs --version
```

If the corresponding version information is output, the installation is successful.

## Usage Example

```bash
git lfs install
git lfs track "*.psd"
git add .gitattributes
git add file.psd
git commit -m "Add design file"
git push origin main
```

Through the above steps, you should be able to successfully install and use `git-lfs` on your GCP VM.
