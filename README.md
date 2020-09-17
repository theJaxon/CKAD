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
kubectl describe <object>
kubectl logs <object>

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