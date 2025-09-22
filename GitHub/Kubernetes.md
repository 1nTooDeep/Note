# k8s 项目导航 🐳

这是我所有和 **Kubernetes** 搞对象的项目合集。
从部署脚本到监控工具，从奇怪的运维小实验到“怎么又踩坑了”的记录，都丢在这边。
反正 k8s 本来就够复杂了，留点笔记也算是给未来的自己留一条生路。


## 部署与运维 🚀

一些用来快速开箱、装集群、折腾配置的东西。

### **KubeSphere**
  描述：这玩意是我学习K8S时的第一个工具，对于一个“新手”来说，如果你有KubeKey来帮
  你安装整个K8S集群，那对你的学习是一件很友好的事情，但是KubeSphere现在闭源了，
  官网已经删除了社区版的所有文档和镜像，如果你还在维护一个KubeSphere集群的话，那
  你有得受了（恭喜你，你被KubeSphere给关上了）。

  特性：自动部署、UI友好、兼容多环境、还会帮你少掉几根头发。

  项目地址：👉 [KubeSphere](https://github.com/kubesphere/kubesphere)

---

### **awesome-operators**
  描述：这是一个社区维护的Kubernetes Operators大全。operator是一个资源目录/
  索引。如果你知道你需要什么operator，这个目录是一个很好的参考。这个仓库已经归档
  （archived），不再活跃维护，但仍然很有参考价值。

  特性：收录种类多：数据库、监控、安全、GitOps、存储、数据处理、虚拟化、备份恢复
  等多个方向的 Operators。

  项目地址：👉 [awesome-operators](https://github.com/operator-framework/awesome-operators)

### **postgres-operator**

  描述：这个 Operator 是用来在 Kubernetes 上创建和管理 PostgreSQL 集群的。
  由 Zalando 负责维护。

  特性：使用 CRDs（自定义资源定义）来配置 Postgres 集群，让操作集中、可版本化，
  无需直接操作 Kubernetes API。

  项目地址：👉 [postgres-operator](https://github.com/zalando/postgres-operator)

### **istio**

  描述：个人感觉istio是服务网格（service mesh）最好用的组件之一。istio支持流
  量管理、安全访问、自动搜集等功能。如果加上Kiali，效果会更好。

  特性：功能丰富。

  项目地址：👉 [istio](https://github.com/istio/istio)

---

## 2. 监控与告警 🔔

集群一挂全挂，所以监控是必需品。

### **Prometheus-Operator**

  描述：你在K8S集群中部署一个Prometheus-Operator后，你的集群就可以监控所有你
  想要监控的资源了。你不用自己写各种 Prometheus/Alertmanager 的配置文件，只
  要用它提供的自定义资源（CRDs），它帮你做部署、配置、监控目标发现之类的自动化事
  情。

  特性：自动化监控目标发现（ServiceMonitor、PodMonitor、Probe 等），根据 
  Kubernetes 的 Label/Annotation 自动生成 “scrape 配置”，你不必自己在 
  config 中硬编码服务地址。

  项目地址：👉 [Prometheus-Operator](https://github.com/prometheus-operator/prometheus-operator)

---

## 3. 日志与收集 📜

如果你的容器挂了，你连错误日志都看不见，这个时候你就需要一个日志搜集工具了。


---

## 4. 其他？ ⚙️

### **operator-sdk**
  
  描述：如果你有自己实现 Operator 的需求，Operator-SDK 可能是你需要的一
  个 **“加速器”**。用它你不用从零开始写那些低级 API，也不用每次自己搭骨架
  写 scaffolding（脚手架）。不管是想实现管理状态、反应事件，还是处理故障
  等 Operator 核心能力，借助这个 SDK 都能少折腾不少。
 
  特性：提供高层次 APIs 和抽象，使写 Operator 的业务逻辑更直观。

  项目地址：👉 [operator-sdk](https://github.com/operator-framework/operator-sdk)

---


