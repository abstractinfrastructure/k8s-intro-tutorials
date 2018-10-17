# Workloads

Workloads within Kubernetes are higher level objects that manage Pods or other higher level objects.

In **ALL CASES** a Pod Template is included, and acts as the base tier of management.

**Note:** 
1) If you are coming directly from the previous tutorials (core), you may still be configured to use the
`minidev` context. Switch to the `minikube` context before proceeding with the rest of the tutorials.

2) Unlike some of the other tutorials, the workload exercises should be cleaned up before moving onto the next
 workload type. The clean-up commands will included after **Summary** section of the exercise.

# Index
* [ReplicaSets](#replicasets)
  * [Exercise: Understanding ReplicaSets](#exercise-understanding-replicasets)
* [Deployments](#deployments)
  * [Exercise: Using Deployments](#exercise-using-deployments)
  * [Exercise: Rolling Back a Deployment](#exercise-rolling-back-a-deployment)
* [DaemonSets](#daemonsets)
  * [Exercise: Managing DaemonSets](#exercise-managing-daemonsets)
  * [Optional: Working with DaemonSet Revisions](#optional-working-with-daemonset-revisions)
* [StatefulSets](#statefulsets)
  * [Exercise: Managing StatefulSets](#exercise-managing-statefulsets)
  * [Exercise: Understanding StatefulSet Network Identity](#exercise-understanding-statefulset-network-identity)
* [Jobs and Cronjobs](#jobs-and-cronjobs)
  * [Exercise: Creating a Job](#exercise-creating-a-job)
  * [Exercise: Scheduling a CronJob](#exercise-scheduling-a-cronjob)
* [Helpful Resources](#helpful-resources)


---

# ReplicaSets
ReplicaSets are the primary method of managing Pod replicas and their lifecycle. This includes their scheduling,
scaling, and deletion.

Their job is simple, **always** ensure the desired number of `replicas` that match the selector are running.

---

### Exercise: Understanding ReplicaSets
**Objective:** Create and scale a ReplicaSet. Explore and gain an understanding of how the Pods are generated from
the Pod template, and how they are targeted with selectors.

---

1) Begin by creating a ReplicaSet called `rs-example` with `3` `replicas`, using the `nginx:stable-alpine` image and
configure the labels and selectors to target `app=nginx` and `env=prod`. The yaml block below or the manifest
`manifests/rs-example.yaml` may be used.

**manifests/rs-example.yaml**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: example-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      env: prod
  template:
    metadata:
      labels:
        app: nginx
        env: prod
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
```

**Command**
```
$ kubectl create -f manifests/rs-example.yaml
```

2) Watch as the newly created ReplicaSet provisions the Pods based off the Pod Template.
```
$ kubectl get pods --watch --show-labels
```
Note that the newly provisioned Pods are given a name based off the ReplicaSet name appended with a 5 character random
string. These Pods are labeled with the labels as specified in the manifest.

3) Scale ReplicaSet `rs-example` up to `5` replicas with the below command.
```
$ kubectl scale replicaset rs-example --replicas=5
```
**Tip:** `replicaset` can be substituted with `rs` when using `kubectl`.

4) Describe `rs-example` and take note of the `Replicas` and `Pod Status` field in addition to the `Events`.
```
$ kubectl describe rs rs-example
```

5) Now, using the `scale` command bring the replicas back down to `3`.
```
$ kubectl scale rs rs-example --replicas=3
```

6) Watch as the ReplicaSet Controller terminates 2 of the Pods to bring the cluster back into it's desired state of
3 replicas.
```
$ kubectl get pods --show-labels --watch
```

7) Once `rs-example` is back down to 3 Pods. Create an independent Pod manually with the same labels as the one
targeted by `rs-example` from the manifest `manifests/pod-rs-example.yaml`.

**manifests/pod-rs-example.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
  labels:
    app: nginx
    env: prod
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
```

**Command**
```
$ kubectl create -f manifests/pod-rs-example.yaml
```

8) Immediately watch the Pods.
```
$ kubectl get pods --show-labels --watch
```
Note that the Pod is created and immediately terminated.

9) Describe `rs-example` and look at the `events`.
```
$ kubectl describe rs rs-example
```
There will be an entry with `Deleted pod: pod-example`. This is because a ReplicaSet targets **ALL** Pods matching
the labels supplied in the selector.

---

**Summary:** ReplicaSets ensure a desired number of replicas matching the selector are present. They manage the
lifecycle of **ALL** matching Pods. If the desired number of replicas matching the selector currently exist when the
ReplicaSet is created, no new Pods will be created. If they are missing, then the ReplicaSet Controller will create
new Pods based off the Pod Template till the desired number of Replicas are present.

---

**Clean Up Command**
```
kubectl delete rs rs-example
```

---

[Back to Index](#index)

---
---

# Deployments
Deployments are a declarative method of managing Pods via ReplicaSets. They provide rollback functionality in addition
to more granular update control mechanisms.

---

### Exercise: Using Deployments
**Objective:** Create, update and scale a Deployment as well as explore the relationship of Deployment, ReplicaSet
and Pod.

---

1) Create a Deployment `deploy-example`. Configure it using the example yaml block below or use the manifest 
`manifests/deploy-example.yaml`. Additionally pass the `--record` flag to `kubectl` when you create the Deployment. 
The `--record` flag saves the command as an annotation, and it can be thought of similar to a git commit message.

**manifests/deployment-example.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-example
spec:
  replicas: 3
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
```

**Command**
```
$ kubectl create -f manifests/deploy-example.yaml --record
```

2) Check the status of the Deployment.
```
$ kubectl get deployments
```

3) Once the Deployment is ready, view the current ReplicaSets and be sure to show the labels.
```
$ kubectl get rs --show-labels
```
Note the name and `pod-template-hash` label of the newly created ReplicaSet. The created ReplicaSet's name will
include the `pod-template-hash`.

4) Describe the generated ReplicaSet.
```
$ kubectl describe rs deploy-example-<pod-template-hash>
```
Look at both the `Labels` and the `Selectors` fields. The `pod-template-hash` value has automatically been added to
both the Labels and Selector of the ReplicaSet. Then take note of the `Controlled By` field. This will reference the
direct parent object, and in this case the original `deploy-example` Deployment.

5) Now, get the Pods and pass the `--show-labels` flag.
```
$ kubectl get pods --show-labels
```
Just as with the ReplicaSet, the Pods name are labels include the `pod-template-hash`.

6) Describe one of the Pods.
```
$ kubectl describe pod deploy-example-<pod-template-hash-<random>
```
Look at the `Controlled By` field. It will contain a reference to the parent ReplicaSet, but not the parent Deployment.

Now that the relationship from Deployment to ReplicaSet to Pod is understood. It is time to update the
`deploy-example` and see an update in action.

7) Update the `deploy-example` manifest and add a few additional labels to the Pod template. Once done, apply the
change with the `--record` flag.
```
$ kubectl apply -f manifests/deploy-example.yaml --record
  < or >
$ kubectl edit deploy deploy-example --record
```
**Tip:** `deploy` can be substituted for `deployment` when using `kubectl`.

8) Immediately watch the Pods.
```
$ kubectl get pods --show-labels --watch
```
The old version of the Pods will be phased out one at a time and instances of the new version will take its place.
The way in which this is controlled is through the `strategy` stanza. For specific documentation this feature, see
the [Deployment Strategy Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy).

9) Now view the ReplicaSets.
```
$ kubectl get rs --show-labels
```
There will now be two ReplicaSets, with the previous version of the Deployment being scaled down to 0.

10) Now, scale the Deployment up as you would a ReplicaSet, and set the `replicas=5`.
```
$ kubectl scale deploy deploy-example --replicas=5
```

11) List the ReplicaSets.
```
$ kubectl get rs --show-labels
```
Note that there is **NO** new ReplicaSet generated. Scaling actions do **NOT** trigger a change in the Pod Template.

12) Just as before, describe the Deployment, ReplicaSet and one of the Pods. Note the `Events` and `Controlled By`
fields. It should present a clear picture of relationship between objects during an update of a Deployment.
```
$ kubectl describe deploy deploy-example
$ kubectl describe rs deploy-example-<pod-template-hash>
$ kubectl describe pod deploy-example-<pod-template-hash-<random>
```

---

**Summary:** Deployments are the main method of managing applications deployed within Kubernetes. They create and
supervise targeted ReplicaSets by generating a unique hash called the `pod-template-hash` and attaching it to child
objects as a Label along with automatically including it in their Selector. This method of managing rollouts along with
being able to define the methods and tolerances in the update strategy permits for a safe and seamless way of updating
an application in place.

---

### Exercise: Rolling Back a Deployment
**Objective:** Learn how to view the history of a Deployment and rollback to older revisions.

**Note:** This exercise builds off the previous exercise: [Using Deployments](#exercise-using-deployments). If you
have not, complete it first before continuing.

---

1) Use the `rollout` command to view the `history` of the Deployment `deploy-example`.
```
$ kubectl rollout history deployment deploy-example
```
There should be two revisions. One for when the Deployment was first created, and another when the additional Labels
were added. The number of revisions saved is based off of the `revisionHistoryLimit` attribute in the Deployment spec.

2) Look at the details of a specific revision by passing the `--revision=<revision number>` flag.
```
$ kubectl rollout history deployment deploy-example --revision=1
$ kubectl rollout history deployment deploy-example --revision=2
```
Viewing the specific revision will display a summary of the Pod Template.

3) Choose to go back to revision `1` by using the `rollout undo` command.
```
$ kubectl rollout undo deployment deploy-example --to-revision=1
```
**Tip:** The `--to-revision` flag can be omitted if you wish to just go back to the previous configuration.

4) Immediately watch the Pods.
```
$ kubectl get pods --show-labels --watch
```
They will cycle through rolling back to the previous revision.

5) Describe the Deployment `deploy-example`.
```
$ kubectl describe deployment deploy-example
```
The events will describe the scaling back of the previous and switching over to the desired revision.

---

**Summary:** Understanding how to use `rollout` command to both get a diff of the different revisions as well as
be able to roll-back to a previously known good configuration is an important aspect of Deployments that cannot
be left out.

---

**Clean Up Command**
```
kubectl delete deploy deploy-example
```

---

[Back to Index](#index)

---
---

# DaemonSets

DaemonSets ensure that all nodes matching certain criteria will run an instance of the supplied Pod.

They bypass default scheduling mechanisms and restrictions, and are ideal for cluster wide services such as
log forwarding, or health monitoring.

---

### Exercise: Managing DaemonSets
**Objective:** Experience creating, updating, and rolling back a DaemonSet. Additionally delve into the process of
how they are scheduled and how an update occurs.

---

1) Create DaemonSet `ds-example` and pass the `--record` flag. Use the example yaml block below as a base, or use
the manifest `manifests/ds-example.yaml` directly.

**manifests/ds-example.yaml**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-example
spec:
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        nodeType: edge
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
```

**Command**
```
$ kubectl create -f manifests/ds-example.yaml --record
```

2) View the current DaemonSets.
```
$ kubectl get daemonset
```
As there are no matching nodes, no Pods should be scheduled.

3) Label the `minikube` node with `nodeType=edge`
```
$ kubectl label node minikube nodeType=edge
```

4) View the current DaemonSets once again.
```
$ kubectl get daemonsets
```
There should now be a single instance of the DaemonSet `ds-example` deployed.

5) View the current Pods and display their labels with `--show-labels`.
```
$ kubectl get pods --show-labels
```
Note that the deployed Pod has a `controller-revision-hash` label. This is used like the `pod-template-hash` in a
Deployment to track and allow for rollback functionality.

6) Describing the DaemonSet will provide you with status information regarding it's Deployment cluster wide.
```
$ kubectl describe ds ds-example
```
**Tip:** `ds` can be substituted for `daemonset` when using `kubectl`.

7) Update the DaemonSet by adding a few additional labels to the Pod Template and use the `--record` flag.
```
$ kubectl apply -f manifests/ds-example.yaml --record
  < or >
$ kubectl edit ds ds-example --record
```

8) Watch the Pods and be sure to show the labels.
```
$ kubectl get pods --show-labels --watch
```
The old version of the DaemonSet will be phased out one at a time and instances of the new version will take its
place. Similar to Deployments, DaemonSets have their own equivalent to a Deployment's `strategy` in the form of
`updateStrategy`. The defaults are generally suitable, but other tuning options may be set. For reference, see the
[Updating DaemonSet Documentation](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/#performing-a-rolling-update).

---

**Summary:** DaemonSets are usually used for important cluster-wide support services such as Pod Networking, Logging,
or Monitoring. They differ from other workloads in that their scheduling bypasses normal mechanisms, and is centered
around node placement. Like Deployments, they have their own `pod-template-hash` in the form of
`controller-revision-hash` used for keeping track of Pod Template revisions and enabling rollback functionality.

---

### Optional: Working with DaemonSet Revisions

**Objective:** Explore using the `rollout` command to rollback to a specific version of a DaemonSet.

**Note:** This exercise is functionally identical to the Exercise[Rolling Back a Deployment](#exercise-rolling-back-deployment).
If you have completed that exercise, then this may be considered optional. Additionally, this exercise builds off
the previous exercise [Managing DaemonSets](#exercise-managing-daemonsets) and it must be completed before continuing.

---

1) Use the `rollout` command to view the `history` of the DaemonSet `ds-example`
```
$ kubectl rollout history ds ds-example
```
There should be two revisions. One for when the Deployment was first created, and another when the additional Labels
were added. The number of revisions saved is based off of the `revisionHistoryLimit` attribute in the DaemonSet spec.

2) Look at the details of a specific revision by passing the `--revision=<revision number>` flag.
```
$ kubectl rollout history ds ds-example --revision=1
$ kubectl rollout history ds ds-example --revision=2
```
Viewing the specific revision will display the Pod Template.

3) Choose to go back to revision `1` by using the `rollout undo` command.
```
$ kubectl rollout undo ds ds-example --to-revision=1
```
**Tip:** The `--to-revision` flag can be omitted if you wish to just go back to the previous configuration.

4) Immediately watch the Pods.
```
$ kubectl get pods --show-labels --watch
```
They will cycle through rolling back to the previous revision.

5) Describe the DaemonSet `ds-example`.
```
$ kubectl describe ds ds-example
```
The events will be sparse with a single host, however in an actual Deployment they will describe the status of
updating the DaemonSet cluster wide, cycling through hosts one-by-one.

---

**Summary:** Being able to use the `rollout` command with DaemonSets is import in scenarios where one may have
to quickly go back to a previously known-good version. This becomes even more important for 'infrastructure' like
services such as Pod Networking.

---

**Clean Up Command**
```
kubectl delete ds ds-example
```

---

[Back to Index](#index)

---
---

# StatefulSets
The StatefulSet controller is tailored to managing Pods that must persist or maintain state. Pod identity including
hostname, network, and storage can be considered **persistent**.

They ensure persistence by making use of three things:
* The StatefulSet controller enforcing predicable naming, and ordered provisioning/updating/deletion.
* A headless service to provide a unique network identity.
* A volume template to ensure stable per-instance storage.
---

### Exercise: Managing StatefulSets
**Objective:** Create, update, and delete a `StatefulSet` to gain an understanding of how the StatefulSet lifecycle
differs from other workloads with regards to updating, deleting and the provisioning of storage.

---

1) Create StatefulSet `sts-example` using the yaml block below or the manifest `manifests/sts-example.yaml`.

**manifests/sts-example.yaml**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-example
spec:
  replicas: 3
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: stateful
  serviceName: app
  updateStrategy:
    type: OnDelete
  template:
    metadata:
      labels:
        app: stateful
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi
```

**Command**
```
$ kubectl create -f manifests/sts-example.yaml
```

2) Immediately watch the Pods being created.
```
$ kubectl get pods --show-labels --watch
```
Unlike Deployments or DaemonSets, the Pods of a StatefulSet are created one-by-one, going by their ordinal index.
Meaning, `sts-example-0` will fully be provisioned before `sts-example-1` starts up. Additionally, take notice of
the `controller-revision-hash` label. This serves the same purpose as the `controller-revision-hash` label in a
DaemonSet or the `pod-template-hash` in a Deployment. It provides a means of tracking the revision of the Pod
Template and enables rollback functionality.

3) More information on the StatefulSet can be gleaned about the state of the StatefulSet by describing it.
```
$ kubectl describe statefulset sts-example
```
Within the events, notice that it is creating claims for volumes before each Pod is created.

4) View the current Persistent Volume Claims.
```
$ kubectl get pvc
```
The StatefulSet controller creates a volume for each instance based off the `volumeClaimTemplate`. It prepends
the volume name to the Pod name. e.g. `www-sts-example-0`.

5) Update the StatefulSet's Pod Template and add a few additional labels.
```
$ kubectl apply -f manifests/sts-example.yaml --record
  < or >
$ kubectl edit statefulset sts-example --record
```

6) Return to watching the Pods.
```
$ kubectl get pods --show-labels
```
None of the Pods are being updated to the new version of the Pod.

7) Delete the `sts-example-2` Pod.
```
$ kubectl delete pod sts-example-2
```

8) Immediately get the Pods.
```
$ kubectl get pods --show-labels --watch
```
The new `sts-example-2` Pod should be created with the new additional labels. The `OnDelete` Update Strategy will
not spawn a new iteration of the Pod until the previous one was **deleted**. This allows for manual gating the
update process for the StatefulSet.

9) Update the StatefulSet and change the Update Strategy Type to `RollingUpdate`.
```
$ kubectl apply -f manifests/sts-example.yaml --record
  < or >
$ kubectl edit statefulset sts-example --record
```

10) Immediately watch the Pods once again.
```
$ kubectl get pods --show-labels --watch
```
Note that the Pods are sequentially updated in descending order, or largest to smallest based on the
Pod's ordinal index. This means that if `sts-example-2` was not updated already, it would be updated first, then
`sts-example-1` and finally `sts-example-0`.

11) Delete the StatefulSet `sts-example`
```
$ kubectl delete statefulset sts-example
```

12) View the Persistent Volume Claims.
```
$ kubectl get pvc
```
Created PVCs are **NOT** garbage collected automatically when a StatefulSet is deleted. They must be reclaimed
independently of the StatefulSet itself.

13) Recreate the StatefulSet using the same manifest.
```
$ kubectl create -f manifests/sts-example.yaml --record
```

14) View the Persistent Volume Claims again.
```
$ kubectl get pvc
```
Note that new PVCs were **NOT** provisioned. The StatefulSet controller assumes if the matching name is present,
that PVC is intended to be used for the associated Pod.

---

**Summary:** Like many applications where state must be taken into account, the planning and usage of StatefulSets
requires forethought. The consistency brought by standard naming, ordered updates/deletes and templated storage
does however make this task easier.

---

### Exercise: Understanding StatefulSet Network Identity

**Objective:** Create a _"headless service"_ or a service without a `ClusterIP` (`ClusterIP=None`) for use with the
StatefulSet `sts-example`, then explore how this enables consistent service discovery.

---

1) Create the headless service `app` using the `app=stateful` selector from the yaml below or the manifest
`manifests/service-sts-example.yaml`.

**manifests/service-sts-example.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  clusterIP: None
  selector:
    app: stateful
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Command**
```
$ kubectl create -f manifests/service-sts-example.yaml
```

2) Describe the newly created service
```
$ kubectl describe svc app
```
Notice that it does not have a `clusterIP`, but does have the Pod Endpoints listed. Headless services are unique
in this behavior.

3) Query the DNS entry for the `app` service.
```
$ kubectl exec sts-example-0 -- nslookup app.default.svc.cluster.local
```
An A record will have been returned for each instance of the StatefulSet. Querying the service directly will do
simple DNS round-robin load-balancing.

4) Finally, query one of instances directly.
```
$ kubectl exec sts-example-0 -- nslookup sts-example-1.app.default.svc.cluster.local
```
This is a unique feature to StatefulSets. This allows for services to directly interact with a specific instance
of a Pod. If the Pod is updated and obtains a new IP, the DNS record will immediately point to it enabling consistent
service discovery.

---

**Summary:** StatefulSet service discovery is unique within Kubernetes in that it augments a headless service
(A service without a unique `ClusterIP`) to provide a consistent mapping to the individual Pods. These mappings
take the form of an A record in format of: `<StatefulSet Name>-<ordinal>.<service name>.<namespace>.svc.cluster.local`
and can be used consistently throughout other Workloads.

---

**Clean Up Command**
```
kubectl delete svc app
kubectl delete statefulset sts-example
kubectl delete pvc www-sts-example-0 www-sts-example-1 www-sts-example-2
```

---

[Back to Index](#index)

---
---

# Jobs and CronJobs
The Job Controller ensures one or more Pods are executed and successfully terminate. Essentially a task executor
that can be run in parallel.

CronJobs are an extension of the Job Controller, and enable Jobs to be run on a schedule.

---

### Exercise: Creating a Job
**Objective:** Create a Kubernetes `Job` and work to understand how the Pods are managed with `completions` and
`parallelism` directives.

---

1) Create job `job-example` using the yaml below, or the manifest located at `manifests/job-example.yaml`

**manifests/job-example.yaml**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-example
spec:
  backoffLimit: 4
  completions: 4
  parallelism: 2
  template:
    spec:
      containers:
      - name: hello
        image: alpine:latest
        command: ["/bin/sh", "-c"]
        args: ["echo hello from $HOSTNAME!"]
      restartPolicy: Never
```

**Command**
```
$ kubectl create -f manifests/job-example.yaml
```

2) Watch the Pods as they are being created.
```
$ kubectl get pods --show-labels --watch
```
Only two Pods are being provisioned at a time; adhering to the `parallelism` attribute. This is done until the total
number of `completions` is satisfied. Additionally, the Pods are labeled with `controller-uid`, this acts as a
unique ID for that specific Job. 

When done, the Pods persist in a `Completed` state. They are not deleted after the Job is completed or failed. 
This is intentional to better support troubleshooting.

3) A summary of these events can be seen by describing the Job itself.
```
$ kubectl describe job job-example
```

4) Delete the job.
```
$ kubectl delete job job-example
```

