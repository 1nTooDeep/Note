# NVIDIA-OPERATOR

## nvidia-operator主要动作

### Helm 安装与资源渲染

```bash
helm install nvidia-operator nvidia/gpu-operator \
  -n gpu-operator --create-namespace
```

- Helm 渲染 Chart 模板，创建以下资源：
  - **CustomResourceDefinition (CRD)**：`ClusterPolicy` 等自定义资源定义
  - **RBAC**：`ClusterRole`、`RoleBinding`、`ServiceAccount`
  - **Deployment**：`gpu-operator` 控制器
  - **CustomResource**：`ClusterPolicy`（核心配置对象）

--------------

#### 1. Operator 控制器启动

- 部署 `gpu-operator` Deployment 并启动控制器 Pod
- 控制器开始监听 `ClusterPolicy` 的创建与变更

---------------

#### 2. Node Feature Discovery（NFD）：首要步骤

- **优先部署** `gpu-feature-discovery` DaemonSet

- NFD 扫描节点硬件，自动打上 GPU 相关标签，例如：

  ```bash
  nvidia.com/gpu.present=true
  nvidia.com/cuda.driver.major=515
  ```

- **后续所有 DaemonSet**（Driver、Toolkit、DevicePlugin 等）通过这些标签进行 nodeSelector 调度

- 若在 Helm 值中禁用 NFD（`nfd.enabled=false`），需手动为节点添加相应标签

------------------------

#### 3. 核心组件部署

##### 3.1 驱动安装与验证

- **默认模式**（`driver.enabled=true`）：
  - 部署 `nvidia-driver-daemonset`，在每个带 GPU 节点上安装或验证驱动
  - 若节点已有兼容驱动，仅执行版本/内核兼容性检查
- **预装模式**（`driver.enabled=false`）：
  - 跳过驱动安装，仅部署轻量验证容器，确认驱动可用

##### 3.2 容器运行时配置（Toolkit）

- 部署 `nvidia-container-toolkit-daemonset`

- 在每个 GPU 节点上执行（以 containerd 为例）：

  1. 生成或修改 `/etc/containerd/config.toml`，注入 NVIDIA runtime：

     ```toml
     [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
       runtime_type = "io.containerd.runc.v2"
       privileged_without_host_devices = false
       [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
         BinaryName = "/usr/bin/nvidia-container-runtime"
     ```

  2. 重启 containerd：

     ```bash
     systemctl restart containerd
     ```

- 若使用 Docker，则修改 `/etc/docker/daemon.json` 加入 `"default-runtime": "nvidia"`，并重启 Docker

##### 3.3 Device Plugin 注册

- 部署 `nvidia-device-plugin-daemonset`

- 向 Kubelet 报告 GPU 资源，完成后可通过：

  ```bash
  kubectl describe node <gpu-node> | grep nvidia.com/gpu
  # Capacity: nvidia.com/gpu: 1
  ```

##### 3.4 Fabric Manager & DCGM Exporter（可选）

- **Fabric Manager**：管理 NVLink/MIG
- **DCGM Exporter**：采集 GPU 指标，配合 Prometheus

-----------------

#### 4. 状态汇总与就绪

- 控制器持续监控各 DaemonSet 与 Pod，当所有组件就绪后更新 `ClusterPolicy.status`

- 最终可通过以下命令确认状态：

  ```bash
  kubectl get clusterpolicy -n gpu-operator
  # State: Ready
  ```

- 此时，GPU 已完整注册，可调度运行 GPU 工作负载

------

## 常见故障与排查

- **无 GPU 标签**：检查 `gpu-feature-discovery` Pod 日志或手动打标签
- **驱动安装失败**：查看 `nvidia-driver-daemonset` Pod 日志
- **运行时无效**：通过 `journalctl -u containerd` 或 `docker logs` 检查
- **权限/污点问题**：确保 Pod 有正确的 tolerations 与相应的 RBAC 权限

```bash
kubectl get pods -n gpu-operator
kubectl describe pod <pod-name> -n gpu-operator
kubectl logs <operator-pod> -n gpu-operator
```