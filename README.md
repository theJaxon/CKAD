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

1. Probes (Liveness, Readiness)
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
```

---


#### :gem: Common commands:

```bash
kubectl cluster-info

# List kubernetes objects 
kubectl api-resources 

kubectl get <object>

# Troubleshooting commands
kubectl describe <object> # Used to also view resource requests and limits
kubectl logs <object>
# When having more than 1 container in a pod use the -c flag 
kubectl logs <pod> -c <container>

# Get All pods in the cluster
kubectl get po -A

# Updating existing object definition
kubectl apply -f file_name.yml
kubectl edit <object> <name>
kubectl exec <pod-name> -- command-here # ls /etc/conf for example


```

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
The sidecar container enhances the functionality of the main container (ex: Git sync to a web server container)

2. Ambassador pod
It receives network traffic then pass it to the main container (ex: ambassador listens on a custom port then forwards traffic to the main hard-coded port)

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

#### Rolling updates and rollbacks:
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

---

#### Jobs and cronJobs:
* Jobs are like pods but they run for a specific job and when it's finished the container stops running
* CronJobs are your regular linux cron jobs, they run regularly according to the specified schedule

---

#### Services:
* Provide network access to a set of pods 
* Service types are:
  * **ClusterIP** : Service is exposed within the cluster using Internal IP address, it's also accessible using cluster DNS 
  * **NodePort** : Service is exposed externally via a port which listens on each node in the cluster
  * LoadBalancer: For cloud providers

```bash
# Get IP address of each replica
kubectl get ep <service-name>
```

---

#### NetworkPolicies:
* Allows the control of what traffic come in and leave the pod

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
* Provide permanent storage for the container
* `emptyDir` volume type allows the creation of a shared storage between the containers

```bash
# List containers inside pod in case there are more than 1 container
kubectl get po <name> -o jsonpath='{.spec.containers[*].name}'
```

---

#### PVs and PVCs:
* Persistent data storage 