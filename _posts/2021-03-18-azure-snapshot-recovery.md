---
layout: default
title:  "VM Generation Issue After Recovery from Azure Snapshot"
categories: projects azure-snapshot-recovery
permalink: /projects/azure-snapshot-recovery-issue
---

A cost efficient way to back your disks up is to create snapshots. On Azure, you can create a managed disk from your backed up snapshot. You can then attach this managed disk to the machine you'd like to restore.

## TL; DR

There's an issue prevalent on Azure where if you try to recover an OS from a snapshot of a Generation v2 VM, the image that will be built of the resulting VM will be a Generation v1 VM and the disk won't attach to the VM because of a Generation mismatch.

## Details

**Details and screenshot to be added.**
