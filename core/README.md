# Exploring the Core

This tutorial covers the fundamental building blocks that make up Kubernetes. Understanding what these components are
and how they are used is crucial to learning how to use the higher level objects and resources.

# Index
* [Namespaces](#namespaces)
  * [Exercise: Using Namespaces](#exercise-using-namespaces)
* [Pods](#pods)
  * [Exercise: Creating Pods](#exercise-creating-pods)
* [Labels and Selectors](#labels-and-selectors)
  * [Exercise: Using Labels and Selectors](#exercise-using-labels-and-selectors)
* [Services](#services)
  * [Exercise: The ClusterIP Service](#exercise-the-clusterip-service)
  * [Exercise: Using the NodePort Service](#exercise-using-the-nodeport-service)
  * [Exercise: The LoadBalancer Service](#exercise-the-loadbalancer-service)
  * [Exercise: Using the ExternalName Service](#exercise-using-the-externalname-service)
* [Cleaning up](#cleaning-up)
* [Helpful Resources](#helpful-resources)

---

# Namespaces
Namespaces are a logical cluster or environment. They are the primary method of partitioning a cluster or scoping
access.

---

### Exercise: Using Namespaces
**Objectives:** Learn how to create and switch between Kubernetes Namespaces using `kubectl`.

**NOTE:** If you are coming from the [cli tutorial](../cli/README.md), you may have completed this already.

---

1) List the current namespaces
```
$ kubectl get namespaces
```

2) Create the `dev` namespace
```
$ kubectl create namespace dev
```

3) Create a new context called `minidev` within the `minikube` cluster  as the `minikube` user, with the namespace
 set to `dev`.
```
$ kubectl config set-context minidev --cluster=minikube --user=minikube --namespace=dev
```

4) Switch to the newly created context.
```
$ kubectl config use-context minidev
```

---

**Summary:** Namespaces function as the primary method of providing scoped names, access, and act as an umbrella for
group based resource restriction. Creating and switching between them is quick and easy, but learning to use them is
essential in the general usage of Kubernetes.

---

[Back to Index](#index)

---
---

# Pods
A pod is the atomic unit of Kubernetes. It is the smallest _“unit of work”_ or _“management resource”_ within the
system and is the foundational building block of all Kubernetes Workloads.

**Note:** These exercises build off the previous Core tutorials. If you have not done so, complete those before continuing.

---

### Exercise: Creating Pods
**Objective:** Examine both single and multi-container Pods; including: viewing their attributes through the cli and
their exposed Services through the API Server proxy.

---

1) Create a simple Pod called `pod-example` using the `nginx:stable-alpine` image and expose port `80`. Use the
manifest `manifests/pod-example.yaml` or the yaml below.

**manifests/pod-example.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
```

**Command**
```
$ kubectl create -f manifests/pod-example.yaml
```

2) Use `kubectl` to describe the Pod and note the available information.
```
$ kubectl describe pod pod-example
```

3) Use `kubectl proxy` to verify the web server running in the deployed Pod.

**Command**
```
$ kubectl proxy
```
**URL**
```
http://127.0.0.1:8001/api/v1/namespaces/dev/pods/pod-example/proxy/
```

The default **"Welcome to nginx!"** page should be visible.

4) Using the same steps as above, create a new Pod called `multi-container-example` using the manifest
`manifests/pod-multi-container-example.yaml` or create a new one yourself with the below yaml.

**manifests/pod-multi-container-example.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-example
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: content
    image: alpine:latest
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          echo $(date)"<br />" >> /html/index.html;
          sleep 5;
        done
  volumes:
  - name: html
    emptyDir: {}
```

**Command**
```
$ kubectl create -f manifests/pod-multi-container-example.yaml
```
**Note:** `spec.containers` is an array allowing you to use multiple containers within a Pod.

5) Use the proxy to verify the web server running in the deployed Pod.

**Command**
```
$ kubectl proxy
```
**URL**
```
http://127.0.0.1:8001/api/v1/namespaces/dev/pods/multi-container-example/proxy/
```

There should be a repeating date-time-stamp.

---

**Summary:** Becoming familiar with creating and viewing the general aspects of a Pod is an important skill. While it
is rare that one would manage Pods directly within Kubernetes, the knowledge of how to view, access and describe them
is important and a common first-step in troubleshooting a possible Pod failure.

---

[Back to Index](#index)

---
---

# Labels and Selectors
Labels are key-value pairs that are used to identify, describe and group together related sets of objects or
resources.

Selectors use labels to filter or select objects, and are used throughout Kubernetes.

---

### Exercise: Using Labels and Selectors
**Objective:** Explore the methods of labeling objects in addition to filtering them with both equality and
set-based selectors.

---

1) Label the Pod `pod-example` with `app=nginx` and `environment=dev` via `kubectl`.

```
$ kubectl label pod pod-example app=nginx environment=dev
```

2) View the labels with `kubectl` by passing the `--show-labels` flag
```
$ kubectl get pods --show-labels
```

3) Update the multi-container example manifest created previously with the labels `app=nginx` and `environment=prod`
then apply it via `kubectl`.

**manifests/pod-multi-container-example.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-example
  labels:
    app: nginx
    environment: prod
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: content
    image: alpine:latest
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 5;
        done
  volumes:
  - name: html
    emptyDir: {}
```

**Command**
```
$ kubectl apply -f manifests/pod-multi-container-example.yaml
```

4) View the added labels with `kubectl` by passing the `--show-labels` flag once again.
```
$ kubectl get pods --show-labels
```

5) With the objects now labeled, use an [equality based selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#equality-based-requirement) 
targeting the `prod` environment.

```
$ kubectl get pods --selector environment=prod
```

6) Do the same targeting the `nginx` app with the short version of the selector flag (`-l`).
```
$ kubectl get pods -l app=nginx
```

7) Use a [set-based selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement)
to view all pods where the `app` label is `nginx` and filter out any that are in the `prod` environment.

```
$ kubectl get pods -l 'app in (nginx), environment notin (prod)'
```

---

**Summary:** Kubernetes makes heavy use of labels and selectors in near every aspect of it. The usage of selectors
may seem limited from the cli, but the concept can be extended to when it is used with higher level resources and
objects.

---

[Back to Index](#index)

---
---

# Services
Services within Kubernetes are the unified method of accessing the exposed workloads of Pods. They are a durable
resource (unlike Pods) that is given a static cluster-unique IP and provide simple load-balancing through kube-proxy.

**Note:** These exercises build off the previous Core tutorials. If you have not done so, complete those before continuing.

---

### Exercise: The clusterIP Service
**Objective:** Create a `ClusterIP` service and view the different ways it is accessible within the cluster.

---

1) Create `ClusterIP` service `clusterip` that targets Pods labeled with `app=nginx` forwarding port `80` using
either the yaml below, or the manifest `manifests/service-clusterip.yaml`.

**manifests/service-clusterip.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Command**
```
$ kubectl create -f manifests/service-clusterip.yaml
```

2) Describe the newly created service. Note the `IP` and the `Endpoints` fields.
```
$ kubectl describe service clusterip
```

3) View the service through `kube proxy` and refresh several times. It should serve up pages from both pods.

**Command**
```
$ kubectl proxy
```
**URL**
```
http://127.0.0.1:8001/api/v1/namespaces/dev/services/clusterip/proxy/
```

4) Lastly, verify that the generated DNS record has been created for the Service by using nslookup within the
`example-pod` Pod that was provisioned in the [Creating Pods](#exercise-creating-pods) exercise.
```
$ kubectl exec pod-example -- nslookup clusterip.dev.svc.cluster.local
```
It should return a valid response with the IP matching what was noted earlier when describing the Service.

---

**Summary:** The `ClusterIP` Service is the most commonly used Service within Kubernetes. Every `ClusterIP` Service
is given a cluster unique IP and DNS name that maps to one or more Pod `Endpoints`. It functions as the main method in
which exposed Pod Services are consumed **within** a Kubernetes Cluster.

---

### Exercise: Using NodePort

**Objective:** Create a `NodePort` based Service and explore how it is available both inside and outside the cluster.

---

1) Create a `NodePort` Service called `nodeport` that targets Pods with the labels `app=nginx` and `environment=dev`
forwarding port `80` in cluster, and port `32410` on the node itself. Use either the yaml below, or the manifest
`manifests/service-nodeport.yaml`.

**manifests/service-nodeport.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport
spec:
  type: NodePort
  selector:
    app: nginx
    environment: prod
  ports:
  - nodePort: 32410
    protocol: TCP
    port: 80
    targetPort: 80
```

**Command**
```
$ kubectl create -f manifests/service-nodeport.yaml
```

2) Describe the newly created Service Endpoint. Note the Service still has an internal cluster `IP`, and now
additionally has a `NodePort`.
```
$ kubectl describe service nodeport
```

3) Use the `minikube service` command to open the newly exposed `nodeport` Service in a browser.
```
$ minikube service -n dev nodeport
```

4) Lastly, verify that the generated DNS record has been created for the Service by using nslookup within
the `example-pod` Pod.
```
$ kubectl exec pod-example -- nslookup nodeport.dev.svc.cluster.local
```
It should return a valid response with the IP matching what was noted earlier when describing the Service.

---

**Summary:** The `NodePort` Services extend the `ClusterIP` Service and additionally expose a port that is either
statically defined, as above (port 32410) or dynamically taken from a range between 30000-32767. This port is then
exposed on every node within the cluster and proxies to the created Service.

---

### Exercise: The LoadBalancer Service
**Objective:** Create a `LoadBalancer` based Service, and learn how it extends both `ClusterIP` and `NodePort` to
make a Service available outside the Cluster.

**Before you Begin**
To use Service Type `LoadBalancer` it requires integration with an external IP provider. In most cases, this is a
cloud provider which will likely already be integrated with your cluster.

For bare-metal and on prem deployments, this must be handled yourself. There are several available tools and products
that can do this, but for this example the Google [metalLB](https://github.com/google/metallb) provider will be used.

**NOTE:** If you are **NOT** using the default virtualbox deployment of Minikube, or using it with a different default
IP range. Edit the manifest `manifests/metalLB.yaml` and change the cidr range on line 20 (`192.168.99.224/28`) to
fit your requirements. Otherwise go ahead and deploy it.
```
$ kubectl create -f manifests/metalLB.yaml
```

1) Create a `LoadBalancer` Service called `loadbalancer` that targets pods with the labels `app=nginx` and
`environment=prod` forwarding as port `80`. Use either the yaml below, or the manifest
`manifests/service--loadbalancer.yaml`.

**manifests/service-loadbalancer.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
    environment: prod
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Command**
```
$ kubectl create -f manifests/service-loadbalancer.yaml
```

2) Describe the Service `loadbalancer`, and note the Service retains the aspects of both the `ClusterIP` and
`NodePort` Service types in addition to having a new attribute `LoadBalancer Ingress`.
```
$ kubectl describe service loadbalancer
```

3) Open a browser and visit the IP noted in the `Loadbalancer Ingress` field. It should directly map to the exposed
Service.

4) Use the `minikube service` command to open the `NodePort` portion of the `loadbalancer` Service in a new browser
window.
```
$ minikube service -n dev loadbalancer
```

