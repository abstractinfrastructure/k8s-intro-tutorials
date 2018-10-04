# WordPress

[WordPress][wordpress] is a commonly used Blog and CMS engine that serves as an excellent introduction to a multi-tier
application.

The WordPress example has two major components: A MySQL database to serve as the backing datastore  and the WordPress
container itself that combines the Apache webserver along with PHP and the needed application dependencies to run an
instance of the blog engine. 

The manifests used in this example function as the bare minimum to provision an instance and should not be used in a
production deployment. For a more production ready deployment, see the [WordPress Helm Chart][wordpress-chart].

---

## Installation

1. Create the Secret used for the MySQL root account:
```
$ kubectl create -f manifests/secret-mysql.yaml 
```

* **[manifests/secret-mysql.yaml](manifests/secret-mysql.yaml)** - Contains a base64 encoded string to serve as the
  MySQL Database password.


2. Create the MySQL [StatefulSet](manifests/sts-mysql.yaml) and its associated [service](manifests/svc-mysql.yaml).
```
$ kubectl create \
  -f manifests/sts-mysql.yaml \
  -f manifests/svc-mysql.yaml
```

* **[manifests/sts-mysql.yaml](manifests/sts-mysql.yaml)** - MySQL StatefulSet.
* **[manifests/svc-mysql.yaml](manifests/svc-mysql.yaml)** - Associated MySQL Service.

**NOTE:** The MySQL StatefulSet does not require a PVC to be created ahead of time for its storage. Instead, it uses
the `volumeClaimTemplates` StatefulSet feature in combination with the default StorageClass provided by Minikube to
dynamically provision a volume.

3. Wait for the Pod to be up and running:
```
$ kubectl get pods --watch
```

3. With MySQL up and running, WordPress can now be provisioned. Start by Creating the 
   [PVC](manifests/pvc-wordpress.yaml) used to store WordPress's internal data.
```
$ kubectl create -f manifests/pvc-wordpress.yaml
```
* **[manifests/pvc-wordpress.yaml](manifests/pvc-wordpress.yaml)** - The Persistent Volume Claim used for the WordPress
  pod's own internal storage.

4. Now create the WordPress deployment and its associated Service.
```
$ kubectl create \
    -f manifests/dep-wordpress.yaml \
    -f manifests/svc-wordpress.yaml
```

* **[manifests/dep-wordpress.yaml](manifests/dep-wordpress.yaml)** - WordPress deployment. The MySQL password is read
  from the secret and passed to MySQL as an environment variable.
* **[manifests/svc-wordpress.yaml](manifests/svc-wordpress.yaml)** - WordPress NodePort service

5. Wait for the Pods to be up and running:
```
$ kubectl get pods --watch
```

6. With both MySQL and WordPress up and running, use the `minikube service` command to access the WordPress deployment.
```
$ minikube service wordpress
```

At this point, you should see the WordPress default installation and configuration page. You can configure it and
give it a go!

---

## Clean Up

```
$ kubectl delete -f manifests/
$ kubectl delete pvc mysql-data-mysql-0
```

[wordpress]: https://wordpress.org/
[wordpress-chart]: https://github.com/helm/charts/tree/master/stable/wordpress