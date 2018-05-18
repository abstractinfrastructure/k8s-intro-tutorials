# Kubernetes Tutorials

## Before you begin

These tutorials accompany the presentation [Introduction to Kubernetes](https://docs.google.com/presentation/d/1zrfVlE5r61ZNQrmXKx5gJmBcXnoa_WerHEnTxu5SMco/edit?usp=sharing) 
and make use of [Minikube](https://github.com/kubernetes/minikube). A tool that allows users to quickly spin up and
run a single instance of Kubernetes locally using a variety of virtualization engines.

Each section assumes an instance of minikube is up and running (`minikube start`).

## Index
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

Tutorials have been validated against minikube v0.26.1 running Kubernetes v1.10.