5) Finally, verify that the generated DNS record has been created for the Service by using nslookup within the
`example-pod` Pod.
```
$ kubectl exec pod-example -- nslookup loadbalancer.dev.svc.cluster.local
```
It should return a valid response with the IP matching what was noted earlier when describing the Service.

---

**Summary:** `LoadBalancer` Services are the second most frequently used Service within Kubernetes as they are the
main method of directing external traffic into the Kubernetes cluster. They work with an external provider to map
ingress traffic destined to the `LoadBalancer Ingress` IP to the cluster nodes on the exposed `NodePort`. These in
turn direct traffic to the desired Pods.

---

### Exercise: Using the ExternalName Service
**Objective:** Gain an understanding of the `ExternalName` Service and how it is used within a Kubernetes Cluster.

---

1) Create an `ExternalName` service called `externalname` that points to `google.com`
```
$ kubectl create service externalname externalname --external-name=google.com
```

2) Describe the `externalname` Service. Note that it does **NOT** have an internal IP or other _normal_ service
attributes.
```
$ kubectl describe service externalname
```

3) Lastly, look at the generated DNS record has been created for the Service by using nslookup within the
`example-pod` Pod. It should return the IP of `google.com`.
```
$ kubectl exec pod-example -- nslookup externalname.dev.svc.cluster.local
```

---

**Summary:** `ExternalName` Services create a `CNAME` entry in the Cluster DNS. This provides an avenue to use
internal Service discovery methods to reference external entities.

---

[Back to Index](#index)

---
---

# Cleaning Up

To remove everything that was created in this tutorial, execute the following commands:
```
kubectl delete namespace dev
kubectl delete -f manifests/metalLB.yaml
kubectl config delete-context minidev
kubectl config use-context minikube
```

---

[Back to Index](#index)

---
---

# Helpful Resources

* [Pod Object Spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#podspec-v1-core)
* [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
* [Concepts: Service Networking](https://kubernetes.io/docs/concepts/services-networking/service/)

---

[Back to Index](#index)

---
