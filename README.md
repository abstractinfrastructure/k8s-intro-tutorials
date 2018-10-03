# Kubernetes Tutorials

## Before you begin

These tutorials accompany the presentation [Introduction to Kubernetes](https://docs.google.com/presentation/d/1zrfVlE5r61ZNQrmXKx5gJmBcXnoa_WerHEnTxu5SMco/edit?usp=sharing) 
and make use of [Minikube](https://github.com/kubernetes/minikube). A tool that allows users to quickly spin up and
run a single instance of Kubernetes locally using a variety of virtualization engines.

Each section assumes an instance of minikube is up and running. To start minikube for the first time, use the command:
```
minikube start --kubernetes-version v1.11.3
```

To launch an alternative version of kubernetes within your minikube instance, supply an alternate version string:
```
minikube start --kubernetes-version <version string>
```

Tutorials have been validated against minikube v0.29 running Kubernetes v1.11.x.


## Installation Guides
* [OSX](#osx-installation-guide)
* [Windows](#windows-installation-guide)


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

## OSX Installation Guide
Installation on OSX is done with [Homebrew](https://brew.sh/), an OSX package manager. If you have not installed it
previously, please see their [installation guide](https://brew.sh/) before continuing.

Install `git`, `kubectl`, `minikube` and `virtualbox`.
```
brew install git kubernetes-cli
brew cask install minikube virtualbox
````

---

## Windows Installation Guide

Installation on Windows is done with [chocolatey](https://chocolatey.org/), a Windows Package Manager. If you have not
installed it previously, please see their [installation guide](https://chocolatey.org/install) before continuing.

**NOTE:** The Windows 10 Fall Creators Update enabled HyperV by default on supported hardware. This tutorial has
**NOT** been tested against HyperV, but should theoretically work. If you wish to use HyperV, you may skip disabling
it and the installation of Virtualbox. However minikube must be
[configured and started in a different manner](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperv-driver).

Disable HyperV (will require restart). HyperV unfortunately prevents Virtualbox from functioning properly and must
be disabled.
```
dism.exe /Online /Disable-Feature:Microsoft-Hyper-V
```

Install `git`, `kubectl`, `minikube` and `virtualbox`.
```
choco install git kubernetes-cli minikube virtualbox
```