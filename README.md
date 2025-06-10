# Kubernetes: Basic Operations and Concepts

## 1. Giới thiệu

Kubernetes là một nền tảng mã nguồn mở để quản lý các ứng dụng được container hóa. Tài liệu này cung cấp hướng dẫn chi tiết về các khái niệm cơ bản như **Node**, **Pod**, **Service**, **Deployment**, và **ReplicaSet**, cùng với các lệnh `kubectl` phổ biến để quản lý chúng. Các lệnh được trình bày kèm ví dụ cụ thể và giải thích rõ ràng.

---

## 2. Các khái niệm cơ bản trong Kubernetes

### 2.1. Node

**Node** là một máy tính (máy vật lý hoặc máy ảo) trong cụm Kubernetes, chịu trách nhiệm chạy các ứng dụng được đóng gói trong container. Mỗi Node chứa các thành phần sau:
- **Kubelet**: Tác nhân chạy trên mỗi Node, đảm bảo các container trong Pod được chạy đúng cách.
- **Kube-proxy**: Quản lý các quy tắc mạng trên Node để kết nối các Pod với nhau và với bên ngoài.
- **Container Runtime**: Phần mềm chạy container (ví dụ: Docker, containerd).

**Lệnh liên quan**:
```bash
kubectl get nodes
```
- **Mô tả**: Liệt kê tất cả các Node trong cụm Kubernetes, hiển thị trạng thái (Ready/NotReady), vai trò (Master/Worker), và thông tin khác như phiên bản Kubernetes.
- **Ví dụ đầu ra**:
  ```
  NAME       STATUS   ROLES    AGE   VERSION
  node1      Ready    worker   10d   v1.28.0
  node2      Ready    worker   10d   v1.28.0
  ```

### 2.2. Pod

**Pod** là đơn vị nhỏ nhất trong Kubernetes, thường chứa một hoặc nhiều container chia sẻ tài nguyên như lưu trữ và mạng. Một Pod đại diện cho một phiên bản của ứng dụng đang chạy.

**Lệnh liên quan**:
```bash
kubectl get pod
```
- **Mô tả**: Liệt kê tất cả các Pod trong namespace hiện tại, hiển thị trạng thái (Running, Pending, Failed), số lần khởi động lại, và thời gian tồn tại.
- **Ví dụ đầu ra**:
  ```
  NAME                     READY   STATUS    RESTARTS   AGE
  nginx-pod                1/1     Running   0          5m
  ```

```bash
kubectl describe pod <pod-name>
```
- **Mô tả**: Cung cấp thông tin chi tiết về một Pod cụ thể, bao gồm trạng thái, sự kiện, và cấu hình.
- **Ví dụ**:
  ```bash
  kubectl describe pod nginx-pod
  ```

```bash
kubectl logs <pod-name>
```
- **Mô tả**: Hiển thị nhật ký (logs) của container trong Pod. Nếu Pod chứa nhiều container, cần chỉ định container cụ thể bằng tùy chọn `--container`.
- **Ví dụ**:
  ```bash
  kubectl logs nginx-pod
  ```

```bash
kubectl exec -it <pod-name> -- /bin/bash
```
- **Mô tả**: Mở một phiên giao tiếp tương tác (interactive terminal) với container trong Pod, cho phép chạy các lệnh bên trong container.
- **Ví dụ**:
  ```bash
  kubectl exec -it nginx-pod -- /bin/bash
  ```
- **Lưu ý**: Nếu container sử dụng shell khác (như `/bin/sh`), thay `/bin/bash` bằng shell tương ứng.

### 2.3. Service

**Service** là một đối tượng Kubernetes định nghĩa cách các Pod được truy cập, thường thông qua một địa chỉ IP hoặc tên DNS nội bộ. Service giúp cân bằng tải (load balancing) và cung cấp khả năng khám phá dịch vụ (service discovery).

Mỗi lần tạo một service thì kubernet sẽ tạo một endppoint 
Service có ip riêng 

**Lệnh liên quan**:
```bash
kubectl get services
```
- **Mô tả**: Liệt kê tất cả các Service trong namespace hiện tại, hiển thị loại Service (ClusterIP, NodePort, LoadBalancer), địa chỉ IP, và cổng.
- **Ví dụ đầu ra**:
  ```
  NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  nginx-svc    ClusterIP   10.96.0.10   <none>        80/TCP    2d
  ```

### 2.4. Deployment

**Deployment** là một bản thiết kế (blueprint) để quản lý các Pod và **ReplicaSet**. Deployment đảm bảo rằng một số lượng Pod nhất định luôn chạy, tự động xử lý việc tạo, cập nhật, và xóa Pod khi cần.