5) View the Pods once more.
```
$ kubectl get pods
```
The Pods will now be deleted. They are cleaned up when the Job itself is removed.

---

**Summary:** Jobs are fire and forget one off tasks, batch processing or as an executor for a workflow engine.
They _"run to completion"_ or terminate gracefully adhering to the `completions` and `parallelism` directives.

---

### Exercise: Scheduling a CronJob
**Objective:** Create a CronJob based off a Job Template. Understand how the Jobs are generated and how to suspend
a job in the event of a problem.

---

1) Create CronJob `cronjob-example` based off the yaml below, or use the manifest `manifests/cronjob-example.yaml`
It is configured to run the Job from the earlier example every minute, using the cron schedule `"*/1 * * * *"`.
This schedule is **UTC ONLY**.

**manifests/cronjob-example.yaml**
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-example
spec:
  schedule: "*/1 * * * *"
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      completions: 4
      parallelism: 2
      template:
        spec:
          containers:
          - name: hello
            image: alpine:latest
            command: ["/bin/sh", "-c"]
            args: ["echo hello from $HOSTNAME!"]
          restartPolicy: Never
```

**Command**
```
$ kubectl create -f manifests/cronjob-example.yaml
```

2) Give it some time to run, and then list the Jobs.
```
$ kubectl get jobs
```
There should be at least one Job named in the format `<cronjob-name>-<unix time stamp>`. Note the timestamp of
the oldest Job.

3) Give it a few minutes and list the Jobs once again
```
$ kubectl get jobs
```
The oldest Job should have been removed. The CronJob controller will purge Jobs according to the
`successfulJobHistoryLimit` and `failedJobHistoryLimit` attributes. In this case, it is retaining strictly the
last 3 successful Jobs.

4) Describe the CronJob `cronjob-example` 
```
$ kubectl describe CronJob cronjob-example
```
The events will show the records of the creation and deletion of the Jobs.

5) Edit the CronJob `cronjob-example` and locate the `Suspend` field. Then set it to true.
```
$ kubectl edit CronJob cronjob-example
```
This will prevent the cronjob from firing off any future events, and is useful to do to initially troubleshoot
an issue without having to delete the CronJob directly.


5) Delete the CronJob
```
$ kubectl delete cronjob cronjob-example
```
Deleting the CronJob **WILL** delete all child Jobs. Use `Suspend` to _'stop'_ the Job temporarily if attempting
to troubleshoot.

---

**Summary:** CronJobs are a useful extension of Jobs. They are great for backup or other day-to-day tasks, with the
only caveat being they adhere to a **UTC ONLY** schedule.

---

**Clean Up Commands**
```
kubectl delete CronJob cronjob-example
```

---

[Back to Index](#index)

---
---

# Helpful Resources

* [Deployment Overview](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [DaemonSet Overview](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
* [StatefulSet Basics](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)
* [StatefulSet Overview](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [Job Overview](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)

---

[Back to Index](#index)
