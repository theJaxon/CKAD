# CKAD
Preparation for the Certified Kubernetes Application Developer V1.19 [CNCF] exam

---

### Objectives:

<details>
<summary>1- Pod Design <b>20%</b></summary>
<p>

1. Deployments, Rolling updates and Rollbacks
2. Jobs and CronJobs
3. Labels, Selectors and annotations
</p>
</details>

<details>
<summary>2- Configuration <b>18%</b></summary>
<p>

1. ConfigMaps
2. SecurityContexts
3. Resource requirements
4. Secrets
5. ServiceAccounts
</p>
</details>

<details>
<summary>3- Observability <b>18%</b></summary>
<p>

1. Probes (Liveness, Readiness) [#Liveness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
2. Container Logging
3. Application Monitoring
4. Debugging

</p>
</details>

<details>
<summary>4- Core concepts <b>13%</b></summary>
<p>

1. API primitives `Objects`
2. Basic pod creation and configuration

</p>
</details>

<details>
<summary>5- Services & Networking <b>13%</b></summary>
<p>

* Understand Services & Network Policies

</p>
</details>

<details>
<summary>6- Multi container pods <b>10%</b></summary>
<p>

* Ambassador
* Adapter
* Sidecar 

</p>
</details>

<details>
<summary>7- State Persistence <b>8%</b></summary>
<p>

* PVs & PVCs

</p>
</details>

---

### Tips:

1. Enable auto completion
```bash
source <(kubectl completion bash)

# To do it persistently store it in .bashrc
echo "source <(kubectl completion bash)" >> ~/.bashrc

# Handking k autocompletion https://github.com/kubernetes/kubectl/issues/120#issuecomment-344137671
source <(kubectl completion bash | sed 's/kubectl/k/g') 

# alias kubectl to k (from https://github.com/krzko/awesome-cka)
alias k='kubectl'
alias kc='kubectl create'
alias kg="kubectl get"
alias kr="kubectl run"
alias kx="kubectl explain"

k completion -h # In case you forgot the source command listed above
```

Also check the [cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) from the documentation for easier access 

2. [Delete](https://stackoverflow.com/a/33510531) all pods in a NS
```bash
kubectl delete --all po --namespace=<name>

# Don't wait for K8s to gracefully delete the object, force delete with grace period zero 
k delete po --all --grace-period=0 --force
```

3. kubectl 
```bash
kubectl <verb> --help | less

kubectl | less

# Get a list of objects shortnames
kubectl api-resources

kubectl get po -Ao wide
kubectl get po -A --show-labels 
kubectl get po -A -L <label>

# Sorting 
# Sort by the creation time 
kubectl  get po -n kube-system --sort-by='{.metadata.creationTimestamp}'

# Check Authorization
kubectl auth can-i <verb> <object> # ex: kubectl auth can-i get namsepace

kubectl auth can-i <verb> <object> as <username>

# Get a shell into the container running
kubectl exec --it <name> -- /bin/bash

# Proxy 
kubectl proxy --port=<number> &
# Display API Groups after using the proxy command 
curl http://localhost:<number>

# Narrow down a selection from the shown API Groups 
curl http://localhost:<number>/version

# Get list of pods in a specific NS
curl http://localhost:<number>/api/v1/namespaces/<Namespace>/pods


# Change context 
k config use-context <context> 

# Switch to a different NS in a context 
k config set-context <context> --namespace=<namespace>
``` 

4. Watch command (2 ways)
* `kubectl get` command supports the `-w` option.
* watch -n 1 `kubectl get po` specifies a watch command that gets updated each 1 second  

5. [Delete a block](https://vi.stackexchange.com/questions/1915/how-do-i-delete-a-large-block-of-text-without-counting-the-lines) from the YAML file using Vim
```bash
# Approach 1 
m then a # Start section 
d then ' then a # End section
```
```bash
# Approach 2 Visual mode 
shift and v (capital V) # Start section 
d # End section
```

6. Use the provided examples to quickly create yaml files
```bash
kubectl create <object> --help | less
kubectl create <object> <name> --image=<image-name> --dry-run=client -o yaml > file.yml

kubectl create cj --help | less # This shows the following example
# kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *" 

# Dry run and redirect output as yaml
kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml > cj.yml

# Same procedure with deployment
kubectl create deployment --help | less
# kubectl create deployment my-dep --image=busybox
kubectl create deployment my-dep --image=busybox --dry-run=client -o yaml > deployment.yml

# Create configMap
kubectl create cm --help | less

# Services are created with expose command
kubectl expose <object> --port=<number> --targetPort=<number>

# Pods
kubectl run <name> --image=<name>

# Run pod with specific service account 
k run <name> --image=<name> --serviceaccount=<sa-name>

# Pod with requests and limits defined
kubectl run <name> --image=<name> --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'

# Create a pod and expose it in the same command
kubectl run <name> --image=<name> --port=<number> --expose
```

7. Use kubectl explain to find the `apiVersion`
```bash
kubectl explain pod 

# similar but 2 step approach

kubectl api-resources | less # /<object>

kubectl api-versions | less # /<term from previous command>
```
---

### :file_folder: Important dirs:
* Kubectl 
  * ~/.kube/config # kubectl config view
* The controller plane is generated using files at `/etc/kubernetes/manifests/` which contains 
  * etcd.yaml
  * kube-apiserver.yaml
  * kube-controller-manager.yaml
  * kube-scheduler.yaml

### JSONPATH commands:
```bash
# Get all pod names 
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'

# Get container names inside each pod
k get po -l key=value -o jsonpath='{.items[0].status.containerStatuses[*].name}'

# Get Pod IP a 
k get po <name> -o jsonpath='{.status.podIP}'

# Get available Memory on the nodes
k get nodes -o jsonpath='{.items[].status.allocatable.memory}'

# Get available CPUs
k get nodes -o jsonpath='{.items.status.allocatable.cpus}'

```

#### :gem: Common commands:

```bash
kubectl cluster-info

# List kubernetes objects 
kubectl api-resources 

kubectl get <object>

# Copy a file to a pod 
k cp /file/path/file_name <pod-name>:/path/inside/pod

# Troubleshooting commands
kubectl describe <object> # Used to also view resource requests and limits

# Use -C to render lines before and after the matched grep expression 
k describe <object> | grep <term> -C 5

kubectl logs <pod-name>

# Get logs of the previous pod instance (if one existed)
kubectl logs <pod-name> -p

# Follow the logs using -f 
kubectl logs <pod-name> -f

# When having more than 1 container in a pod use the -c flag 
kubectl logs <pod> -c <container>

# Check events of specific object
# Found in events in stackdriver docs 
kubectl get events --field-selector involvedObject.name=<object-name>

# Get All pods in the cluster
kubectl get po -A

# Updating existing object definition
kubectl apply -f file_name.yml
kubectl edit <object> <name>
kubectl exec -it <pod-name> -- command-here # ls /etc/conf for example
kubectl exec --it <pod-name> -c <container-name> -- /bin/bash # Getting into multi-containter pod

```

* `kubectl describe` command gets the current state of the object from `etcd` 
* `kubectl logs` always go to a pod so no need to mention the type

---

#### Containers:

* Use Namespaces, the linux kernel allows for strict isolation between different system components using NSs, isolation is as follows:
  * File (a container is like a `chroot` environment and only contents of specific directories can be viewed)
  * IPCs
  * network (every container can do its own networking)
  * processes (Container can only see its own processes)
  * users (Containers have their own users that don't exist on the host OS)
* They also use `CGroups` which offers resource allocation and limition 

The containers section helps identifying whether it's an `init Container` or a normal one, `imagePullSecrets` can be used to download from private registeries.

---

#### Pods:
- Each pod has a unique IP address
- Containers within this pod share the same IP address
- Use `port forwarding` to access pods via **localhost** (For testing purposes only)
* Usually SVC or Ingress (DNS) is used for that purpose
```bash
kubectl port-forward pod/<name> <host-port>:<container-port>
# Verify that kubectl is listening 
ss -tunap | grep <host-port>
curl localhost:<host-port>
```

#### :clock1: Pod Lifecycle (Status):

| Status            | Meaning                                                                                                                    |
|-------------------|----------------------------------------------------------------------------------------------------------------------------|
| Pending           | When a pod is first created it's in a pending state which means that the scheduler is figuring out where to place the pod  |
| containerCreating | Images are being pulled so that the container starts                                                                       |
| Running           | The pod is started, it keeps running till it does its job and then terminates                                              |

* pod.spec
  * .containers
    * .resources 
    * .securityContext
    * .env # Load a single env variable
      * .name
      * .valueFrom
        * .configMapKeyRef
        * .secretKeyRef
    * .envFrom # Takes an array
      * .configMapRef.name
      * .secretRef.name
    * .volumeMounts
      * .mountPath (what you want to store from the container on the host)
  * .volumes
    * .emptyDir (creates temporary directory on the host)
    * .hostPath (Mounts a directory located on the host to the container)
    * ~~.gitrepo~~ # Deprecated
    *  .azureDisk, .awsElasticBlockStore, .gcePresistentDisk # Cloud and all
    * .cephfs # ceph object storage
    * .nfs
    * .persistentVolumeClaim
      * .claimName (Name of the PVC)
    * configMap
      * .name
      * .items
        * key 
        * path
    * .secret.secretName

```bash
# Init containers
k explain pod.spec.initContainers
```

* The `Scheduling` fields helps determine where the pod will be scheduled
  * **nodeSelector** schedules on a target node using provided labels
  * **nodeName** schedules on a target node using just the node name
  * `Tolerations` allow the pod to be scheduled on a node with a specific `taint` (The pod becomes tolerant to that node)
  * `Affinity` similar to `nodeSelector` or `nodeName` but with the capability of having conditionals to what nodes the pod can be scheduled on
    * `NodeAffinity` similar to `nodeSelector` where we schedule using a label that matches
    * `podAffinity` schedules the pod on a node where some other specific pods are already running
    * `PodAntiAffinity` prevents the scheduling of a pod on a node containing some other specific pod
  * [`priorityClassName`](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#example-priorityclass) specifies pod priority where if the pod can't be scheduled then the scheduler evict lower prioroty pods to allow the scheduling of this pod to happen
```bash
k create priorityclass --help
```
* `hostNetwork` controls whether the pod may use the node network NS (is set to False by default)


##### ENV value Types:
* Plain Key Value
* ConfigMap
* Secrets
* fieldRef

```bash
# Create a pod with an environment variable
# 1- Plain key value type 
k run <pod-name> --image=busybox --o yaml --dry-run=client --env=EXAM=CKAD --command env
```

* When a ConfigMap is created `--from-env-file` the file must adhere to the format `key=value`
* `--from-file`

Using single configMap env variable:

```yml
env:
- name:
  valueFrom:
    configMapKeyRef:
      name: <configMap-object-name>
      key: <key-from-cm>
```
Using single Secret env variable:
```yml
env:
- name: <name>
  valueFrom:
    secretKeyRef:
      name: <secret-name>
      key: <key-name>
```

* Note that if CM or Secret weren't found and they're being referenced then the object creation will fail, to mitigate that make them optional.
Option CM example:
```yml
env:
- name: <name>
  valueFrom:
    configMapKeyRef:
      name: <cm-name>
      key: <key-name>
      optional: True 
```

FieldRef example:
```yml
env: 
- name: POD_NAME
  valueFrom:
    fieldRef:
      fieldPath: metadata.name
- name: NS 
  valueFrom:
    fieldRef:
      fieldPath: metadata.namespace
- name: IP_A
  valueFrom:
    fieldRef:
      fieldPath: status.podIP
```

---

#### Namespaces:
```bash
# Create ns
kubectl create ns <name>
```

#### Contexts:
```bash
# Change the default namespace in ~/.kube/config 
k config set-context --current --namespace <name>

# Verify using 
k config view

# Revert back to default ns 
k config set-context --current --namespace=
```

##### :gem: Namespace List:

|       Name      |                                                             Description                                                             |
|:---------------:|:-----------------------------------------------------------------------------------------------------------------------------------:|
|     default     |                                      Used by default when no namespace for objects is specified                                     |
|   kube-system   |                                                 NS for objects created by K8s system                                                |
|   kube-public   | - Created automatically and is readable by all users. - Contains info like CA that helps kubeadm join and authenticate worker nodes |
| kube-node-lease |                                    Contains lease objects that are used to determine node health                                    |

---


#### ConfigMaps and [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/):
- Special type of volumes
- Just like volumes, they must be created before the pod that will use them
- Used to separate dynamic data from static data in the pod
- ConfigMaps can be mounted on the location where the application expects to find a configuration file
- ConfigMaps can be included either as a `env` variable or as a `volume`
- When mounting a ConfigMap as a volume, in the mount point files will be created with the name of the Key and inside it exists the value
- `Secrets` are just encoded ConfigMaps
- Secrets allow stroing sensitive data as passwords, auth tokens and SSH keys
- Secrets are used so that the data doesn't have to be put in a pod this reducing risk of accidential exposure
- There are 3 types of secrets:
  - docker-registry which is used for connecting to the docker registry :open_mouth:
  - TLS which creates a TLS secret
  - **generic which creates a secret from a local file, a directory or a literal value**
- Secrets can be used by pods in 2 ways:
  - as environemnt variables
  - Mounted as volumes
- ConfigMaps can be created from different sources
  - Directories: uses multiple files in a directory 
  - Files: puts the contents of a file in ConfigMap
  - Literal Values: provide variables and command arguments that are to be used by a pod

* The behavior of `--from-file` is that if no `key=<file-name` was specified then the file-name itself is set as the key name.
* if a key was passed then it overrides the file_name that was going to be set by default as the key

```bash
# Command line ConfigMap specification 
kubectl create cm --from-literal='key=value' --from-literal='key2=value2'

# From a file 
kubectl create cm --from-file <file_name>

# From a file but the key is set to the desired name we want
kubectl create cm --from-file=desired_name=<file_name>

# Create a secret from file 
kubectl create secret <type> <name> --from-file=/absolute/path

# Create a secret from literal type=generic
kubectl create secret <type> <name> --from-literal='key=encoded_value'
```

Secrets can also be created from YAML files but the values need to be `base64` encrypted

```bash
echo <string> | base64 # encode the string

echo <base-64-encrypted-string> | base64 -d # decode

```

file_name  ex:
```ini
VAR1=abc
VAR2=efg
```

.data.variables

---

#### SecurityContext:
* Define privilege and access control settings for **pods** and **containers**
* They're part of the Pod specification
* They can be specified either at pod level or at container level (although `capabilites` is only available at container level)
* Running as privileged or unprivileged user
  * Without a securityContext, the user running the commands in the container inside the pod will be `root` by default
* SELinux where security labels can be applied OR AppArmor
* Discretionary access control which is about permissions used to access an object
* `AllowPrivilegeEscalation` which controls if a process can gain more privileges than its parent process

```bash
kubectl explain pod.spec.securityContext

k explain pod.spec.containers.securityContext
```

* runAsUser # Specifies the user of the running processes in the containers
* runAsGroup # Specifies the primary group of all processes within the containers 
* fsGroup # Applies settings to volumes that support ownership management, these volumes are modified to be owned by the GID specified in the fsGroup

* pod.spec.containers.securityContext
  * .runAsUser
  * .runAsGroup
  * .runAsNonRoot
  * .fsGroup
  * .capabilities

Capabilities example:
```yml
containers:
- name: nginx
  image: nginx
  securityContext:
    capabilities:
      add: ["SYS_TIME"]
```

---

#### Resource requirements:
- Define container's memory and cpu requirements using `requests` and `limits`
- `resource request` is the amount of resources necessary to run a container (The node must have enough available resources for the requests)
- `resource limits` is the maximum value for the resource usage of a container
- CPU core usage can be specified using `m` which means `milli CPU` or `milli core`
  - **250m** means a **quarter** of a cpu core 
  - **500m** means **half** of a cpu core and so on

```bash
# Set requests and limits for a running nginx pod
k set resources po/nginx --requests=memory=50Mi,cpu=200m --limits=memory=256Mi,cpu=750m
```

---

#### Resource Quota:
- Used to limit resources in a `Namespace`

```bash
k create quota --help

kubectl create quota <name> --hard=<comma separated list of hardware specs>

kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2
```

* ResourceQuota
  * .metadata.**namespace**
  * .spec
    * .hard
      * .pods # Max number of pod on this ns
      * .requests.cpu
      * .requests.memory 
      * .limits.cpu
      * .limits.memory

---

#### Secrets:
* Store sensitive data in a secure fashion (using base64 encoding lol)

```bash
k create secret generic <name> --from-literal=key=value
```

---

#### ServiceAccount:
* There are 2 types of accounts in K8s, a `user` account and a `service` account.
* User accounts are used by individuals (admin, developer, etc) while service accounts are used by the pods (Prometheus uses a service account to get performance metrics using the API, Jenkins also uses a service account to deploy apps on the cluster)
* A service account provides an identity for processes that run in a Pod
* Allows the container to interact with kubernetes API
* Image pull secrets authenticate with private container registeries
* When a `sa` is created it also generates a token which is stored as a secret

```bash
kubectl create serviceaccount <name>

# Shorter version
k create sa <name>

# Use the created SA with a pod 
kubectl run <pod-name> --image=<name> --serviceaccounts=<SA-name>
```

just specify the `serviceAccountName` in the pod `spec` definition

---

#### Multi container pod:
Containers in the same pod can communicate with each other in 3 different ways:
1. Through the shared network (http://localhost:containerPort)
2. Through the use of a shared storage volume
3. Throguh shared process namespace `shareProcessNamespace: true` in the pod spec

Multi container design patterns:
1. Sidecar pod 
The sidecar container enhances the functionality of the main container.
use cases:
  * logging agents that collect and ship logs to a central aggregation system
  * File sync services and watchers - ex: [Git sync](https://github.com/kubernetes/git-sync) to a web server container
2. Ambassador pod
It receives network traffic then pass it to the main container (ex: ambassador listens on a custom port then forwards traffic to the main hard-coded port)
use cases - commonly used with DBs

3. Adapter pod
Changes the output of the main container (for example the output is changed to fit some sort of a monitoring tool that will receive it like prometheus or logstash)

* In Multi container pod, the same namespace can be shared between containers using **`shareProcessNamespace: True`** in pod spec, this can be useful in certain use cases where we need access for inter process communication or to fetch metrics about app processes in the container.

---

#### Init Containers:
* An init container is an additional container in a pod that completes a task before the main container is started
* The main container gets started only after the init container has finished its job

#### Probes:
- Allows customizing how K8s determines container status
- Liveness probe indicates whether the container is running properly and decides whether the cluster will automatically stop or restart the container
  - It just does health checks
- Readiness probe indicates whether the container is ready to receive requests (It makes sure that the pod is not published as available until it's able to access it)
- If the Readiness probe fails the pod doesn't get replaced, the pod will just stop receiving traffic and that's all.
- If Liveness probe fails then the pod is restarted

Types of probes:
- exec: a command that gets executed and returns a zero exit value
- httpGet: an HTTP request that returns a response code between 200 and 399
- tcpSocket: connectivity to a TCP socket (available port) is successfull

```
* pod.spec.containers.readinessProbe
  * .periodSeconds
  * .initialDelaySeconds
  * .exec
    * .command
```

```
* pod.spec.containers.livenessProbe
  * .failureThreshold # Minimum consecutive failures for the probe to be considered failed.
```

readinessProbe:
1. httpGet
```yml
readinessProbe:
  httpGet:
    port: <number>
    path: </path/here>
```
2. exec
```yml
readinessProbe:
  exec:
    command:
    - cat
    - </path/here>
```
3. tcpSocket
```yml
readinessProbe:
  tcpSocket:
    port: <number>
```

---

##### Metrics server installation:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml

# Verify server installation
kubectl get --raw /apis/metrics.k8s.io/

```

* This didn't work, the [soultion](https://github.com/kubernetes-sigs/metrics-server/issues/300) was to add 
```
- --kubelet-insecure-tls=true
- --kubelet-preferred-address-types=InternalIP
```
in the deployment section
---

#### Monitoring apps:

* `Metric server` comes pre-installed so no worries, just know the commands: 

```bash
kubectl top po

# Narrow down to a specific pod
kubectl top po <name>

kubectl top nodes
```

---

#### Debugging:
1. Check the status of `kubelet.service` using `systemctl status kubelet`
2. Check the logs of the `kubelet` service using `journalctl -u kubelet` 

---

#### Labels, Selectors and Annotations:
```bash
# Get pods based on specific label
kubectl get po -l key=value 

# The previous command is eqivalent to 
k get po --selector k=v

# Get pods based on multiple labels 
kubectl get po -l key=value,key=value

# Get pods that doesn't match the labels 
kubectl get po -l key!=value

# Set based selectors 
kubectl get po -l 'key in (val1,val2,val3)'

# Watch rollout and changes in the deployment 
kubectl get po -w

# To label an object use the label command
kubectl label <object> <name> key=value

# Remove label
kubectl label po <name> key-

# Add anootation 
kubectl annotate <object> <name> key=value
```

* The `selector` option is used to search for items that have a specific label set `kubectl get po --selector='key=value'`
* The previous command is also equivalent to using `kubectl get po -l key=value` 
* Annotations provide metadata ex: licences, maintainer etc
---

#### Annotations [Metadata]:
* Useful when dealing with external tools for automation where we need attach additional info to our pods
* Just add them in `metadata` section using `annotations:` then add key value pairs

---

#### Deployments:

* Represent multiple replicas of a Pod
* The `metadata` section of the deployment template doesn't need a name because k8s automatically generates a name for each replica 
* Any major change results in the deployment creating a new RS that uses new properties
* The old RS is kept but the number of replicas is set to 0 (Useful for rollbacks)
* Max number of pods = number of replicas + number specified in maxSurge

```bash
# Scale deployment using kubectl 
kubectl scale deployment <name> --replicas=<number>

# Can also be done via kubectl edit 
kubectl edit deployment <name>

# See rollout history of a deployment 
kubectl rollout history <object> <name>

# Check details about specific rollout revision
kubectl rollout history <object> <name> --revision=<number>

# Undo previous change
kubectl rollout undo

# Rollback to specific revision
kubectl rollout undo deployment <name> --to-revision=<number>

# Create a deployment then set an environment variable (There's no --env option for deployment so it's a 2 step process)
k create deploy <name> --image=<name>
k set env deploy <name> --env=K=V --env=K=V

# Set from a configMap or from a secret 
k set env deploy/<name> --from=configmap/<cm-name> --keys="key1,key2"

k set env deploy/<name> --from=secret/<secret-name> --keys="key1,key2"

# Both the previous approaches load single keys specified in the cm or secret, to load the whole cm or secret just use --from=<name>

# Load all configMap data
k set env deploy/<name> --from=configmap/<name>

# Load all secrets data 
k set env deploy/<name> --from=secret/<name>

# Setting resource requests and limits for deployment
k set resources deploy/nginx --requests=memory=256Mi,cpu=500m --limits=memory=500Mi,cpu=750m

```

* deployment.spec
  * .replicas
  * .strategy.type
    * .rollingUpdate (Updates pods one at a time to guarantee availability of the application)
      * .maxUnavailable (Max number of pods that are upgraded at the same time)
      * .maxSurge (Max number of pods that can be created over the desired number of pods, the value can be absolute number of a percentage of the desired pods)
    * .recreate (All pods are killed and new Pods are created which results in temporary unavailability, useful if we can't run differnt versions of the application)
  * .selector 
  * .template



---

#### AutoScaling Deployments:
* Works by setting a target CPU along with min and max number of replicas
* Target CPU is expressed as a percantage of the Pod's CPU requests
* `Metrics server` is used to collect the metrics and based on it the desired state is specified

<details>
<summary>HorizontalPodAutoscaler yml example:</summary>
<p>

```yml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler 
metadata:
  name: redis-hpas
  labels:
    app: redis
spec:
  maxReplicas: 5
  minReplicas: 1
  targetCPUUtilizationPercentage: 70
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: redis-deploy

```

</p>
</details>


Same with kubectl:
```bash
kubectl autoscale deployment <name> --max=<number> --min=<number> --cpu-percent=70

# Get HorizontalPodAutoscaler
kubectl get hpa
```

---

#### Rolling updates and rollbacks:
* Rollout is a process of updating and replacing replicas with replicas matching a new `Deployment` template
* Deployments create RS by defaults
* Rolling updates provide a way to update a deployment to a new container version by gradually updating replicas thus no downtime is introduced
* Rollouts are only triggered when there's a change in the **pod spec** , if there's a change for number of replicas for example, this doesn't trigger a rollout.

```bash
kubectl set image deployment/<deployment-name> <container name>=<image name> --record # --record flag recrods info about the update so that it can be rolled back later

# If the --record was forgotten then manually annotate the deployment 
k annotate deploy/<name> kubernetes.io/change-cause="Reason for rollout"

# Scale a deployment 
kubectl scale deployment <name> --replicas <number>

kubectl rollout status deployment/<name>

# Check deployment versions 
kubectl rollout history deployment/<name>

# Get more info about a specific revision
kubectl rollout history deployment/<name> --revision=<number>

# Pause the rollout of a deployment
kubectl rollout pause deployment <name>

# Resume the rollout of a deployment
kubectl rollout resume deployment <name>

# Rollback
kubectl rollout undo deployment/<name>

# Rollback to a specific revision
kubectl rollout undo deployment/<name> --to-revision=<number>

```
In `rollingUpdate` strategy we can specify 
  * `maxSurge%` specifies how many replicas over the desired total are allowed during rollout, a higher surge % allows new pods to be created without waiting for the old ones to be deleted
  * `maxUnavailable%` specifies how many of the old pods to be deleted without waiting for the new pods to be ready

---

#### Jobs and cronJobs:
* `Jobs` are like pods but they run for a specific job and when it's finished the container stops running
* Useful for tasks like backup, calculation, batch processing
* A pod start by a job must have its `restartPolicy` set either to
  * OnFailure (re-run the failing container on the same pod)
  * Never (re-run the failing container in a new pod)
* 3 Different job types can be started which is specified by **completions** (Number of pods that will be created) and **Parallelism** parameters
  * Non parallel jobs: 1 Pod is started unless the pod fails
    * Completions = 1
    * Parallelism = 1
  * Parallel jobs with a fixed completion count where the jobs is complete after running as many times as specified by `jobs.spec.completions`
    * Completions = n
    * Parallelism = m
  * Parallel jobs with a work queue where multiple jobs are started but all is finished when one job of those that were started is finished
    * Completions = 1
    * Parallelism = n
* `CronJobs` are your regular linux cron jobs, they run regularly according to the specified schedule
* When running a CronJob a Job will be scheduled which will start a new pod
```bash
# get Jobs 
kubectl get jobs

# get cron jobs
kubectl get cj 
```

* job.Spec
  * .activeDeadlineSeconds # Durtation after which the job will be terminated
  * .completions # number of times the job should be completed
  * .parallelism # max number of pods that should run at any given time 
  * .template 

* cronjob.spec
  * .schedule
  * .jobTemplate
    * .spec.template.spec.restartPolicy
* `CronJobs` create `jobs` which in turn create `pods`
---

#### Services:
* Provide network access to a set of pods 
* A service has a fixed IP address
* It handles Pod IP changes
* Distribute requests across pods in the group
* When left for k8s to allocate port number, it allocats within the range [30K - 32,767] 
* Service types are:
  * **ClusterIP** : Service is exposed within the cluster using Internal IP address, it's also accessible using cluster DNS, `kube-proxy` is responsible for proxying requests for the service (to one of its `ep`s)
  * **NodePort** : Service is exposed externally via a port which listens on each node in the cluster (The `ClusterIP` is still given to the service) so each request to the NodePort gets routed to the ClusterIP
  * LoadBalancer: For cloud providers
  * externalName: Enabled by DNS not proxying, we configure an external NameService with a DNS name and requests for the service return a CNAME record with the external DNS name (used by the services running outside of k8s such as DB as a service)

- NodePort creates ClusterIP
- LoadBalancer creates both NodePort and ClusterIP

```bash
# Get IP address of each replica
kubectl get ep <service-name>
```

* service.spec
  * [.ports](https://stackoverflow.com/a/61452441)
    * **.port** (Port of the service itself)
    * .targetPort (The port where the pod will forward traffic to)
    * .nodePort (Port on the node where external traffic comes in)

##### Example:
```bash
# 1- Create a pod
kubectl run nginx --image=nginx

# 2- Expose the pod (Create a SVC)
kubectl expose pod nginx --port=8888 --target-port=80

# Create a headless service 
k create svc clusterip <name> --clusterip="None" --dry-run=client -o yaml
```

Running `kubectl describe svc nginx` returns 
![command_result](https://github.com/theJaxon/CKAD/blob/master/etc/svc/SVC.jpg)

Here any traffic from `IP:Port` gets redirected to `EP:TargetPort` 


---

#### :small_orange_diamond: NetworkPolicies:
* Allows the control of what traffic come in [`ingress`] and leave [`egress`] the pod (Same functionality of a firewall)
* Equivalent to the relation between a securityGroup and an EC2 instance in AWS
* NetworkPolicies are **Namespaced** so we can configure a NetworkPolicy for each NS
* Network isolation can be configured to block traffic to pods by running pods in dedicated namespaces
* NetworkPolicy depends on the support from the network provider (Not all providers offer support) and if there's no support then the policy applied won't have any effect
* Connections are stateful (just like SG) so allowing one direction is enough
* There are 2 kinds of pods when it comes to NetworkPolicies, a `non-isolated` pod and an `isolated` one, the non-isolated receives traffic from any source but when we apply a NetworkPolicy to the pod it becomes isolated
* If the `podSelector` section in the NetworkPolicy yaml definition is empty then the policy will be applied to all pods in that namespace
* If only `ingress` was specified in the yaml description then all egress traffic will be allowed by the policy
* If the `from` is removed from the `ingress` definition then all traffic is allowed to enter the port, if `ports` list is removed then all traffic is allowed **from the specified sources**
* `from` list can either be:
  * podSelector
  * namespaceSelector
  * ipBlock 
* `k explain networkpolicy.spec.ingress.from`
* In networkPolicies, `allow` rules override `deny` rules
`policyTypes` sets whether the policy governs ingress, egress or both 

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy 
metadata:
  name: nw-policy 
spec:
  podSelector: # 1
    matchLabels:
      key: value 
  
  policyTypes: # 2
  - Ingress
  - Egress 
  
  ingress: 
  - from: 
    ports:

```

```bash
kubectl get networkpolicies

# Use netpol for short 
kubectl get netpol

kubectl describe networkpolicy <name>
```

* NetworkPolicy.spec
  * .podSelector.matchLabels
  * .policyTypes
    * - Ingress
    * - Egress
  * .ingress
    * .from
    * .ports
  * .egress
    * .to 
    * .ports
  

---

#### Volumes:
* Tied to pod lifecycle (Different from `PV`)
* Share data betwwen containers and tolerate container restarts
* Used for non durable storage that gets deleted when the pod is deleted
* `emptyDir` volume type allows the creation of a shared storage between the containers

```bash
# List containers inside pod in case there are more than 1 container
kubectl get po <name> -o jsonpath='{.spec.containers[*].name}'
```

[Volume Types](https://kubernetes.io/docs/concepts/storage/volumes/) List:
* hostPath
* configMap
* emptyDir
* persistentVolumeClaim
* secret

1. hostPath
```yml
volumes:
- hostPath:
    path: </path/on/host/os>
    type: Directory
```

5. secret 
```yml
volumes:
- name: mysecret 
  secret:
    secretName: <secret-object-name>
```

---

#### PVs and PVCs:
* Persistent Volume allows Pods to connect to external data storage 
* Independent of Pod's Lifetime
* PVC access modes are:
  * readWriteOnce (Allow volumes for RW operations only by a single **node** at a time)
  * readOnlyMany
  * readWriteMany
* ConfigMaps are specific volume objects that connect to config files and variables
* Secrets are like configMaps but the data is encoded
* PVC will bind to a PV according to the availability of the requested `accessModes` and `capacity`

* pv.spec
  * .capacity
    * .storage 
  * **.accessModes**
    * .ReadWriteOnce
    * .ReadWriteMany 
    * .ReadOnlyMany
  * .<type> [.hostPath, .nfs, .cephfs]


* pvc.spec
  * **.accessModes**
  * .resources
    * .requests.storage

* **`AccessModes`** between PV and PVC must match so that the state becomes `bound`

PV example:
```yml
spec:
  accessModes:
  - ReadWriteOnce 
  capacity:
    storage: 1Gi 
  hostPath: 
    path: /tmp/data
```

PVC example:
```yml
spec:
  accessModes:
  - ReadWriteOnce 
  resources:
    requests:
      storage: 500Mi
```

---

#### Ingress:
* Gives `Services` externally reachable URLs
* Allows traffic load balancing across services
* Offers name based virtual hosting
* Ingress controller must be setup to allow the use of ingress resources
* Examples of ingress controllers are nginx, haproxy, traefik, kong, contour
* We commonly configure a `default` backend for the ingress controller so that the traffic that doesn't match any `path` gets redirected to it
* ingress.spec
  * .rules
    * .host (optional and if not supplied then the rule applies to all HTTP traffic)
    * .http.paths (List of paths each having its own backend)
      * .path
        * .backend (Service name + Service Port)
          * .serviceName
          * .servicePort
      * .backend
      * .path
      * .pathType
---

#### Service Discovery mechanisms:
1. Environment Variables
  * SVC address automatically injected in containers
  * Environment vars follow naming conventions based on SVC name
  * Hyphens (-) gets replaced with underscores

:o: Naming pattern for environment variables service discovery:
  - IP address: <SVC_NAME_ALL_CAPS>_SERVICE_HOST
  - Port: <SVC_NAME_ALL_CAPS>_SERVICE_PORT
  - Named port: <SVC_NAME_ALL_CAPS>_SERVICE_PORT_<PORT_NAME_ALL_CAPS>

2. DNS
  * DNS records automatically created in the cluster DNS
  * Containers automatically configured to query cluster DNS

:o: Naming pattern for DNS service discovery:
  - IP address: <SVC_NAME>.<SVC_NS>
  - Port: Needs to be extracted from SRV DNS record



- When using Environment variables for service discovery, the service must be created before the pod, also both the svc + po must be in the same NS

---

### :zap: Examples:

<details>
<summary>1- SVC + Po example</summary>
<p>

```yml
# Pod definition section 
apiVersion: v1
kind: Pod 
metadata:
  name: nginx-po
  labels:
    app: web-server
spec:
  containers:
  - name: nginx
    image: nginx 

---

# Service definition section
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: web-server 
spec:
  type: NodePort
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 80
  selector:
    app: web-server

```

</p>
</details>

---

<details>
<summary>2- Multi container Po + NS</summary>
<p>

```yml
# 1- Create NS 
apiVersion: v1
kind: Namespace 
metadata:
  name: multi-container-ns
  labels:
    app: counter

---

# 2- Create Pod
apiVersion: v1
kind: Pod 
metadata:
  name: multi-container-po
  namespace: multi-container-ns
spec:
  containers:
  - name: redis
    image: redis:latest 
    imagePullPolicy: IfNotPresent # Prevent k8s from always pulling the image
    ports:
    - containerPort: 6379

  - name: server
    image: lrakai/microservices:server-v1
    ports:
    - containerPort: 8080
    env:
    - name: REDIS_URL
      value: redis://localhost:6379

  - name: counter
    image: lrakai/microservices:counter-v1 
    env:
    - name: API_URL
      value: http://localhost:8080

  - name: poller
    image: lrakai/microservices:poller-v1
    env:
    - name: API_URL
      value: http://localhost:8080


```

</p>
</details>

---

<details>
<summary>3- Service Discovery</summary>
<p>

```yml
# 1- Create Namespace 
apiVersion: v1
kind: Namespace 
metadata:
  name: discovery-ns
  labels:
    app: counter

--- 

# 2- Data tier 
# 2.1 Data tier Service
apiVersion: v1
kind: Service
metadata:
  name: data-tier
  namespace: discovery-ns 
  labels:
    app: microservice 
spec:
  type: ClusterIP
  selector:
    tier: data
  ports:
  - protocol: TCP
    port: 6379
    name: redis 

---

# 2.2 Data tier Pod 
apiVersion: v1
kind: Pod 
metadata:
  name: data-tier 
  labels:
    tier: data
    app: microservice
spec:
  containers:
  - name: redis
    image: redis:latest 
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 6379

---

# 3- App tier 
# 3.1 App tier SVC
apiVersion: v1
kind: Service
metadata:
  name: app-tier
  labels:
    app: microservice
spec:
  type: ClusterIP
  selector:
    tier: app 
  ports:
  - protocol: TCP
    port: 8080

---

# 3.2 App tier Po
apiVersion: v1
kind: Pod 
metadata:
  name: app-tier 
  labels:
    app: microservice
spec:
  containers:
  - name: server
    image: lrakai/microservices:server-v1
    ports:
    - containerPort: 8080
    env:
    - name: REDIS_URL
      value: redis://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)

---

# 4- Support tier
apiVersion: v1
kind: Pod 
metadata:
  name: support-tier
  labels:
    app: microservice
    tier: support
spec:
  containers:
  - name: counter
    image: lrakai/microservices:counter-v1
    env:
    - name: API_URL
      value: http://app-tier.discovery-ns:$(APP_TIER_SERVICE_PORT)

  - name: poller
    image: lrakai/microservices:poller-v1
    env:
    - name: API_URL
      value: http://app-tier.discovery-ns:$(APP_TIER_SERVICE_PORT)
```

</p>
</details>

---

Qs
1. Create configMap named `ckad-cm` with `EXAM=CKAD` and add it one time as a env variable in a pod and another time as a volume.
2- 