**Quan hệ**: Deployment → ReplicaSet → Pod → Container
- **Deployment**: Quản lý ReplicaSet và định nghĩa trạng thái mong muốn của ứng dụng.
- **ReplicaSet**: Đảm bảo số lượng Pod được chỉ định luôn chạy.
- **Pod**: Chứa các container thực thi ứng dụng.
- **Container**: Chứa ứng dụng thực tế.

**Lệnh liên quan**:
```bash
kubectl get deployment
```
- **Mô tả**: Liệt kê tất cả các Deployment trong namespace hiện tại, hiển thị số lượng bản sao (replicas), trạng thái, và thời gian tồn tại.
- **Ví dụ đầu ra**:
  ```
  NAME            READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-deploy    3/3     3            3           1h
  ```

```bash
kubectl create deployment <name> --image=<image> [--dry-run]
```
- **Mô tả**: Tạo một Deployment mới với tên và hình ảnh container được chỉ định. Tùy chọn `--dry-run` cho phép kiểm tra cấu hình mà không thực sự tạo.
- **Ví dụ**:
  ```bash
  kubectl create deployment nginx-deploy --image=nginx
  ```
  - Tạo một Deployment có tên `nginx-deploy` sử dụng hình ảnh `nginx` từ Docker Hub.

```bash
kubectl edit deployment <name>
```
- **Mô tả**: Mở trình chỉnh sửa (ví dụ: `vi`) để sửa đổi cấu hình của Deployment trực tiếp. Điều này hữu ích để thay đổi số lượng bản sao, hình ảnh container, hoặc các thông số khác.
- **Ví dụ**:
  ```bash
  kubectl edit deployment nginx-deploy
  ```

```bash
kubectl delete deployment <name>
```
- **Mô tả**: Xóa một Deployment và các tài nguyên liên quan (ReplicaSet và Pod).
- **Ví dụ**:
  ```bash
  kubectl delete deployment nginx-deploy
  ```

### 2.5. ReplicaSet

**ReplicaSet** đảm bảo rằng một số lượng Pod nhất định luôn chạy tại mọi thời điểm. Thông thường, ReplicaSet được quản lý tự động bởi Deployment, nên bạn hiếm khi cần tạo hoặc xóa ReplicaSet trực tiếp.

**Lệnh liên quan**:
```bash
kubectl get replicaset
```
- **Mô tả**: Liệt kê tất cả các ReplicaSet trong namespace hiện tại, hiển thị số lượng bản sao mong muốn, hiện tại, và trạng thái.
- **Ví dụ đầu ra**:
  ```
  NAME                     DESIRED   CURRENT   READY   AGE
  nginx-deploy-abc123      3         3         3       1h
  ```

**Lưu ý**: Không cần tạo hoặc xóa ReplicaSet thủ công vì chúng được quản lý bởi Deployment.

### 2.6. Áp dụng cấu hình từ tệp YAML

Kubernetes hỗ trợ quản lý tài nguyên thông qua các tệp cấu hình YAML hoặc JSON. Điều này cho phép định nghĩa trạng thái mong muốn của các đối tượng như Deployment, Service, hoặc Pod.

**Lệnh liên quan**:
```bash
kubectl apply -f <config-file.yaml>
```
- **Mô tả**: Áp dụng cấu hình từ tệp YAML hoặc JSON, tạo hoặc cập nhật các tài nguyên tương ứng.
- **Ví dụ**:
  ```bash
  kubectl apply -f nginx-deployment.yaml
  ```
