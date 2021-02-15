---
layout: default
title:  "Restriction Using SSH Public Keys"
description: "How we can restrict users to specific commands using their public keys"
tags: projects ssh-key command restriction
permalink: /projects/ssh-key-command-restriction
---

## Public Keys

___

To use public keys to restrict users to specific commands, we need to first know how public and private keys work, specifically in the context of logging in to systems. This is fairly high-level, and we're not getting into how the SSH protocol checks for available authentication methods.

When a user logs in to a system using an SSH keypair:

1. The username they are using to login is checked in `/etc/passwd`
2. The path to the home directory is checked in that file
3. The `authorized_keys` file in the user's `.ssh` directory is opened
4. Of the public keys found in the `authorized_keys` file, the keys are checked against the private key on the client system

To delve into more detail, here's what an OpenSSH formatted public key looks like:

> \<SSH options\> \<SSH key type\> \<PEM encoded public key\> \<comment\>

where options can be anything from `man ssh`.
