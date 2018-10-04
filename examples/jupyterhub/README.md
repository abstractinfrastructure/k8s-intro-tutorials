# JupyterHub

[JupyterHub][hub] is a multi-user hub that spawns, manages, and proxies to single-user instances of the
[Jupyter notebook server][jupyter]. It is commonly used to serve notebooks to students, enterprise data scientists,
or other scientific research groups.

The JupyterHub Team has a project known as [Zero to JupyterHub][ztjh] that makes it easy to install and manage
JupyterHub within a Kubernetes cluster. This project makes use of [Helm][helm], a package manager for Kubernetes. 

The example application deployment found in this repo is a stripped down version of the standard helm deployment;
running in single-user mode. It is for demo and example purposes only, and should not be used in any production form.
For that, see the documentation at the [Zero to JupyterHub][ztjh] site.

## How does it Work?
The JupyterHub Kubernetes stack makes use of two major components: The hub which acts as a user notebook spawner and a
dynamic proxy to redirect a user to their specific notebook instance. 

When a user logs in, a new Pod is provisioned to serve as their personal notebook server and a proxy rule is added
automatically. Together they make for a fairly seamless Jupyter experience.

---

## Prereqs

Create the service accounts and rbac policies with the below command.
```
$ kubectl create -f manifests/rbac.yaml
```

**NOTE:** RBAC is out of scope for the introductory tutorials, however they're required for both the Hub and Proxy to
be able to communicate with the Kubernetes API. If you are interested at exploring RBAC, see the docs here:
[Using RBAC Authorization][rbac]

---

## Installation

1. Create the 3 ConfigMaps:
```
$ kubectl create \
  -f manifests/cm-hub-config.yaml \
  -f manifests/cm-ingress.yaml \
  -f manifests/cm-nginx.yaml
```
* **[cm-hub-config.yaml](manifests/cm-hub-config.yaml)** - Functions as the Config for JupyterHub and is mounted as a
  volume within the Hub Pod.
* **[cm-ingress.yaml](manifests/cm-ingress.yaml)** - A placeholder empty config used by the nginx ingress controller
  container within the Proxy Pod.
* **[cm-nginx.yaml](manifests/cm-nginx.yaml)** - Nginx specific configuration options.

2. Create the [secret](manifests/secret-hub.yaml) used by the Proxy to authenticate to the Hub.
```
$ kubectl create -f manifests/secret-hub.yaml
```

3. Create the [PVC](manifests/pvc-hub.yaml) used by the Hub to store it's internal database.
```
$ kubectl create -f manifests/pvc-hub.yaml
```

4. Now create the 4 services used by both the Hub and Proxy:
```
$ kubectl create \
  -f manifests/svc-hub.yaml \
  -f manifests/svc-proxy-api.yaml \
  -f manifests/svc-proxy-http.yaml \
  -f manifests/svc-proxy-public.yaml
```

* **[svc-hub.yaml](manifests/svc-hub.yaml)** - The internal ClusterIP service that targets the Hub server.
* **[svc-proxy-api.yaml](manifests/svc-proxy-api.yaml)** - Internal ClusterIP service that points to the JupyterHub
  [Configurable HTTP Proxy (CHP)][chp-proxy] api within the Proxy Pod.

* **[svc-proxy-http.yaml](manifests/svc-proxy-http.yaml)** - Internal ClusterIP service that points to CHP within the
  Proxy Pod, which in turn points to the Hub server.
* **[svc-proxy-public.yaml](manifests/svc-proxy-public.yaml)** - External User facing NodePort Service that maps to
  nginx within the Proxy pod. This service will direct the User to the Hub server and the spawned User Notebooks.

5. With everything else provisioned, the two deployments for the Hub Server and Proxy may now be created.
```
$ kubectl create \
 -f manifests/deploy-hub.yaml \
 -f manifests/deploy-proxy.yaml
```

* **[deploy-hub.yaml](manifests/deploy-hub.yaml)** - Hub server deployment.
* **[deploy-proxy.yaml](manifests/deploy-proxy.yaml)** - Proxy deployment.

6. Wait for the Pods to be up and running:
```
$ kubectl get pods --watch
```
**NOTE:** It is common for the Hub Server to restart at least once.

7. When ready, use `minikube service` to access the proxy-public service and login to JupyterHub with the credentials:
  `admin/admin`.
```
$ minikube service proxy-public
```
**NOTE:** It may take some time for the service to actually become available. Refresh it once or twice within 30 seconds.

8. Watch the Pods once again.
```
$ kubectl get pods --watch
```
There will be Pod spinning up with the name `jupyter-admin`. This is the dynamically provisioned notebook server being
  spun up. 

With that you should have a fully functional instance of the JupyterHub provisioned and available to explore.

---

## Clean Up

```
$ kubectl delete -f manifests/
$ kubectl delete pod jupyter-admin
$ kubectl delete pvc claim-admin
```

[hub]: https://jupyterhub.readthedocs.io/en/latest/
[jupyter]: https://jupyter-notebook.readthedocs.io/en/latest/
[ztjh]: https://zero-to-jupyterhub.readthedocs.io/en/latest/
[helm]: https://www.helm.sh/
[rbac]: https://kubernetes.io/docs/admin/authorization/rbac/
[chp-proxy]: https://github.com/jupyterhub/configurable-http-proxy