- **Ví dụ nội dung tệp `nginx-deployment.yaml`**:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deploy
  spec:
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
  ```

---

## 3. Quy trình làm việc cơ bản

Dưới đây là quy trình cơ bản để triển khai và quản lý một ứng dụng trong Kubernetes:
1. **Tạo Deployment**:
   ```bash
   kubectl create deployment nginx-deploy --image=nginx
   ```
2. **Kiểm tra trạng thái Deployment**:
   ```bash
   kubectl get deployment
   ```
3. **Kiểm tra Pod**:
   ```bash
   kubectl get pod
   ```
4. **Kiểm tra nhật ký**:
   ```bash
   kubectl logs nginx-deploy-<pod-id>
   ```
5. **Truy cập Pod**:
   ```bash
   kubectl exec -it nginx-deploy-<pod-id> -- /bin/bash
   ```
6. **Chỉnh sửa Deployment** (ví dụ: thay đổi số lượng bản sao):
   ```bash
   kubectl edit deployment nginx-deploy
   ```
7. **Xóa Deployment**:
   ```bash
   kubectl delete deployment nginx-deploy
   ```

---


Selector for deployment and for service is diff 
Service for rute traffic 
Deployment for management 

port: 80 là cổng mà Service lắng nghe trong cluster.
targetPort: 8080 là cổng mà các Pod đang chạy thật sẽ lắng nghe bên trong (trong container).
Vì vậy, Service nhận request ở cổng 80, rồi chuyển tiếp (proxy) nó tới các Pod có label phù hợp, vào cổng 8080 của chúng.

containerPort = targetPort 

Default service is kubernetes:
- Là dịch vụ mặc định của API server trong cluster. Nó cho phép các pod giao tiếp với API server để điều khiển cluster.

kubectl describe service nginx-service
kubectl get pod -o wide
-> check endpoint of service and IP of pod 

Check status 
kubectl get deployment nginx-deployment -o yaml

Mỗi pod sẽ tạo 2 container: 1 cái main và 1 cái pause để giữ namespace 
Một pod có thể có nhiều hơn container nếu dùng theo 
- Sidecar container Ví dụ: chạy một container ứng dụng + một container Fluentd để đẩy log lên server.
- Init container Các container khởi tạo chạy trước khi container chính bắt đầu (ví dụ: tải dữ liệu, thiết lập config...).
- Ambassador / Adapter container Đóng vai trò như cổng giao tiếp hoặc chuyển đổi giao thức.


minikube service NAME 

## namespace 

3 system namespace 

advoice conflict with other teams 
advoice conflict name of resosurce 
group the resource for management 
seperate many stage environment 
access and limit the resource for each team  
share resource between diffirent environment 
-> some resource lives globaly withoout namespace 
-> without providing namespace -> use default namespace 

Một Namespace có thể có các Pod chạy trên nhiều Node khác nhau.


## ingress 
Ingress là cổng chính cho các HTTP request từ ngoài vào, như một web server reverse proxy.
Ingress resource
Ingress controller 
Ingress = Cách chuẩn trong Kubernetes để thay thế việc tự cài NGINX thủ công.
default backend -> create another pod with the same name to handle error rule request 
using URL to make diffirect app accessable -> can use many host instead 

kubectl get ingress -n kubernetes-dashboard
kubectl describe ingress <name> -n <namespace>

## Helm
clone common yaml configuration file
define template file with value yaml file
release management

## Volume 
persistent volume Là tài nguyên lưu trữ được quản lý bởi admin hoặc provisioned tự động.

PersistentVolumeClaim Là một yêu cầu lưu trữ từ người dùng hoặc ứng dụng. Người dùng sẽ yêu cầu, sau đó PVC sẽ tìm PV phù hợp rồi thực hiện bind. Mục đích là tách biệt người dùng với hạ tầng lưu trữ, tại sử dụng lại các tài nguyên hiệu quả 
Pod request volum through PVC -> PVC try to find PV 
PVC phải chung namespace với pod 

Volume sẽ được mount vào trong filesystem của container 
Có thể mount configmap và secretkey như volume 

StorageClass định nghĩa "cách" mà Kubernetes sẽ tạo ra một PersistentVolume (PV) một cách tự động, gọi là dynamic provisioning.

## StatefullSet 
StatefulSet trong Kubernetes là một loại workload controller đặc biệt dùng để chạy các ứng dụng có trạng thái (stateful applications), tức là ứng dụng cần lưu trữ dữ liệu lâu dài, định danh ổn định, hoặc thứ tự khởi chạy cụ thể.

individual dns for each pod
when pod restart -> IP change -> name and endpoint stay same 
pod can retain state and role 

kubernets sẽ tạo pvc cho từng pod 

Tóm lại: Định danh Pod trong StatefulSet được tạo thế nào?
Định danh	Cách tạo	Ví dụ
Tên Pod	<StatefulSet name>-<số thứ tự>	web-0, mysql-1
Hostname	Giống tên Pod	mysql-0
DNS truy cập	<pod>.<svc>.<namespace>.svc.cluster.local	mysql-0.mysql.default.svc.cluster.local
PVC riêng biệt	Tự động từ volumeClaimTemplates	data-mysql-0

headless service giúp truy cập dns cố định 



## K8s service 

ClusterIP service: 
- default type 
- ...

Headless service 
- Client want to communicate with specific pod directly, not randomly selected 
- use in statefull application like database 

NodePort 
- Mở một port cho node để traffic bên ngoài có thể vào \

LoadBalancer is an extension of nodeport 
- cân bằng tải và chỉ sử dụng một IP tĩnh duy nhất 

## Note: 
virtual network in minikube, check ip through /etc/cni 
using brigde network 
