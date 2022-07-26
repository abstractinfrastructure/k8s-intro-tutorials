# Kubernetes Tutorials

## Before you begin

These tutorials accompany the presentation [Introduction to Kubernetes][intro-slides]  and make use of
[kind][kind]. A tool that allows users to quickly spin up and run a single instance of Kubernetes locally using
Docker. To install it and the other tutorial dependencies, see the
[Installation Guides](#installation-guides) section.

Each section assumes an instance of kind is up and running. To start kind for the first time, use the command:
```
kind create cluster
```

Tutorials have been validated against kind v0.14.0 running Kubernetes v1.24.x and kubectl 1.24.0

---

## Tutorial Index
* [cli](/cli/README.md) - Covers the basics of using `kubectl` to interact with a Kubernetes cluster.
* [core](/core/README.md) - Tutorial of the core concepts, or building blocks of Kubernetes.
* [workloads](/workloads/README.md) - Walkthrough of the different types of application workloads.
* [storage](/storage/README.md) - Explores the relationship between Persistent Volumes, Persistent Volume Claims,
and Volumes themselves.
* [configuration](/configuration/README.md) - Tutorials going over how to use the two Configuration objects
ConfigMaps and Secrets.
* [Examples](/examples/README.md) - Examples of full blown applications to explore after the tutorials have been
completed.

---

## Installation Guides

The Installation guides are centered around using Docker. Please ensure you have Docker installed by following
their [installation guide](https://docs.docker.com/engine/install/#desktop) for your platform.

* [OSX](#osx-installation-guide)
* [Windows](#windows-installation-guide)
* [Linux](#linux)
* [Verifying Install](#verifying-install)
* [Troubleshooting Install Problems](#troubleshooting-install-problems)

---

## OSX Installation Guide

Installation on OSX is done with [Homebrew][brew], an OSX package manager. If you have not installed it previously,
please see their [installation guide][brew] before continuing.

Install `git`, `kubectl`, and `kind`
```
brew install git kubernetes-cli kind
```

Once done, [verify your Install](#verifying-install).

**Optional:** You can improve the general user experience of working with `kubectl` by installing
[command-completion][osx-completion]

---

## Windows Installation Guide

Installation on Windows is done with [chocolatey][choco], a Windows Package Manager. If you have not
installed it previously, please see their [installation guide][choco-install] before continuing.

```
dism.exe /Online /Disable-Feature:Microsoft-Hyper-V
```

Install `git`, `kubectl`, and `kind`.
```
choco install git kubernetes-cli kind
```

Once done, [verify your Install](#verifying-install).

---

## Linux

Linux installation is different for each distro. General install information is linked below:

* [git][linux-git]
* [kubectl][linux-kubectl]
* [kind][linux-kind]

Once done, [verify your Install](#verifying-install).

**Optional:** You can improve the general user experience of working with `kubectl` by installing
[command-completion][linux-completion]

---

## Verifying Install

With the software installed you can verify it is working correctly by executing:
```
kind create cluster
```

This will take a little bit of time the first time it is run as it will download its needed dependencies and starts the
container. When it completes, you can verify it is working correctly by executing:
```
kubectl version
```

You should get output similar to the following:
```
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:50:19Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", GitCommit:"4ce5a8954017644c5420bae81d72b09b735c21f0", GitTreeState:"clean", BuildDate:"2022-05-19T15:39:43Z", GoVersion:"go1.18.1", Compiler:"gc", Platform:"linux/amd64"}
```

After that you may stop the container with:
```
kind delete cluster
```

---

## Troubleshooting Install Problems

(This tutorial has been recently updated. After running tutorials it will be updated with what we've learned!)



 [intro-slides]: https://docs.google.com/presentation/d/1zrfVlE5r61ZNQrmXKx5gJmBcXnoa_WerHEnTxu5SMco/edit?usp=sharing
 [kind]: https://kind.sigs.k8s.io
 [brew]: https://brew.sh/
 [osx-completion]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#on-macos-using-bash
 [choco]: https://chocolatey.org/
 [choco-install]: https://chocolatey.org/install
 [linux-git]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
 [linux-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl
 [linux-kind]: https://github.com/kubernetes-sigs/kind#installation-and-usage
 [linux-completion]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion