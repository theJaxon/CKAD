# CKAD
Preparation for the Certified Kubernetes Application Developer V1.19 [CNCF] exam

---

### Objectives:
* The hashtag provided means use the keyword to search in the documentation at [k8s website](https://kubernetes.io/docs/home/)

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

# alias kubectl to k (from https://github.com/krzko/awesome-cka)
alias k='kubectl'

k completion -h # In case you forgot the source command listed above
```

2. [Delete](https://stackoverflow.com/a/33510531) all pods in a NS
```bash
kubectl delete --all po --namespace=<name>
```

3. kubectl 
```bash
kubectl <verb> --help | less

kubectl | less

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

# Get a shell into the container running https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/ 
kubectl exec --stdin --tty <name> -- /bin/bash

# Proxy 
kubectl proxy --port=<number> &
# Display API Groups after using the proxy command 
curl http://localhost:<number>

# Narrow down a selection from the shown API Groups 
curl http://localhost:<number>/version

# Get list of pods in a specific NS
curl http://localhost:<number>/api/v1/namespaces/<Namespace>/pods

``` 

4. Watch command (2 ways)
* `kubectl get` command supports the `-w` option.
* watch -n 1 `kubectl get po` specifies a watch command that gets updated each 1 second  

5. [Delete a block](https://vi.stackexchange.com/questions/1915/how-do-i-delete-a-large-block-of-text-without-counting-the-lines) from the YAML file 
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
---

### :file_folder: Important dirs:
* Kubectl 
  * ~/.kube/config # kubectl config view

### JSONPATH commands:
```bash
# Get all pod names 
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'

```


#### :gem: Common commands:

```bash
kubectl cluster-info

# List kubernetes objects 
kubectl api-resources 

kubectl get <object>

# Troubleshooting commands
kubectl describe <object> # Used to also view resource requests and limits

kubectl logs <object>

# Follow the logs using -f 
kubectl logs <pod-name> -f

# When having more than 1 container in a pod use the -c flag 
kubectl logs <pod> -c <container>

# Get All pods in the cluster
kubectl get po -A

# Updating existing object definition
kubectl apply -f file_name.yml
kubectl edit <object> <name>
kubectl exec <pod-name> -- command-here # ls /etc/conf for example
kubectl exec --tty --stdin <pod-name> -c <container-name> -- /bin/bash # Getting into multi-containter pod

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

---

#### Pods:
- Each pod has a unique IP address
- Containers within this pod share the same IP address

---

#### Namespaces:
```bash
# Create ns
kubectl create ns <name>
```
---


#### ConfigMaps:
- ConfigMaps can be included either as a `env` variable or as a `volume`
- When they're included as a volume, for each key there is a corresponding file that contains the value that is created on the mountPath

---

#### SecurityContext:
* Define privilege and access control settings for pods and containers
* They're part of the Pod specification
* Without a securityContext, the user running the commands in the container inside the pod will be `root` by default

---

#### Resource requirements:
- Define container's memory and cpu requirements using `requests` and `limits`
- `resource request` is the amount of resources necessary to run a container (The node must have enough available resources for the requests)
- `resource limits` is the maximum value for the resource usage of a container
- CPU core usage can be specified using `m` which means `milli CPU`
  - **250m** means a **quarter** of a cpu core 
  - **500m** means **half** of a cpu core and so on

---

#### Secrets:
* Store sensitive data in a secure fashion (using base64 encryption lol)

---

#### ServiceAccount:
* Allows the container to interact with kubernetes API
* Image pull secrets authenticate with private container registeries

```bash
kubectl create serviceaccount <name>
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

---

#### Probes:
- Allows customizing how K8s determines container status
- Liveness probe indicates whether the container is running properly and decides whether the cluster will automatically stop or restart the container
  - It just does health checks
- Readiness probe indicates whether the container is ready to receive requests 

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

```bash
kubectl top po

# Narrow down to a specific pod
kubectl top po <name>

kubectl top nodes
```

#### Labels & Selectors:
```bash
# Get pods based on specific label
kubectl get po -l key=value

# Get pods based on multiple labels 
kubectl get po -l key=value,key=value

# Get pods that doesn't match the labels 
kubectl get po -l key!=value

# Set based selectors 
kubectl get po -l 'key in (val1,val2,val3)'

# Watch rollout and changes in the deployment 
kubectl get po -w

```
---

#### Annotations [Metadata]:
* Useful when dealing with external tools for automation where we need attach additional info to our pods
* Just add them in `metadata` section using `annotations:` then add key value pairs

---

#### Deployments:
* Represent multiple replicas of a Pod
* The `metadata` section of the deployment template doesn't need a name because k8s automatically generates a name for each replica 

```bash
# Scale deployment using kubectl 
kubectl scale deployment <name> --replicas=<number>
```

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

```bash
kubectl set image deployment/<deployment-name> <container name>=<image name> --record # --record flag recrods info about the update so that it can be rolled back later

kubectl rollout status deployment/<name>

# Check deployment versions 
kubectl rollout history deployment/<name>

# Get more info about a specific revision
kubectl rollout history deployment/<name> --revision=<number>

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
* Jobs are like pods but they run for a specific job and when it's finished the container stops running
* CronJobs are your regular linux cron jobs, they run regularly according to the specified schedule

```bash
# get cron jobs
kubectl get cj 
```

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

---

#### NetworkPolicies:
* Allows the control of what traffic come in and leave the pod
* Equivalent to the relation between a securityGroup and an EC2 instance in AWS
* NetworkPolicies are Namespaced so we can configure a NetworkPolicy for each NS
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

kubectl describe networkpolicy <name>
```

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
* configMap
* emptyDir
* persistentVolumeClaim
* secret

---

#### PVs and PVCs:
* Persistent data storage 
* Independent of Pod's Lifetime
* PVC access modes are:
  * readWriteOnce (Allow volumes for RW operations only by a single **node** at a time)
  * readOnlyMany
  * readWriteMany

---

#### Service discovery mechanisms:
1. Environment Variables
  * SVC address automatically injected in containers
  * Environment vars follow naming conventions based on SVC name

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

