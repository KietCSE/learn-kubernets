# Kubernetes: Basic Operations and Concepts

---

## 📚 Table of Contents

- [1. Introduction](#1-introduction)
- [2. Core Kubernetes Concepts](#2-core-kubernetes-concepts)
  - [2.1. Node](#21-node)
  - [2.2. Pod](#22-pod)
  - [2.3. Service](#23-service)
  - [2.4. Deployment](#24-deployment)
  - [2.5. ReplicaSet](#25-replicaset)
  - [2.6. Namespace](#26-namespace)
  - [2.7. Ingress](#27-ingress)
  - [2.8. Helm](#28-helm)
  - [2.9. Volume](#29-volume)
  - [2.10. StatefulSet](#210-statefulset)
  - [2.11. Network](#211-network)
- [3. Writing YAML Files for Kubernetes](#3-writing-yaml-files-for-kubernetes)
  - [3.1. Structure of a Kubernetes YAML File](#31-structure-of-a-kubernetes-yaml-file)
  - [3.2. Key Sections Explained](#32-key-sections-explained)

---

## 1. Introduction

Kubernetes is an open-source platform for managing containerized applications. This document provides detailed guidance on core concepts such as **Node**, **Pod**, **Service**, **Deployment**, and **ReplicaSet**, along with commonly used `kubectl` commands to manage them.

---

## 2. Core Kubernetes Concepts

### 2.1. Node

A **Node** is a physical or virtual machine in a Kubernetes cluster responsible for running containerized applications. There are two types of Nodes:
- **Master Node**: Runs the control plane components, such as API Server, Scheduler, and Controller Manager.
- **Worker Node**: Runs user applications in Pods, managed by the following components:

| Component          | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| **Kubelet**        | An agent running on each Node, ensuring containers in Pods operate correctly. |
| **Kube-proxy**     | Manages network rules to connect Pods internally and externally.             |
| **Container Runtime** | Software for running containers, e.g., containerd, CRI-O (Docker deprecated since Kubernetes 1.20). |

> **Note**: Since Kubernetes 1.20, Docker is no longer the default container runtime. Common runtimes include **containerd** and **CRI-O**.

**Related Commands**:
```bash
kubectl get nodes
kubectl describe node <node-name>
kubectl cordon <node-name>
kubectl drain <node-name>
```
---
### 2.2. Pod

A **Pod** is the smallest unit in Kubernetes, typically containing one or more containers that share resources such as storage and networking. Each Pod represents a single instance of a running application and is usually managed by higher-level objects like Deployments, ReplicaSets, or StatefulSets to ensure auto-recovery, scaling, and more.

**Related Commands**:
```bash
kubectl get pod
kubectl get pod -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/bash
kubectl scale <resource> <resource-name> --replicas=<number>

```

**Pod States**:
| State              | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| **Running**        | The Pod is running, and all containers are operating normally.               |
| **Pending**        | The Pod is waiting to be scheduled or containers are not yet ready.          |
| **Failed**         | The Pod has encountered an error, with one or more containers terminated with an error code. |
| **CrashLoopBackOff** | A container in the Pod is repeatedly restarting due to an error.            |

---
### 2.3. Service

A **Service** in Kubernetes defines how Pods are accessed, typically via a virtual IP address (ClusterIP) or an internal DNS name. It provides **load balancing** and **service discovery**, ensuring stable connectivity to Pods even as they are created, deleted, or rescheduled.

When a Service is created, Kubernetes automatically generates an **Endpoints** object (or **EndpointSlice** starting from Kubernetes 1.21) to track the IP addresses and ports of Pods matching the Service's selector. If no Pods match the selector, the Endpoints object will be empty, and the Service cannot route traffic.


**Service Port Configuration**:
- **port**: The port the Service listens on within the cluster (e.g., 80).
- **targetPort**: The port the Pod's container listens on (e.g., 8080). The Service proxies traffic from `port` to `targetPort` of matching Pods.

**Note**: Service selectors differ from Deployment selectors, as Services route traffic to Pods, while Deployments manage Pod creation and scaling.

**Service Types**:
| Type            | Description                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| **ClusterIP**   | Default type. Assigns a virtual IP accessible only within the cluster.       |
| **NodePort**    | Exposes the Service on a specific port of each Node for external access.     |
| **LoadBalancer**| Provides a public IP through a cloud provider's load balancer (e.g., AWS, GCP, Azure). |
| **Headless**    | No ClusterIP (`clusterIP: None`). DNS returns the IPs of matching Pods directly. |

**Related Commands**:
```bash
kubectl get services
kubectl get services --all-namespaces
kubectl describe service <service-name>
kubectl get endpoints <service-name>
```

**Default service** is kubernetes: It is the default service of the API server in the cluster. It allows pods to communicate further with the API server to control the cluster.

**Note**
- ClusterIP is used for internal communication in node
- NodePort is used for external communication but not effecient in many nodes
- LoadBalancer is used for external communication and effecient for many nodes

---
### 2.4. Deployment

A **Deployment** is a Kubernetes controller that manages the desired state of Pods, ensuring the specified number of Pod replicas are running, updated, or rolled back as needed. It uses a **ReplicaSet** to maintain the desired number of Pods.

**Key Features**:
- Manages Pod scaling and updates.
- Uses selectors to identify managed Pods (distinct from Service selectors, which route traffic).
- Supports rolling updates and rollbacks for application updates.

**Rollback** 


**Related Commands**:
```bash
kubectl get deployment
kubectl describe deployment <deployment-name>
kubectl get deployment <deployment-name> -o yaml
kubectl rollout status deployment <deployment-name>
kubectl rollout undo deployment <deployment-name>
kubectl set image deployment/<tên-deployment> <tên-container>=<tên-image>:<tag>
kubectl rollout undo deployment/<tên-deployment>                                  # rollback previous version 
kubectl rollout history deployment/<tên-deployment>                               # history of all versions
kubectl rollout undo deployment/<tên-deployment> --to-revision=2                  # rollback to version 2

```
---
### 2.5. ReplicaSet

A **ReplicaSet** ensures a specified number of Pod replicas are running at all times. It is typically managed by a Deployment to handle scaling and updates but can be used independently.

**Key Features**:
- Maintains the desired number of Pods based on a selector.
- Automatically replaces failed or deleted Pods.

**Related Commands**:
```bash
kubectl get replicaset
kubectl describe replicaset <replicaset-name>
kubectl scale replicaset <replicaset-name> --replicas=<number>
```
---

### 2.6. Namespace

A **Namespace** is a virtual cluster within a Kubernetes cluster, used to organize and isolate resources. It helps avoid naming conflicts, manage resources for different teams or environments, and control access.

**Use Cases**:
- Avoid naming conflicts between teams or applications.
- Group resources for management.
- Separate environments (e.g., dev, staging, production).
- Limit resource access per team.
- Share resources across environments.

**Note**:
- Some resources (e.g., Nodes, PersistentVolumes) are global and not tied to a namespace.
- Without specifying a namespace, the `default` namespace is used.
- Pods in a namespace can run on multiple Nodes.
- Kubernetes includes three system namespaces: `default`, `kube-system`, and `kube-public`.
- Each pod with create two container: main for running and pause for kêp track namespace 
- Một pod có thể có nhiều hơn container nếu dùng theo 
+ Sidecar container Ví dụ: chạy một container ứng dụng + một container Fluentd để đẩy log lên server.
+ Init container Các container khởi tạo chạy trước khi container chính bắt đầu (ví dụ: tải dữ liệu, thiết lập config...).
+ Ambassador / Adapter container Đóng vai trò như cổng giao tiếp hoặc chuyển đổi giao thức.

**Related Commands**:
```bash
kubectl get namespace
kubectl create namespace <namespace-name>
kubectl get pod --namespace <namespace-name>
kubectl describe namespace <namespace-name>
```
---

### 2.7. Ingress

An **Ingress** is a Kubernetes resource used to manage external HTTP/HTTPS traffic entering the cluster. It acts as a **reverse proxy**, routing requests to the appropriate Service based on the **hostname** or **URL path** — effectively replacing the need to manually install and configure NGINX.

### Why Use Ingress?

Without Ingress, exposing a Service externally requires:
- **NodePort**: Limited to port range `30000–32767`, not user-friendly or scalable.
- **LoadBalancer**: Each Service requires a dedicated load balancer → **high resource usage, costly (especially on cloud platforms)**.

**Ingress allows multiple routes (hostnames/paths) to be managed via a single IP or LoadBalancer**, improving efficiency and scalability.

### Advantages of Using NGINX Ingress Controller

When you deploy the **NGINX Ingress Controller**, you:
- Don’t need to install NGINX manually.
- Don’t need to write manual config files.
- Kubernetes automatically manages the reverse proxy config based on Ingress rules.
- Requests are automatically mapped → Ingress → Service → Pod.

### Key Components
- **Ingress Resource**: Declares routing rules for HTTP/HTTPS traffic.
- **Ingress Controller**: The software (e.g., NGINX, Traefik, HAProxy) that enforces Ingress rules.
- **Default Backend**: A Pod that handles unmatched requests (commonly returns 404).
- **Multiple Hosts/Paths**: Allows routing traffic to different applications based on domain names or paths.

### Related Commands

```bash
kubectl get ingress
kubectl get ingress --namespace <namespace-name>
kubectl describe ingress <ingress-name> --namespace <namespace-name>
```

---

### 2.8. Helm

**Helm** is a package manager for Kubernetes, simplifying application deployment and management through reusable templates.

**Key Features**:
- Clones common YAML configurations.
- Defines templates with customizable values in `values.yaml`.
- Manages releases for versioned deployments and rollbacks.

**Related Commands**:
```bash
helm install <release-name> <chart-name>
helm upgrade <release-name> <chart-name>
helm list
helm rollback <release-name> <revision>
```
---

### 2.9. Volume

A **Volume** in Kubernetes provides persistent storage for Pods. Volumes can be mounted into a container’s filesystem and may include ConfigMaps or Secrets. Persistence volume has to detach from pod and node, it is belong to cluster. 

#### Key Components:
- **PersistentVolume (PV)**:  
  A cluster-wide storage resource managed by a cluster administrator or dynamically provisioned using a `StorageClass`.  
  - PVs represent real physical storage (e.g., disk on a node, NFS share, cloud disk).
  - PVs are created independently of the Pod lifecycle, allowing persistent data across Pod restarts.

- **PersistentVolumeClaim (PVC)**:  
  A request for storage from a user or application.  
  - A PVC specifies size, access modes, and optionally a `StorageClass`.
  - Kubernetes tries to bind the PVC to an appropriate PV.
  - **PVC and the Pod using it must reside in the same namespace.**
  - This abstraction allows users to **request storage without needing to know implementation details**, promoting resource reuse and decoupling.

- **Pod → PVC → PV Workflow**:
  1. A Pod requests a volume using a PVC.
  2. The PVC looks for a matching PV.
  3. Once a suitable PV is found, the PVC is bound to it.
  4. Kubernetes mounts the volume into the Pod's container filesystem.

- **StorageClass**:  
  Defines *how* PVs are dynamically provisioned (known as **dynamic provisioning**).  
  - Specifies details like provisioner type (e.g., AWS EBS, GCE PD), volume binding mode, reclaim policy.
  - Allows automatic creation of PVs when a PVC is made.

- **Mounting ConfigMaps and Secrets**:
  - Kubernetes allows mounting **ConfigMaps** and **Secrets** as volumes inside containers.
  - This is commonly used for injecting configuration files, environment variables, or credentials.

**Related Commands**:
```bash
kubectl get pv
kubectl get pvc
kubectl describe pvc <pvc-name>
kubectl get storageclass

```

#### Volume Types (Overview)

| Volume Type       | Description                                |
|-------------------|--------------------------------------------|
| `emptyDir`        | Temporary storage, deleted when Pod stops  |
| `hostPath`        | Mounts a file/directory from host node     |
| `configMap`       | Injects configuration files                |
| `secret`          | Injects sensitive data like credentials    |
| `persistentVolumeClaim` | Mounts a bound Persistent Volume     |
| `nfs`             | Mounts a shared NFS volume                 |
| `csi`             | Container Storage Interface (pluggable)    |


#### emptyDir 
emptyDir là một loại volume được tạo ra khi một Pod được gán vào một Node. Thư mục trống (empty directory) này tồn tại trong suốt vòng đời của Pod. Khi Pod bị xóa (dù container bên trong bị restart), dữ liệu trong emptyDir sẽ biến mất.
- If container restarts, emptyDir also remain 
- You can not check emptyDir using command, you have to check in the yaml file
**Hữu ích cho các trường hợp như:**
- Cache tạm thời
- Buffer dữ liệu
- Dùng để chia sẻ dữ liệu giữa các containe
**Disadvatage** 
- Temporary storage 
- Each pod will has a emptyDir so problem come with replica 

#### hostPath 
hostPath là volume trỏ tới một đường dẫn cụ thể trên Node (máy chủ vật lý hoặc ảo) mà Pod đang chạy. Dữ liệu lưu trên hostPath vẫn tồn tại ngay cả khi Pod bị xóa.

**Disadvantage** 
- Không portable: Không dùng tốt trong môi trường nhiều Node (cluster lớn).
- Nguy cơ bảo mật cao: Container có thể truy cập vào file hệ thống host.

#### CSI 
CSI (Container Storage Interface) là mộft tiêu chuẩn mở giúp Kubernetes tích hợp với các hệ thống lưu trữ bên ngoài (external storage systems) một cách linh hoạt và mạnh mẽ hơn so với cơ chế volume plugin truyền thống. Cung cấp nhiều tính năng nâng cao hơn như snapshot, resize,... 

#### How to declare a persistent volume 
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <pv-name>  # Tên PV
spec:
  capacity:
    storage: <size>  # Dung lượng (vd: 10Gi)
  accessModes:       # Chế độ truy cập
    - <access-mode>  # Ví dụ: ReadWriteOnce
  persistentVolumeReclaimPolicy: <reclaim-policy>  # Retain/Delete/Recycle
  storageClassName: <storage-class>  # Tên StorageClass (nếu có)
  volumeMode: Filesystem  # Hoặc Block (tùy use case)
  # Phần khai báo volume (nfs, hostPath, awsEBS, etc.)
  <volume-type>:
    <config-details>
```
**Explain**
``accessModes``
- ReadWriteOnce (RWO): Chỉ 1 node được ghi/đọc.
- ReadOnlyMany (ROX): Nhiều node đọc.
- ReadWriteMany (RWX): Nhiều node ghi/đọc (NFS, Ceph hỗ trợ).

``persistentVolumeReclaimPolicy``
- Retain: Giữ lại data sau khi PVC xóa (admin phải xóa thủ công).
- Delete: Tự động xóa data (phụ thuộc vào storage backend).
- Recycle (deprecated): Xóa data và chuẩn bị PV để tái sử dụng.

``storageClassName``
- Khớp với tên StorageClass nếu dùng dynamic provisioning.
- Để trống nếu dùng static provisioning.

``volumeMode``
- Filesystem: Mount như thư mục (mặc định).
- Block: Truy cập trực tiếp ở dạng block device (cho database).

---

### 2.10. StatefulSet

A **StatefulSet** is a Kubernetes workload controller for stateful applications requiring stable identifiers, persistent storage, or specific startup ordering.

**Key Features**:
- Assigns unique, stable DNS names and hostnames to Pods.
- Maintains Pod identity (name and endpoint) even after restarts, despite IP changes.
- Automatically creates a PVC for each Pod using `volumeClaimTemplates`.
- Uses a **Headless Service** (`clusterIP: None`) for direct DNS-based access to individual Pods.

**Pod Identity**:
| Identifier         | Format                                    | Example                              |
|--------------------|-------------------------------------------|--------------------------------------|
| Pod Name           | `<StatefulSet-name>-<ordinal>`            | `web-0`, `mysql-1`                  |
| Hostname           | Same as Pod name                          | `mysql-0`                           |
| DNS Access         | `<pod>.<service>.<namespace>.svc.cluster.local` | `mysql-0.mysql.default.svc.cluster.local` |
| PVC                | Created via `volumeClaimTemplates`         | `data-mysql-0`                      |

**Related Commands**:
```bash
kubectl get statefulset
kubectl describe statefulset <statefulset-name>
kubectl scale statefulset <statefulset-name> --replicas=<number>
```

**Note**: Whenever pod restarts, IP will be changed but DNS will be retained.

---

## 2.11 Network

- When running a **Deployment**, Kubernetes automatically assigns an IP address to each Pod.
- Kubernetes also automatically creates **environment variables inside the Pod** that store information about Services (including IP and port).
- However, it is **not recommended to rely directly on Pod IPs or environment variables** for communication, since Pod IPs can change when Pods are restarted. Instead, use **Kubernetes internal DNS** for stable service discovery.

**Communication**
- Pods should **communicate with each other through Services**.
- Each Service has an internal DNS address within the cluster, in the format: `<service-name>.<namespace>`.
- Containers within the same Pod share the **same network namespace**, including IP and ports, so they can communicate with each other via `localhost:<port>`.


---

## 3. Writing YAML Files for Kubernetes

Kubernetes resources are defined using **YAML** files, which specify the desired state of objects like Pods, Services, Deployments, and more. These files follow a structured format with key sections, including metadata, spec, and status. This guide explains how to write a Kubernetes YAML file, highlights the **status** section managed by Kubernetes, and addresses updates.

### 3.1 Structure of a Kubernetes YAML File

A typical Kubernetes YAML file includes the following key components:

| Component       | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| **apiVersion** | Specifies the Kubernetes API version for the resource (e.g., `v1`, `apps/v1`). |
| **kind**       | Defines the type of resource (e.g., `Pod`, `Service`, `Deployment`).          |
| **metadata**   | Contains resource metadata, such as `name`, `namespace`, and `labels`.       |
| **spec**       | Defines the desired state of the resource (e.g., container images, ports, replicas). |
| **status**     | Managed by Kubernetes, reflects the current state of the resource. Not user-editable. |

### 3.2 Key Sections Explained

#### 3.2.1 apiVersion and kind

- **`apiVersion`**: Specifies the API group and version used by the resource. It helps identify the version of the Kubernetes API that the object is using. This is part of the versioning process in the Kubernetes API system:
  - `v1` for core resources (e.g., `Pod`, `Service`, `ConfigMap`)
  - `apps/v1` for resources like `Deployment`, `StatefulSet`, `DaemonSet`
  - `batch/v1` for resources like `Job`, `CronJob`
  - `networking.k8s.io/v1` for `Ingress`, `NetworkPolicy`
  - `rbac.authorization.k8s.io/v1` for RBAC resources like `Role`, `ClusterRole`, `RoleBinding`

- **`kind`**: Describes the type of Kubernetes object being defined.
  - Examples: `Pod`, `Service`, `Deployment`, `Ingress`, `ConfigMap`, `Secret`, `Job`


#### 3.2.2 metadata
- Includes:
  - `name`: Unique name for the resource within its namespace.
  - `namespace`: Namespace where the resource resides (defaults to `default` if omitted).
  - `labels`: Key-value pairs for identifying and selecting resources.
  - `annotations`: Additional non-identifying metadata for tools or users.

#### 3.2.3 spec
- Defines the desired configuration, varying by resource type. Examples:
  - **Pod**: Specifies containers, images, ports, and volumes.
  - **Service**: Defines selectors, ports, and type (e.g., ClusterIP, NodePort).
  - **Deployment**: Includes replicas, selector, and Pod template.

#### 3.2.4 status
- Automatically generated and managed by Kubernetes.
- Reflects the current state of the resource and automaticaly update, these resource such as:
  - Pod phase (e.g., `Running`, `Pending`).
  - Service ClusterIP and Endpoints.
  - Deployment conditions (e.g., `Available`, `Progressing`).
- **Note**: Users cannot edit the `status` section directly. Kubernetes updates it based on the cluster's state.

### 4. The `status` Section and Updates

- **Kubernetes-Managed `status`**:
  - The `status` section is populated by Kubernetes to reflect the actual state of the resource.
  - Example: For a Deployment, `status` includes `availableReplicas`, `readyReplicas`, and `conditions`.
  - Users do not include `status` in YAML files when creating or updating resources, as it is overwritten by Kubernetes.

- **Handling Updates**:
  - To update a resource, modify the `spec` section in the YAML file and apply it using:
    ```bash
    kubectl apply -f <filename>.yaml
    ```
  - Kubernetes reconciles the desired state (`spec`) with the current state (`status`).
  - If the `status` does not match the `spec` (e.g., fewer replicas than desired), Kubernetes takes corrective actions (e.g., creating new Pods).
  - Use `kubectl get <resource> -o yaml` to view the current `status`:
    ```bash
    kubectl get deployment nginx-deployment -o yaml
    ```

- **Common Update Scenarios**:
  - **Scaling**: Update `replicas` in the `spec` and apply the YAML.
  - **Image Update**: Change the container `image` in the `spec` for rolling updates.
  - **Configuration Changes**: Modify `spec` fields like environment variables or ports.