---
layout: post
title: "GitHub Connection Under Extreme Conditions"
date: 2024-10-17 12:00:00 +0800
description: "A guide for multiple researchers sharing a single server account to use Git and GitHub independently."
tags: [SSH, Git, GitHub]
categories: [tools, tips]
giscus_comments: true
---

Sometimes, graduate students in universities need or have to use their school's servers. Under such conditions, students usually have one independent account each; but there are exceptions. For example, a capable PhD student may have several undergraduate students. In such cases, the undergraduates, especially exchange students, often do not have their own server accounts. As a result, they have to share a single server account. In most situations this is fine, but what if they want to use Git and GitHub to manage and commit their own code? In other words, how can multiple researchers reasonably use their own Git and GitHub under a single device and a single account? 🎈

Unfortunately, this problem may be too niche, and the scenario too extreme, so few people have considered it carefully. Sometimes, just to save trouble, students even use SSH to transfer code directly between their local machine and the school server. To solve this problem, I wrote this post before losing 114,514 hairs ＞﹏＜, for your reference.

# Create a Personal Working Directory

Here we assume that you have successfully logged into the server.

First, create your personal working directory under the shared account. The shared account is usually the user folder, but sometimes it is a designated folder, depending on the requirements of different schools and labs.

# Generate SSH Keys

## Generate personal SSH keys

```bash
# Generate personal SSH keys
ssh-keygen -t rsa -b 4096 -f ~/.ssh_myname/id_rsa_myname -C "myname@example.com"
```

## Configure SSH config

Create and edit the SSH configuration file:

```bash
# ~/.ssh_myname/config
Host github.com-myname
    HostName github.com
    User git
    IdentityFile ~/.ssh_myname/id_rsa_myname
```

## Add public key to GitHub

View and copy the public key content:

```bash
# Note: you should be in myname_workspace at this point
cat .ssh_myname/id_rsa_myname.pub
```

Add the public key to your [GitHub account settings](https://github.com/settings/keys).

# Update the Agent

Add the SSH key to the SSH agent:

```bash
eval $(ssh-agent -s)  # Confirm the agent is working
ssh-add .ssh_myname/id_rsa_myname  # Temporarily add your own key
```

# Start Working

After updating the agent, you should generally be able to work normally. At this point, Git operations are no different from usual 🎉. You can clone a private repository to test if it works. If something goes wrong, try updating the agent again.

# Tips

Please note that the file paths here are all under your own working directory. Be careful not to accidentally navigate to the shared account (usually `home`) 🤣.
