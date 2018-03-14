
# Using the CLI
The Kubernetes client, `kubectl` is the primary method of interacting with a Kubernetes cluster. Getting to know it is essential to using Kubernetes itself.


## Index
* [Syntax Structure](#syntax-structure)
* [Context and kubeconfig](#context-and-kubeconfig)
  * [kubectl config](#kubectl-config)
  * [Exercise: Using Contexts](#exercise-using-contexts)
* [kubectl Basics](#kubectl-basics)
  * [kubectl get](#kubectl-get)
  * [kubectl create](#kubectl-create)
  * [kubectl apply](#kubectl-apply)
  * [kubectl delete](#kubectl-delete)
  * [kubectl describe](#kubectl-describe)
  * [kubectl logs](#kubectl-logs)
  * [Exercise: The Basics](#exercise-the-basics)
* [Accessing the Cluster](#accessing-the-cluster)
  * [kubectl proxy](#kubectl-proxy)
  * [Dashboard](#dashboard)
  * [Exercise: Using the Proxy](#exercise-using-the-proxy)
* [Cleaning up](#cleaning-up)
* [Helpful Resources](#helpful-resources)


---


## Syntax Structure

`kubectl` uses a common syntax for all operations in the form of:

```
kubectl <command> <type> <name> <flags>
```

* **command** - The command or operation to perform. e.g. `apply`, `create`, `delete`, and `get`
* **type** - The resource type or object
* **name** - The name of the resource or object
* **flags** - Optional flags to pass to the command

**Examples**
```
$ kubectl create -f mypod.yaml
$ kubectl get pods
$ kubectl get pod mypod
$ kubectl delete pod mypod
```


[Back to Index](#index)


---


## Context and kubeconfig
`kubectl` allows you to interact with and manage multiple Kubernetes clusters. To do this, it requires what is known as a `context`. A combination of `cluster`, `namespace` and `user`.
* **cluster** - A friendly name, server address, and certificate for the Kubernetes cluster.
* **namespace (optional)** - The logical cluster or environment to use. If none is provided, it will use the default `default` namespace.
* **user** - The credentials used to connect to the cluster. This can be a combination of client certificate and key, username/password, or token.

These contexts are stored in a local `yaml` based config file referred to as the `kubeconfig`. For `*nix` based systems, the `kubeconfig` is stored in `$HOME/.kube/config` for Windows, it can be found in `%USERPROFILE%/.kube/config`

This config is viewable without having to view the file directly.

**Command**
```
$ kubectl config view
```

**Example**
```
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/example/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    namespace: dev
    user: minikube
  name: minidev
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minidev
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/example/.minikube/client.crt
    client-key: /Users/example/.minikube/client.key
```


---


### `kubectl config`

Managing all aspects of contexts is done via the `kubectl config` command. You can:
* See the active context with `kubectl config current-context`
* Get a list of available contexts with `kubectl config get-contexts`
* Switch to using another one with the `kubectl config use-context <context-name>` command
* Add a new one with the `kubectl config set-context <context name> --cluster=<cluster name> --user=<user> --namespace=<namespace>`

There can be quite a few specifics involved when adding a context, for the available options, please see the [Configuring Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) Kubernetes documentaiton.


---


### Exercise: Using Contexts
**Objective:** Create a new context called `minidev` and switch to it.

1. View the current contexts
```
$ kubectl config get-contexts
```
2. Create a new context called `minidev` within the `minikube` cluster with the `dev` namespace, as the `minikube` user
```
$ kubectl config set-context minidev --cluster=minikube --user=minikube --namespace=dev
```
3. View the newly added context
```
kubectl config get-contexts
```
4. Switch to the `minidev` context using `use-context`
```
$ kubectl config use-context minidev
```
5. View the current active context
```
$ kubectl config current-context
```


[Back to Index](#index)


---


## Kubectl Basics
There are several `kubectl` commands you will frequently use for any sort of day-to-day operations. `get`, `create`, `apply`, `delete`, `describe`, and `logs`.  Other commands can be listed simply with `kubectl help`, or `kubectl <command> --help`.


### `kubectl get`
`kubectl get` fetches and lists objects of a certain type or a specific object itself. It also supports outputting the information in several different useful formats including: `json`, `yaml`, `wide` (additional columns), or`name` (names only) via the `-o` or `--output` flag.

**Command**
```
kubectl get <type>
kubectl get <type> <name>
kubectl get <type> <name> -o <output format>
```

**Examples**
```
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    4h
kube-public   Active    4h
kube-system   Active    4h
$
$kubectl get pod mypod -o wide
NAME      READY     STATUS    RESTARTS   AGE       IP           NODE
mypod     1/1       Running   0          5m        172.17.0.6   minikube
```


---


### `kubectl create`
`kubectl create` creates an object from a `json`/`yaml` manifest or `stdin`. The manifests can be specified with the `-f` or `--filename` flag that can point to either a file, or a directory containing multiple manifests.

**Command**
```
kubectl create <type> <parameters>
kubectl create -f <path to manifest>
```

**Examples**
```
$ kubectl create namespace dev
namespace "dev" created
$
$ kubectl create -f manifests/mypod.yaml
pod "mypod" created
```

---


### `kubectl apply`
`kubectl apply` is similar to `kubectl create`, however it will essentially update the resource if it is already created, or simply create it if does not yet exist. When it updates the config, it will save the previous version of it in an `annotation` on the created object itself. **WARNING:** If the object was not created initially with `kubectl apply` it's updating behaviour will act as a two-way diff. For more information on this, please see the [kubectl apply](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#kubectl-apply) documentation.

Just like `kubectl create` it takes a `json`/`yaml` manifest with the `-f` flag or accepts input from `stdin`.

**Command**
```
kubectl apply -f <path to manifest>
```

**Examples**
```
$ kubectl apply -f manifests/mypod.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
pod "mypod" configured
```

---


### `kubectl delete`
`kubectl delete` deletes the object from Kubernetes.

**Command**
```
kubectl delete <type> <name>
```

**Examples**
```
$ kubectl delete pod mypod
pod "mypod" deleted
```

---


### `kubectl describe`
`kubectl describe` lists detailed information about the specific Kubernetes object. It is a very helpful troubleshooting tool.

**Command**
```
kubectl describe <type>
kubectl describe <type> <name>
```

**Examples**
```
$ kubectl describe pod mypod
Name:         mypod
Namespace:    dev
Node:         minikube/192.168.99.100
Start Time:   Sat, 10 Mar 2018 13:12:53 -0500
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"mypod","namespace":"dev"},"spec":{"containers":[{"image":...
Status:       Running
IP:           172.17.0.6
Containers:
  nginx:
    Container ID:   docker://5a0c100de6599300b1565e73e64e8917f9a4f4b06325dc4890aad980d582cf04
    Image:          nginx:stable-alpine
    Image ID:       docker-pullable://nginx@sha256:db5acc22920799fe387a903437eb89387607e5b3f63cf0f4472ac182d7bad644
    Port:           80/TCP
    State:          Running
      Started:      Sat, 10 Mar 2018 13:12:53 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-s2xd7 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-s2xd7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-s2xd7
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     <none>
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              5s    default-scheduler  Successfully assigned mypod to minikube
  Normal  SuccessfulMountVolume  5s    kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-s2xd7"
  Normal  Pulled                 5s    kubelet, minikube  Container image "nginx:stable-alpine" already present on machine
  Normal  Created                5s    kubelet, minikube  Created container
  Normal  Started                5s    kubelet, minikube  Started container
  ```


---


### `kubectl logs`
`kubectl logs` outputs the combined `stdout` and `stderr` logs from a pod. If more tha one container exist in a `pod` the `-c` flag is used and the container name must be specified.

**Command**
```
kubectl logs <pod name>
kubectl logs <pod name> -c <container name>
```

**Examples**
```
$ kubectl logs mypod
172.17.0.1 - - [10/Mar/2018:18:14:15 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.57.0" "-"
172.17.0.1 - - [10/Mar/2018:18:14:17 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.57.0" "-"
```

---


### Exercise: The Basics
**Objective:** Create a namespace and a pod, then use the basics to describe and delete it.

**NOTE:** You should still be using the `minidev` context created earlier.

1) Create the `dev` namespace
```
kubectl create namespace dev
```

2) Apply the `mypod.yaml` manifest in the manifests folder
```
kubectl apply -f manifests/mypod.yaml
```

3) Get the `yaml` output of the created pod `mypod`
```
kubectl get pod mypod -o yaml
```

4) Describe the pod `mypod`
```
kubectl describe pod mypod
```

5) Clean up the pod by deleting it
```
kubectl delete pod mypod
```

[Back to Index](#index)


---


## Accessing the Cluster

`kubectl` provides several mechanisms for accessing resources within the cluster remotely. For this tutorial, the focus will be on using `kubectl proxy` to gain access to the API and the API proxy.

### `kubectl proxy`
`kubectl proxy` enables you to both access the Kubernetes API-Server or access a resource running with the cluster securely from your local computer. By default it creates a connection to the API-Server that can be accessed at `127.0.0.1:8001` or an alternative port by supplying the `-p` or `--port` flag.


**Command**
```
kubectl proxy
kubectl proxy --port=<port>
```

**Example**
```
$ kubectl proxy
Starting to serve on 127.0.0.1:8001

<from another terminal>
$ curl 127.0.0.1:8001/version
{
  "major": "",
  "minor": "",
  "gitVersion": "v1.9.0",
  "gitCommit": "925c127ec6b946659ad0fd596fa959be43f0cc05",
  "gitTreeState": "clean",
  "buildDate": "2018-01-26T19:04:38Z",
  "goVersion": "go1.9.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

The Kubernetes API-Server has the built in capability to proxy to running services or pods within the cluster. In conjunction with the `kubectl proxy` this allows you to access those services or pods without having to expose them outside of the cluster.

```
http://<proxy_address>/api/v1/namespaces/<namespace>/<services|pod>/<service_name|pod_name>[:port_name]/proxy
```
* **proxy_address** - The local proxy address - `127.0.0.1:8001`
* **namespace** - The namespace that the resource you wish to proxy to exists in
* **service|pod** - The type of resource you are trying to access, either `service` or `pod`.
* **service_name|pod_name** - The name of the `service` or `pod` you are trying to access..
* **[:port]** - An optional port to proxy to. Will default to the first one exposed.

**Example**
```
http://127.0.0.1:8001/api/v1/namespaces/default/pods/mypod/proxy/
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/
```


---


### Dashboard

While the Kubernetes Dashboard is not something that is required to be deployed in a cluster, it frequently is and is a handy tool to quickly explore the system; however it should not be relied upon for cluster support.

![Kubernetes Dashboard](images/dashboard.png)

To access the dashboard you use the `kubectl proxy` command and acess the `kubernetes-dashboard` service within the `kube-system` namespace.
```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/
```

Leaving the proxy up and going, may not be desirable for quick dev-work, so minikube itself has a command that will open the dashboard up in a new browser window through an exposed service on the minikube VM.

**Command**
```
$ minikube dashboard
```

---


### Exercise: Using the Proxy
**Objective:** Create a pod and access it through the proxy. Then access the dashboard via the minikube command.


1) Create the Pod `mypod` from the mypod manifest.
```
$ kubectl create -f manifests/mypod.yaml
```

2) Start the `kubectl proxy` with the defaults.
```
$ kubectl proxy
```

3) Access the Pod through the proxy.
```
http://127.0.0.1:8001/api/v1/namespaces/dev/pods/mypod/proxy/
```

4) Access the Dashboard through the proxy.
```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/
```

5) Access the Dashboard through minikube.
```
$ minikube proxy
```


[Back to Index](#index)


---


## Cleaning up
To remove everything that was created in this tutorial, execute the following commands:
```
kubectl delete pod mypod
kubectl delete namespace dev
kubectl config delete-context minidev
```

[Back to Index](#index)


---


### Helpful Resources
* [kubectl Overview](https://kubernetes.io/docs/reference/kubectl/overview/)
* [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [kubectl reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
* [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/)


[Back to Index](#index)