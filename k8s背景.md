云原生技术包括：容器、服务网格、微服务、不可变的基础设施和声明式API

docker底层两个最关键的技术

+ cGroup：限制内存、CPU（用作资源限制）
+ Namespace：用作资源隔离
  + PID 隔离进程
  + UTS 主机名隔离
  + IPC 通信隔离
  + Network 网络隔离
  + Mount 存储隔离
  + User 用户隔离，安全性提升

+ 联合文件系统



k8s对于每个对象基本都会提供一个控制器去控制



  Pod为单位，共同管理底层的Pod

+ 无状态模式
+ 有状态模式
+ 守护进程模式
+ 批处理模式



通过label selector选择需要管理的资源对象



deploymen: 提供了声明式、自定义策略的Pod升级支持

> 重在：升级

deployment不是直接管理Pod，而是通过管理RS，RS去管理Pod

+ 升级回滚
+ 暂停恢复：rollout
+ 弹性、按比例扩缩容 与HPA联动



有状态：典型应用

+ Zookeeper、MongoDB、etcd、mysql
+ 有身份的（三大要素）
  + 域名
  + PVC存储
  + Pod name
+ 严格的启动顺序
