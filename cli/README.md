
# Using the CLI
The Kubernetes client, `kubectl` is the primary method of interacting with a Kubernetes cluster. Getting to know it
is essential to using Kubernetes itself.


## Index
* [Syntax Structure](#syntax-structure)
* [Context and kubeconfig](#context-and-kubeconfig)
  * [kubectl config](#kubectl-config)
  * [Exercise: Using Contexts](#exercise-using-contexts)
* [kubectl Basics](#kubectl-basics)
  * [kubectl get](#kubectl-get)
  * [kubectl create](#kubectl-create)
  * [kubectl apply](#kubectl-apply)
  * [kubectl edit](#kubectl-edit)
  * [kubectl delete](#kubectl-delete)
  * [kubectl describe](#kubectl-describe)
  * [kubectl logs](#kubectl-logs)
  * [Exercise: The Basics](#exercise-the-basics)
* [Accessing the Cluster](#accessing-the-cluster)
  * [kubectl exec](#kubectl-exec)
  * [Exercise: Executing Commands within a Remote Pod](#exercise-executing-commands-within-a-remote-pod)
  * [kubectl proxy](#kubectl-proxy)
  * [Dashboard](#dashboard)
  * [Exercise: Using the Proxy](#exercise-using-the-proxy)
* [Cleaning up](#cleaning-up)
* [Helpful Resources](#helpful-resources)

---

# Syntax Structure

`kubectl` uses a common syntax for all operations in the form of:

```
kubectl <command> <type> <name> <flags>
```

* **command** - The command or operation to perform. e.g. `apply`, `create`, `delete`, and `get`.
* **type** - The resource type or object.
* **name** - The name of the resource or object.
* **flags** - Optional flags to pass to the command.

**Examples**
```
$ kubectl create -f mypod.yaml
$ kubectl get pods
$ kubectl get pod mypod
$ kubectl delete pod mypod
```

---

[Back to Index](#index)

---
---


# Context and kubeconfig
`kubectl` allows a user to interact with and manage multiple Kubernetes clusters. To do this, it requires what is known
as a context. A context consists of a combination of `cluster`, `namespace` and `user`.
* **cluster** - A friendly name, server address, and certificate for the Kubernetes cluster.
* **namespace (optional)** - The logical cluster or environment to use. If none is provided, it will use the default
`default` namespace.
* **user** - The credentials used to connect to the cluster. This can be a combination of client certificate and key,
username/password, or token.

These contexts are stored in a local yaml based config file referred to as the `kubeconfig`. For \*nix based
systems, the `kubeconfig` is stored in `$HOME/.kube/config` for Windows, it can be found in
`%USERPROFILE%/.kube/config`

This config is viewable without having to view the file directly.

**Command**
```
$ kubectl config view
```

**Example**
```yaml
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

Managing all aspects of contexts is done via the `kubectl config` command. Some examples include:
* See the active context with `kubectl config current-context`.
* Get a list of available contexts with `kubectl config get-contexts`.
* Switch to using another context with the `kubectl config use-context <context-name>` command.
* Add a new context with `kubectl config set-context <context name> --cluster=<cluster name> --user=<user> --namespace=<namespace>`.

There can be quite a few specifics involved when adding a context, for the available options, please see the
[Configuring Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
Kubernetes documentation.

---

### Exercise: Using Contexts
**Objective:** Create a new context called `minidev` and switch to it.

---

1. View the current contexts.
```
$ kubectl config get-contexts
```

2. Create a new context called `minidev` within the `minikube` cluster with the `dev` namespace, as the
`minikube` user.
```
$ kubectl config set-context minidev --cluster=minikube --user=minikube --namespace=dev
```

3. View the newly added context.
```
kubectl config get-contexts
```

4. Switch to the `minidev` context using `use-context`.
```
$ kubectl config use-context minidev
```

5. View the current active context.
```
$ kubectl config current-context
```

---

**Summary:** Understanding and being able to switch between contexts is a base fundamental skill required by every
Kubernetes user. As more clusters and namespaces are added, this can become unwieldy. Installing a helper
application such as [kubectx](https://github.com/ahmetb/kubectx) can be quite helpful. Kubectx allows a user to quickly
switch between contexts and namespaces without having to use the full `kubectl config use-context` command.

---

[Back to Index](#index)

---
---

## Kubectl Basics
There are several `kubectl` commands that are frequently used for any sort of day-to-day operations. `get`, `create`,
`apply`, `delete`, `describe`, and `logs`.  Other commands can be listed simply with `kubectl --help`, or
`kubectl <command> --help`.

---

### `kubectl get`
`kubectl get` fetches and lists objects of a certain type or a specific object itself. It also supports outputting the
information in several different useful formats including: json, yaml, wide (additional columns), or name
(names only) via the `-o` or `--output` flag.

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
`kubectl create` creates an object from the commandline (`stdin`) or a supplied json/yaml manifest. The manifests can be
specified with the `-f` or  `--filename` flag that can point to either a file, or a directory containing multiple 
manifests.

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
`kubectl apply` is similar to `kubectl create`. It will essentially update the resource if it is already created, or
simply create it if does not yet exist. When it updates the config, it will save the previous version of it in an
`annotation` on the created object itself. **WARNING:** If the object was not created initially with
`kubectl apply` it's updating behavior will act as a two-way diff. For more information on this, please see the
[kubectl apply](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#kubectl-apply)
documentation.

Just like `kubectl create` it takes a json or yaml manifest with the `-f` flag or accepts input from `stdin`.

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

### `kubectl edit`
`kubectl edit` modifies a resource in place without having to apply an updated manifest. It fetches a copy of the
desired object and opens it locally with the configured text editor, set by the `KUBE_EDITOR` or `EDITOR` Environment
Variables. This command is useful for troubleshooting, but should be avoided in production scenarios as the changes
will essentially be untracked.

**Command**
```
$ kubectl edit <type> <object name>
```

**Examples**
```
kubectl edit pod mypod
kubectl edit service myservice
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
`kubectl describe` lists detailed information about the specific Kubernetes object. It is a very helpful
troubleshooting tool.

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
`kubectl logs` outputs the combined `stdout` and `stderr` logs from a pod. If more than one container exist in a
`pod` the `-c` flag is used and the container name must be specified.

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
**Objective:** Explore the basics. Create a namespace, a pod, then use the `kubectl` commands to describe and delete
what was created.

**NOTE:** You should still be using the `minidev` context created earlier.

---

1) Create the `dev` namespace.
```
kubectl create namespace dev
```

2) Apply the manifest `manifests/mypod.yaml`.
```
kubectl apply -f manifests/mypod.yaml
```

3) Get the yaml output of the created pod `mypod`.
```
kubectl get pod mypod -o yaml
```

4) Describe the pod `mypod`.
```
kubectl describe pod mypod
```

5) Clean up the pod by deleting it.
```
kubectl delete pod mypod
```

---

**Summary:** The `kubectl` _"CRUD"_ commands are used frequently when interacting with a Kubernetes cluster. These
simple tasks become 2nd nature as more experience is gained.

---

[Back to Index](#index)

---
---

# Accessing the Cluster

`kubectl` provides several mechanisms for accessing resources within the cluster remotely. For this tutorial, the focus
will be on using `kubectl exec` to get a remote shell within a container, and `kubectl proxy` to gain access to the
services exposed through the API proxy.

---

### `kubectl exec`
`kubectl exec` executes a command within a Pod and can optionally spawn an interactive terminal within a remote
container. When more than one container is present within a Pod, the `-c` or `--container` flag is required, followed
by the container name.

If an interactive session is desired, the `-i` (`--stdin`) and `-t` (`--tty`) flags must be supplied.

**Command**
```
kubectl exec <pod name> -- <arg>
kubectl exec <pod name> -c <container name> -- <arg>
kubectl exec  -i -t <pod name> -c <container name> -- <arg>
kubectl exec  -it <pod name> -c <container name> -- <arg>
```


**Example**
```
$ kubectl exec mypod -c nginx -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=mypod
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
NGINX_VERSION=1.12.2
HOME=/root
$
$ kubectl exec -i -t mypod -c nginx -- /bin/sh
/ #
/ # cat /etc/alpine-release
3.5.2
```

---

### Exercise: Executing Commands within a Remote Pod
**Objective:** Use `kubectl exec` to both initiate commands and spawn an interactive shell within a Pod.

---

1) If not already created, create the Pod `mypod` from the manifest `manifests/mypod.yaml`.
```
$ kubectl create -f manifests/mypod.yaml
```

2) Wait for the Pod to become ready (`running`).
```
$ kubectl get pods --watch
```

3) Use `kubectl exec` to `cat` the file `/etc/os-release`.
```
$ kubectl exec mypod -- cat /etc/os-release
```
It should output the contents of the `os-release` file.

4) Now use `kubectl exec` and supply the `-i -t` flags to spawn a shell session within the container.
```
$ kubectl exec -i -t mypod -- /bin/sh
```
If executed correctly, it should drop you into a new shell session within the nginx container.

5) use `ps aux` to view the current processes within the container.
```
/ # ps aux
```
There should be two nginx processes along with a `/bin/sh` process representing your interactive shell.

6) Exit out of the container simply by typing `exit`.
With that the shell process will be terminated and the only running processes within the container should
once again be nginx and its worker process.

---

**Summary:** `kubectl exec` is not often used, but is an important skill to be familiar with when it comes to Pod
debugging.

---

### `kubectl proxy`
`kubectl proxy` enables access to both the Kubernetes API-Server and to resources running within the cluster
securely using `kubectl`. By default it creates a connection to the API-Server that can be accessed at
`127.0.0.1:8001` or an alternative port by supplying the `-p` or `--port` flag.


**Command**
```
kubectl proxy
kubectl proxy --port=<port>
```

**Examples**
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

The Kubernetes API-Server has the built in capability to proxy to running services or pods within the cluster. This
ability in conjunction with the `kubectl proxy` command allows a user to access those services or pods without having
to expose them outside of the cluster.

```
http://<proxy_address>/api/v1/namespaces/<namespace>/<services|pod>/<service_name|pod_name>[:port_name]/proxy
```
* **proxy_address** - The local proxy address - `127.0.0.1:8001`
* **namespace** - The namespace owning the resources to proxy to.
* **service|pod** - The type of resource you are trying to access, either `service` or `pod`.
* **service_name|pod_name** - The name of the `service` or `pod` to be accessed.
* **[:port]** - An optional port to proxy to. Will default to the first one exposed.

**Example**
```
http://127.0.0.1:8001/api/v1/namespaces/default/pods/mypod/proxy/
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/
```
---

### Dashboard

While the Kubernetes Dashboard is not something that is required to be deployed in a cluster, it frequently is and is
a handy tool to quickly explore the system. **However**, it should not be relied upon for cluster support.

![Kubernetes Dashboard](images/dashboard.png)

To access the dashboard use the `kubectl proxy` command and access the `kubernetes-dashboard` service within the
`kube-system` namespace.
```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/
```

Leaving the proxy up and going may not be desirable for quick dev-work. Minikube itself has a command that will
open the dashboard up in a new browser window through an exposed service on the Minikube VM.

**Command**
```
$ minikube dashboard
```

---

### Exercise: Using the Proxy
**Objective:** Examine the capabilities of the proxy by accessing a pod's exposed ports and using the dashboard.

---

1) Create the Pod `mypod` from the manifest `manifests/mypod.yaml`. (if not created previously)
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
You should see the "Welcome to nginx!" page.


4) Access the Dashboard through the proxy.
```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/
```

5) Lastly, stop the proxy (`CTRL+C`) and access the Dashboard through minikube.
```
$ minikube dashboard
```
Minikube offers a convenient shortcut to access the dashboard (without the proxy) for local development use.

---

**Summary:** Being able to access the exposed Pods and Services within a cluster without having to consume an
external IP, or create firewall rules is an incredibly useful tool for troubleshooting cluster services.

---

[Back to Index](#index)

---
---

## Cleaning up
**NOTE:** If you are proceeding with the next tutorials, simply delete the pod with:
```
$ kubectl delete pod mypod
```
The namespace and context will be reused. 

To remove everything that was created in this tutorial, execute the following commands:
```
kubectl delete namespace dev
kubectl config delete-context minidev
```

---

[Back to Index](#index)

---
---

### Helpful Resources
* [kubectl Overview](https://kubernetes.io/docs/reference/kubectl/overview/)
* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [kubectl Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
* [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/)


[Back to Index](#index)
