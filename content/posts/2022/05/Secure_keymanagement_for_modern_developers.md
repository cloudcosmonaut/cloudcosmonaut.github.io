---
title: "Secure key management for modern developers"
date: 2022-05-27T12:00:01+02:00
tags: [ ssh, github, enterprise, server, ghes ]
draft: false
---

When we talk about password, most people use a password manager and no longer use the same password for multiple accounts. But I notice, that a lot of people share there SSH key with multiple accounts and don't even protect them with a passphrase.

# What do we need?

Similar to keys to your house keys, you want seperate keys for the different persona's you imporsonate or roles you have. E.g. for your home projects, your work projects, and client projects. But also per Saas or PaaS you connect to (e.g. GitHub, Azure DevOps, etc.).


You want this in two forms. Keys to connect to the remote systems, and keys to sign your commits. Commit signing will be story for another time ;) It's a good security practice, to add passwords to your keys. Please don't forget this. Services like GitHub and Azure DevOps can enforce this with a policy, but don't do this by default.

# Setting up SSH-keys for secure connection

Note, you should create keys according to the current standards. Currently, you should use a Elyptic curve based key. One downside, Azure DevOps doesn't support these modern keys and rely on a legacy RSA key.

Generating a key for my work role @Xpirit on GitHub:
```bash
ssh-keygen -t ed25519 -C "xpirit@github" -f ~/.ssh/id_ed25519_xpirit_github
```

```bash
ssh-keygen -t rsa -b 4096 -C "customer@azdo" -f ~/.ssh/id_rsa_customer_azdo
```

GitHub stores the keys on your account level, you can only assign specific keys to an organization when SSO is enabled for this organization.

You can add your ssh key [https://github.com/settings/keys](https://github.com/settings/keys).

On Azure DevOps, keys are spicificly bound to your account on an organization. So this gives you a little better fine grained control. You can only assign a key to a single organization. Azure DevOps will prevent you from re-using a key.

Just go the the accouns settings of your user on the organization, like:
[https://dev.azure.com/<<ORGANIZATION>>/_usersSettings/keys](https://dev.azure.com/<<ORGANIZATION>>/_usersSettings/keys).
