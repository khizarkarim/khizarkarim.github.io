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

> `<SSH options> <SSH key type> <PEM encoded public key> <comment>`

where options can be anything from `man authorized_keys`, or for more details, see the [authorized_keys man page](https://man.openbsd.org/OpenBSD-current/man8/sshd.8#AUTHORIZED_KEYS_FILE_FORMAT).

In that case, this is what the public key would look like:

`command="echo $SSH_ORIGINAL_COMMAND" ssh-rsa AAAAB... khizar@hostname`

This also goes to show that we have access to environment variables. We can also forward more environment variables to the remote server. This needs to be set using the `PermitUserEnvironment` variable in the `/etc/ssh/sshd_config` file. More importantly, how can we run multiple commands on the remote server?

The answer is simple: by taking the concept above one step further and running a script when we create a session on the remote server, like so:

`command="/bin/bash /usr/bin/run-this.bash" ssh-rsa AAAAB... khizar@hostname`

We'll need to use conditions in this script to check against a set of commands, hence, I used `case` statements in my code below:

```bash
#!/bin/bash

# We need to ensure that only certain commands are entered, I wanted my system to only be able to scp

if [[ -z "$SSH_ORIGINAL_COMMAND" || "${SSH_ORIGINAL_COMMAND%% *}" != "scp" ]]; then
  printf "Command not allowed!\n" >&2
  exit 1
fi

# Optionally run this to make sure newly created files are read-only
# umask 0377

while getopts "rdt:f:" opt ${SSH_ORIGINAL_COMMAND#* }; do
  case $opt in
  r)
    # recursive
    OPT_R="-r"
    ;;
  d)
    # directory
    OPT_D="-d"
    ;;
  t)
    # Transfer from the source
    CANONICAL_TARGET=$(readlink -m "$1$OPTARG")
    if [[ "${CANONICAL_TARGET}" != $(readlink -m "$1")* ]]; then
      printf "Error: Unsafe target\n" >&2
      exit 2
    fi

    OPT_T="-t ${CANONICAL_TARGET}"
    mkdir -pm 700 "${CANONICAL_TARGET}"
    ;;
  f)
    # Transfer to the source
    CANONICAL_TARGET=$(readlink -m "$1$OPTARG")
    if [[ "${CANONICAL_TARGET}" != $(readlink -m "$1")* ]]; then
      printf "Error: Unsafe target\n" >&2
      exit 2
    fi

    OPT_T="-f ${CANONICAL_TARGET}"
    ;;
  esac
done

scp $OPT_R $OPT_T

# run any post-processing below. I needed to send scp'd files to my Azure blob container, so I did that here
```
