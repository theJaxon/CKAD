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