
---

## 1. Introduction to Kubernetes

Kubernetes, often abbreviated as **K8s**, is a **portable, extensible, open-source platform for managing containerized workloads and services**. It facilitates both declarative configuration and automation. Kubernetes emerged from Google's internal Borg project, addressing the limitations of simply running containers with tools like Docker.

**Why Kubernetes is Essential:**  
While Docker allows you to package applications into containers, a single Docker container often cannot serve millions of requests. When moving to a production-like environment, managing Docker containers becomes complex. This is where Kubernetes steps in, acting as a **container orchestration platform**. It manages your Docker containers, providing crucial features that Docker itself doesn't offer by default, such as:
- **Scalability**
- **Automated Deployment**
- **Auto-healing**
- **Rollout and Rollback**
- **Load Balancing**
- **Enterprise-level Support**

---

## Kubernetes Architecture: The Master-Worker Model

![Kubernetes Architecture](https://github.com/user-attachments/assets/5d5b2f81-7ec9-4f63-bb30-6cfdc6b13335)

### Control Plane (Master Node Components)
- **API Server**
- **etcd**
- **Scheduler**
- **Controller Manager**
- **Cloud Controller Manager**

### Data Plane (Worker Node Components)
- **Kubelet**
- **Kube-proxy**
- **Container Runtime**

---

## 2. Getting Started: Setting up a Kubernetes Cluster (Minikube)

**Prerequisites:**
- `kubectl` installed
- 2 CPUs, 2GB+ RAM, Hypervisor like VirtualBox/Docker

**Install kubectl (Linux Example):**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

**Install Minikube:**
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**Start Minikube:**
```bash
minikube start --memory=4096 --cpus=2 --driver=docker
```

**Check Cluster:**
```bash
kubectl get nodes
```

---

## 3. Kubernetes Objects - The Building Blocks

---

### 3.1 Pods

**Definition:**  
Pod is the smallest deployable unit in K8s. Can contain 1+ containers, shares networking/storage.

**YAML:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
  labels:
    app: nginx-web
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

**Deploy Commands:**
```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod my-nginx-pod
kubectl logs my-nginx-pod
kubectl exec -it my-nginx-pod -- /bin/bash
kubectl delete pod my-nginx-pod
```

---

### 3.2 Deployments

**Purpose:** Abstraction over Pods. Handles scaling, rollout, healing.

**YAML:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

**Commands:**
```bash
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get pods
kubectl delete pod <name>
kubectl scale deployment nginx-deployment --replicas=3
```

---

### 3.3 Services

**Purpose:** Stable networking + load balancing across Pods

**YAML (NodePort):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
```

**Access:**
```bash
minikube ip
curl http://<minikube-ip>:30080
```

---

## 3.4 ConfigMaps & Secrets

---

### ConfigMap Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  DB_PORT: "3306"
  APP_ENV: "production"
```

**Commands to Use ConfigMap:**
```bash
kubectl apply -f my-configmap.yaml
kubectl get configmap
kubectl describe configmap my-app-config
```

**Use in Deployment (as Environment Variables):**
```yaml
env:
  - name: DATABASE_PORT
    valueFrom:
      configMapKeyRef:
        name: my-app-config
        key: DB_PORT
  - name: ENVIRONMENT
    valueFrom:
      configMapKeyRef:
        name: my-app-config
        key: APP_ENV
```

**Mount as Volume:**
```yaml
volumes:
  - name: config-volume
    configMap:
      name: my-app-config
volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

---

### Secrets Example

```bash
kubectl create secret generic my-db-secret --from-literal=DB_USERNAME=admin --from-literal=DB_PASSWORD=my_secure_password
```

**Use in Deployment:**
```yaml
env:
  - name: DATABASE_USERNAME
    valueFrom:
      secretKeyRef:
        name: my-db-secret
        key: DB_USERNAME
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: my-db-secret
        key: DB_PASSWORD
```

---

## 4. Deployment Strategies

---

### RollingUpdate (default)

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

### Recreate Strategy

```yaml
strategy:
  type: Recreate
```

### Canary Deployment (Manual Example)

1. Create new deployment with new label/version.
2. Route small percentage of traffic via a modified Service selector.
3. Gradually increase traffic and remove old one.

### Blue-Green Deployment (Conceptual Steps)

1. Two Deployments: `blue` (live), `green` (new).
2. New version deployed in green.
3. Switch service selector from blue → green.
4. Delete or scale down blue.

---

## 5. Helm - Kubernetes Package Manager

---

### Helm Chart Structure

```
my-nginx-chart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── _helpers.tpl
```

### Chart.yaml
```yaml
apiVersion: v2
name: my-nginx-chart
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
```

### values.yaml
```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.14.2"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30080
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-nginx-chart.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "my-nginx-chart.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "my-nginx-chart.name" . }}
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

### Helm Commands

```bash
helm install my-nginx ./my-nginx-chart
helm upgrade my-nginx ./my-nginx-chart
helm uninstall my-nginx
```

---

## 6. Controllers: Advanced Workloads

---

### StatefulSet

Used for stateful applications (e.g., databases) requiring stable network IDs, persistent storage.

**Example:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

---

### DaemonSet

Runs a Pod on every Node.

**Example:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      name: node-exporter
  template:
    metadata:
      labels:
        name: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter
```

---

### Job

Runs a task to completion.

**Example:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello World"]
      restartPolicy: Never
```

---

### CronJob

Scheduled task, like crontab.

**Example:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["echo", "Hello CronJob"]
          restartPolicy: OnFailure
```

---

## 7. Monitoring with Prometheus & Grafana

---

### Prometheus Setup via Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
```

**Expose UI:**
```bash
kubectl expose service prometheus-server --type=NodePort --name=prometheus-ext
minikube service prometheus-ext --url
```

---

### Grafana Setup via Helm

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
```

**Expose UI:**
```bash
kubectl expose service grafana --type=NodePort --name=grafana-ext
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode
minikube service grafana-ext --url
```

**Steps inside Grafana UI:**
- Add Prometheus as data source
- Import Dashboard ID `3662` for Kubernetes metrics

---

## 8. Custom Resources & Operators

---

### CustomResourceDefinition (CRD)

Extends K8s API with new object types.

**CRD Example:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

---

### Operator

Custom controller that manages CRs and enforces desired state.

**Popular Operator Examples:**
- ArgoCD Operator
- Prometheus Operator
- ElasticSearch Operator

Deploy via:
```bash
kubectl apply -f <operator-crd.yaml>
kubectl apply -f <operator-controller.yaml>
```

Use Custom Resource:
```bash
kubectl apply -f <custom-resource.yaml>
```

Check logs:
```bash
kubectl logs <operator-pod-name> -n <namespace>
```

---

## 9. Kubeconfig Management

A `kubeconfig` file is a YAML configuration file used by `kubectl` and other tools to communicate securely with the Kubernetes API server.

**Key Sections:**
- **Clusters**: API server endpoint + certificate info
- **Users**: Auth credentials (token, certs)
- **Contexts**: Link a user to a cluster

**Commands:**
```bash
kubectl config view
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <context-name>
```

**RBAC for Access Control:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 10. Kubernetes Scheduling & Placement

---

### Node Selector

Assign Pods to Nodes with specific labels.

**Add Label to Node:**
```bash
kubectl label node <node-name> disktype=ssd
```

**Pod with NodeSelector:**
```yaml
spec:
  nodeSelector:
    disktype: ssd
```

---

### Taints and Tolerations

**Taint a Node:**
```bash
kubectl taint nodes <node-name> key=value:NoSchedule
```

**Toleration in Pod:**
```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

---

### Node Affinity

**Hard Requirement:**
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

**Soft Preference:**
```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
        matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

---

### Pod Affinity & Anti-affinity

**Pod Affinity:**
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - frontend
      topologyKey: "kubernetes.io/hostname"
```

**Pod Anti-Affinity:**
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - frontend
      topologyKey: "kubernetes.io/hostname"
```

---

## 11. Horizontal Pod Autoscaler (HPA)

Scales Pods based on CPU or custom metrics.

**Enable Metrics Server (Minikube):**
```bash
minikube addons enable metrics-server
```

**Create HPA:**
```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=5
```

**Check HPA Status:**
```bash
kubectl get hpa
```

---

## 12. Cluster Autoscaler (CAS)

Adjusts node count based on pending Pods.

**Install on EKS/GKE using Helm or manifest**
- Requires cloud permissions
- Watches for unschedulable Pods

More: [https://github.com/kubernetes/autoscaler](https://github.com/kubernetes/autoscaler)

---

## 13. Helmfile

Helmfile is a declarative tool to manage multiple Helm releases via a single YAML.

**helmfile.yaml Example:**
```yaml
repositories:
  - name: prometheus-community
    url: https://prometheus-community.github.io/helm-charts

releases:
  - name: prometheus
    namespace: monitoring
    chart: prometheus-community/prometheus
    version: 15.0.0
    values:
      - prometheus-values.yaml
```

**Commands:**
```bash
helmfile sync       # installs/upgrades
helmfile apply      # shows diff and applies
helmfile destroy    # uninstall all
```

---

## 14. Helm Hooks

Hooks are Kubernetes objects with annotations that run during release lifecycle events.

**Hook Annotations:**
```yaml
metadata:
  annotations:
    "helm.sh/hook": pre-install,post-install
    "helm.sh/hook-delete-policy": hook-succeeded
```

**Hook Types:**
- pre-install
- post-install
- pre-delete
- post-delete
- pre-upgrade
- post-upgrade
- pre-rollback
- post-rollback
- test

---

## 15. GitOps Mentions

GitOps tools use CRDs and controllers to manage declarative infrastructure.

**Popular Tools:**
- Argo CD
- Flux CD
- Jenkins X

**Key Concepts:**
- Desired state stored in Git
- Reconciliation loops
- Rollbacks via Git history

---

## 🎯 Conclusion

Mastering Kubernetes is like becoming the conductor of a dynamic orchestra. Each Kubernetes object—Pods, Deployments, Services, ConfigMaps, and Secrets—is an instrument, meticulously designed to play a specific role in your application's symphony.

Tools like Helm are your sheet music, simplifying complex compositions and ensuring that your entire orchestra can perform coherently across different stages. Understanding advanced concepts like custom resources and comprehensive monitoring tools (Prometheus and Grafana) equips you not just to conduct, but to compose new pieces and ensure every note is perfectly tuned and heard.

By practicing these fundamentals and delving into the "why" behind each component, you'll be well-prepared to troubleshoot complex issues, design robust solutions, and confidently lead your own cloud-native ensemble.

---
