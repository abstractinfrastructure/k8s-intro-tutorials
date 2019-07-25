# Kubernetes Tutorials

## Before you begin

These tutorials accompany the presentation [Introduction to Kubernetes][intro-slides]  and make use of
[Minikube][minikube]. A tool that allows users to quickly spin up and run a single instance of Kubernetes locally using
a variety of virtualization engines. To install it and the other tutorial dependencies, see the
[Installation Guides](#installation-guides) section.

Each section assumes an instance of minikube is up and running. To start minikube for the first time, use the command:
```
minikube start --kubernetes-version v1.15.1
```

To launch an alternative version of kubernetes within your minikube instance, supply an alternate version string:
```
minikube start --kubernetes-version <version string>
```

Tutorials have been validated against minikube v1.2.0 running Kubernetes v1.15.1 and kubectl 1.15.1

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

Ensure your laptop has Virtualization Extensions (Intel VT or AMD-V) enabled and is capable of running a 2 core, 2GB
RAM Virtual Machine.

The Installation guides are centered around using the Virtualbox Hypervisor. [Others are available][minikube-hypervisors],
but their installation and use have not been tested. 

* [OSX](#osx-installation-guide)
* [Windows](#windows-installation-guide)
* [Linux](#linux)
* [Verifying Install](#verifying-install)
* [Troubleshooting Install Problems](#troubleshooting-install-problems)

---

## OSX Installation Guide

Installation on OSX is done with [Homebrew][brew], an OSX package manager. If you have not installed it previously,
please see their [installation guide][brew] before continuing.

**WARNING:** If you have previously installed virtualbox manually, do not try and upgrade it or reinstall it via brew.
Upgrade it by manually by [downloading it from the virtualbox site][vbox-dl] or by uninstalling it then reinstalling
via brew.

**Note to High Sierra (10.13)+ Users:** High Sierra introduced stricter security settings over the area in which brew
installs some of its binaries. If you had brew previously installed (pre High Sierra) and have not done anything with it
in some time, it should be reinstalled. For details regarding this issue,
[see this stackoverflow thread][so-osx-brew-bug].


Install `git`, `kubectl`, `minikube` and `virtualbox`.
```
brew install git kubernetes-cli
brew cask install minikube virtualbox
````

Once done, [verify your Install](#verifying-install).

**Optional:** You can improve the general user experience of working with `kubectl` by installing
[command-completion][osx-completion]

---

## Windows Installation Guide

Installation on Windows is done with [chocolatey][choco], a Windows Package Manager. If you have not
installed it previously, please see their [installation guide][choco-install] before continuing.

**NOTE:** The Windows 10 Fall Creators Update enabled HyperV by default on supported hardware. This tutorial has
**NOT** been tested against HyperV, but should theoretically work. If you wish to use HyperV, you may skip disabling
it and the installation of Virtualbox. However minikube must be
[configured and started in a different manner][minikube-hyperv].

Disable HyperV (will require restart). HyperV unfortunately prevents Virtualbox from functioning properly and must
be disabled.
```
dism.exe /Online /Disable-Feature:Microsoft-Hyper-V
```

Install `git`, `kubectl`, `minikube` and `virtualbox`.
```
choco install git kubernetes-cli minikube virtualbox
```

Once done, [verify your Install](#verifying-install).

---

## Linux

Linux installation is different for each distro. General install information is linked below:

* [git][linux-git]
* [kubectl][linux-kubectl]
* [minikube][linux-minikube]
* [virtualbox][linux-vbox]

Once done, [verify your Install](#verifying-install).

**Optional:** You can improve the general user experience of working with `kubectl` by installing
[command-completion][linux-completion]

---

## Verifying Install

With the software installed you can verify it is working correctly by executing:
```
minikube start --kubernetes-version v1.15.1
```

This will take a little bit of time the first time it is run as it will download its needed dependencies and starts the
virtual machine. When it completes, you can verify it is working correctly by executing:
```
kubectl version
```

You should get output similar to the following:
```
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T14:25:20Z", GoVersion:"go1.12.7", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

After that you may stop the virtual machine with:
```
minikube stop
```

---

## Troubleshooting Install Problems

### OSX: Virtualbox fails to install via brew

Error Message:
```
installer: The install failed (The Installer encountered an error that caused the installation to fail. Contact the software manufacturer for assistance.)
==> Purging files for version 5.2.18,124319 of Cask virtualbox
Error: Failure while executing; `/usr/bin/sudo -E -- env LOGNAME=bob USER=bob USERNAME=bob /usr/sbin/installer -pkg /usr/local/Caskroom/virtualbox/5.2.18,124319/VirtualBox.pkg -target /` exited with 1. Here's the output:
installer: Package name is Oracle VM VirtualBox
installer: Installing at base path /
installer: The install failed (The Installer encountered an error that caused the installation to fail. Contact the software manufacturer for assistance.)
```

This occurs on High Sierra if you had brew previously installed before upgrading OSX. High Sierra introduced stricter security settings over the area in which brew
installs some of its binaries. 

To fix the issue, simply reinstall brew following the directions on the [brew site][brew]. Then proceed with installing
virtualbox.

 For details regarding this issue, [see this stackoverflow thread][so-osx-brew-bug].

---

### All Platforms: Minikube fails to start with a "kubeadm init error"

This can be caused by a variety of issues. However the solution is the same. 

First try deleting and recreating the minikube instance
```
minikube delete
minikube start --kubernetes-version v1.15.1
```

If that does not resolve the issue, delete the local minikube configs and try starting it again
```
minikube delete
rm -rf ~/.minikube
minikube start --kubernetes-version v1.15.1
```
That should hopefully resolve the kubeadm init error.



 [intro-slides]: https://docs.google.com/presentation/d/1zrfVlE5r61ZNQrmXKx5gJmBcXnoa_WerHEnTxu5SMco/edit?usp=sharing
 [minikube]: https://github.com/kubernetes/minikube
 [minikube-hypervisors]: https://github.com/kubernetes/minikube#requirements
 [brew]: https://brew.sh/
 [vbox-dl]: https://www.virtualbox.org/wiki/Downloads
 [so-osx-brew-bug]: https://stackoverflow.com/questions/46459152/cant-chown-usr-local-for-homebrew-in-osx-10-13-high-sierra
 [osx-completion]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#on-macos-using-bash
 [choco]: https://chocolatey.org/
 [choco-install]: https://chocolatey.org/install
 [minikube-hyperv]: https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperv-driver
 [linux-git]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
 [linux-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl
 [linux-minikube]: https://github.com/kubernetes/minikube#linux
 [linux-vbox]: https://www.virtualbox.org/wiki/Linux_Downloads
 [linux-completion]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion