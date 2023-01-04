- [K8s进阶篇-细粒度权限控制](#k8s进阶篇-细粒度权限控制)
  - [1.1 RBAC简介](#11-rbac简介)
  - [1.2 RBAC配置解析](#12-rbac配置解析)
  - [1.3 RBAC 常用配置示例](#13-rbac-常用配置示例)
  - [1.4 聚合ClusterRole](#14-聚合clusterrole)
  - [1.5 RBAC 实践](#15-rbac-实践)
  - [1.6 RBAC企业实战：如何管理权限的用户](#16-rbac企业实战如何管理权限的用户)
  - [1.7 RBAC企业实战：不同用户不同权限](#17-rbac企业实战不同用户不同权限)

### K8s进阶篇-细粒度权限控制 ###

---

#### 1.1 RBAC简介 ####

负责K8s整个集群的控制

+ 管理员：可以使用kubectl针对k8s做任何想做的事情！

+ 其他人员：开发，测试、运维等

:deciduous_tree: 权限控制（在生产环境中是需要用RBAC做细粒度的权限）

+ 防止对资源误删
+ 有其他的工具，但是主推RBAC



:deciduous_tree: RBAC介绍：

 ![image-20221228180158355](images/image-20221228180158355.png)

+ ClusterRole：对整个集群做想做的事，作用于整个集群
+ ClusterRolebinding:ClusterRole配置权限授权给
  + user
  + group
  + serviceAccount
+ Role：对该 namespace 做想做的事
+ RoleBinding：指定的 “人” 对该namespac做想做的事情
  + user
  + group
  + serviceAccount

:warning: 有重复的 Role 存在的话，可以列到ClusterRole。 

没有拒绝的操作，只有附加



#### 1.2 RBAC配置解析 ####

官方文档：https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/

启用RBAC一般默认是打开的：

```SH
# cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep RBAC
    - --authorization-mode=Node,RBAC
```

:deciduous_tree: role 示例

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:	# 配置具体的权限
- apiGroups: [""] # "" 标明 core API 组，对那些资源限制，资源属于哪个组，如deployment 的apiVersion：apps/v1 就是属于 apps 组;
  resources: ["pods"]
  verbs: ["get", "watch", "list"]	# 定义执行的操作
```

:deciduous_tree: ClusterRole 示例

+ 与 Role 相比 不需要 namepace ，Kind 不同；其他配置都一样

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" 被忽略，因为 ClusterRoles 不受名字空间限制
  name: secret-reader
rules:
- apiGroups: [""]
  # 在 HTTP 层面，用来访问 Secret 资源的名称为 "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

一般只写：  resources 和 verbs 配置都很简单；



:deciduous_tree: RoleBinding 和 ClusterRoleBinding

RoleBinding 示例

```YAML
apiVersion: rbac.authorization.k8s.io/v1
# 此角色绑定允许 "jane" 读取 "default" 名字空间中的 Pod
# 你需要在该命名空间中有一个名为 “pod-reader” 的 Role
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# 你可以指定不止一个“subject（主体）”
- kind: User
  name: jane # "name" 是区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" 指定与某 Role 或 ClusterRole 的绑定关系
  kind: Role        # 此字段必须是 Role 或 ClusterRole
  name: pod-reader  # 此字段必须与你要绑定的 Role 或 ClusterRole 的名称匹配
  apiGroup: rbac.authorization.k8s.io
```

RoleBinding 也可以引用 ClusterRole, 这种引用使得你可以跨整个集群定义一组通用的角色， 之后在多个名字空间中复用。

ClusterRoleBinding 示例:

+ 无 namepace

```YAML
apiVersion: rbac.authorization.k8s.io/v1
# 此集群角色绑定允许 “manager” 组中的任何人访问任何名字空间中的 Secret 资源
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager      # 'name' 是区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```



:warning: 在生产环境中推荐使用 sa （serviceaccounts ） 去进行访问集群资源（安全可靠）

+ 不需要重启 apiservice，对于配置用户可能需要重启才生效





#### 1.3 RBAC 常用配置示例

https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/

允许读取在核心 API 组下的 `"pods"`：

```SH
rules:
- apiGroups: [""]
  # 在 HTTP 层面，用来访问 Pod 资源的名称为 "pods"
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

命令行：

+ 一般使用 yaml 创建

```yaml
# kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods --dry-run=client -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: pod-reader
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```

创建 `rolebinding` 和 `clusterrolebinding`

使用命令行简单一点

```SH
# kubectl create rolebinding bob-admin-binding --clusterrole=admin --user=bob --namespace=acme --dry-run=client -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: bob-admin-binding
  namespace: acme
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: bob

Kubectl create clusterrolebinding root-cluster-admin-binding --clusterrole=cluster-admin --user=root
kubectl create clusterrolebinding myapp-view-binding --clusterrole=view --serviceaccount=acme:myapp
```



#### 1.4 聚合ClusterRole

你可以将若干 ClusterRole **聚合（Aggregate）** 起来，形成一个复合的 ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"	# 聚合具有这个标签的ClusterRolerole
rules: [] # 控制面自动填充这里的规则
```

再创建一个具有 rbac.example.com/aggregate-to-monitoring: "true" 的 ClusterRole

然后创建一个clusterrolebinding 



#### 1.5 RBAC 实践

1. 创建一个名为 deployment-clusterrole 的 clusterrole

+ 该 cluster 只允许创建Deployment、Daemonset、Statefulset

2. 在名字为 app-team1 的 namespace 下创建一个名为 cicd-token 的 serviceAccount，并将上一步创建的 clusterrole 的权限绑定到该 sa

结果：cicd-token 只具有 创建 Deployment、Daemonset、Statefulset 资源的权限

:deciduous_tree: 创建 namespace 和 sa

```SH
kubectl create ns app-team1
kubectl create sa cicd-token -n app-team1
```

:deciduous_tree: 创建 clusterrole

```SH
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployment-clusterrole
rules:
- apiGroups: ["apps"]
  resources: ["deployments","daemonsets","statefulsets"]
  verbs: ["create"]
```

无namepace 限制

```SH
kubectl create -f dp-clusterrole.yaml
# kubectl get clusterrole deployment-clusterrole -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2022-12-28T12:14:43Z"
  name: deployment-clusterrole
  resourceVersion: "2421320"
  uid: a5a296dc-0017-4696-b00c-465794edd425
rules:
- apiGroups:
  - apps
  resources:
  - deployments
  - daemonsets
  - statefulsets
  verbs:
  - create
```

:deciduous_tree: clusterrole 的权限绑定到该 sa

创建RoleBinding 

```yaml
kubectl create rolebinding deployment-rolebinding --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token --namespace=app-team1

# kubectl get -n app-team1 rolebindings.rbac.authorization.k8s.io deployment-rolebinding -oyaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2022-12-28T12:21:15Z"
  name: deployment-rolebinding
  namespace: app-team1
  resourceVersion: "2422047"
  uid: a192fb50-5550-48d7-b2f2-051530e6a238
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: deployment-clusterrole
subjects:
- kind: ServiceAccount
  name: cicd-token
  namespace: app-team1
```

查看token

```SH
# kubectl get secrets -n app-team1
NAME                     TYPE                                  DATA   AGE
cicd-token-token-klhzw   kubernetes.io/service-account-token   3      20m
default-token-vnwj4      kubernetes.io/service-account-token   3      21m
# kubectl describe -n app-team1 secrets cicd-token-token-klhzw
最后的token 就是登陆的token
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjZFWVN6SUZRMkJ6N1VqQ25MaC1sRG5odkM0UTBCZWZUYjJRejN2N3FBOEUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJhcHAtdGVhbTEiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiY2ljZC10b2tlbi10b2tlbi1rbGh6dyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJjaWNkLXRva2VuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYjdhOTJlYzItN2VjMC00YTI3LTk1MWEtZTUxMGZlNzNmZWFiIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmFwcC10ZWFtMTpjaWNkLXRva2VuIn0.ERE2dqFivE4hwHVuoxR9_gHesixpawXqNdywudlP0dZIowNdaylXZsmqFDzYJHANJ9j3QMq8N2j_icH_XQAqUd9ETQZCf9VDyH5LeofCkzS16_v0xynvaloGhdHep3C99am_SdxV64Z1MFdTraHuFvrssbt5FuFD9Cn6k7pNMHAr6W14tHaRDqvMj_g-GUOJxCDb0PGzFSmwou_rzp1DV0PPUcM7ag759MFqwD2_yTKiELJpWFVV5sAwFF-Rh8NvbkL34QAZzFQAiho3HegapdlwEh5a-TqOtjbjnAy6j1D_QsOkHEJAjL0uJjqJqG2hlPnrH4SSXzEuUk0D3kA6yA
```



#### 1.6 RBAC企业实战：如何管理权限的用户

在企业中使用RBAC还有很多的细节和经验

用起来和很好的用起来还是有些区别的，用户很多管理不好可能会很乱

:deciduous_tree:用户很多如何更好的管理

 ![image-20221229165141571](images/image-20221229165141571.png)

 

 通用权限（做为ClusterRole）

+ Namespace 列表查看权限
+ 日志查看权限
+ 执行命令权限
+ Pod 删除权限
+ 资源编辑权限
+ 其他权限

:deciduous_tree: 用户管理

 ![image-20221230114727793](images/image-20221230114727793.png)

使用 sa

+ 错误做法：每个namespace 创建 一个 sa

正确做法：

+ 创建一个 namespace 专门放置用户，所有用户都创建一个 amespace 下
+ 权限都绑定在 namespace 某个用户上，用户就能操作其他namespace（对很多个 namespace 访问）



#### 1.7 RBAC企业实战：不同用户不同权限

需求：

1. 用户 tom 可以查看 default、kube-system 下的 Pod 的 日志
2. 用户jack 可以在 default 下的 Pod中执行命令，并且可以删除Pod

执行命令、删除Pod、查看日志都是通用权限，所以可以使用ClusterRole

:deciduous_tree:删除Pod

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-delete
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["delete","get","list"]
```

:deciduous_tree:执行命令

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-exec
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create"]
```

:deciduous_tree:查看日志

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-log
rules:
- apiGroups: [""]
  resources: ["pods","pods/log"]
  verbs: ["get", "watch", "list"]
```

:deciduous_tree: namespace 读权限

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: namespace-readonly
rules:
- apiGroups: [""]
  resources: ["pods","namespaces"]
  verbs: ["get", "watch", "list"]
```

创建以上四个：

```SH
# kubectl get clusterrole
NAME                                                                   CREATED AT
namespace-readonly                                                     2023-01-01T01:48:15Z
pod-delete                                                             2023-01-01T01:50:15Z
pod-exec                                                               2023-01-01T01:50:15Z
pod-log                                                                2023-01-01T01:50:15Z

```

:deciduous_tree:用户管理

创建用户管理的命名空间、用户

```SH
kubectl create ns kube-users
kubectl create sa tom -n kube-users
kubectl create sa jack -n kube-users
# kubectl get sa -n kube-users
NAME      SECRETS   AGE
default   1         12s
jack      1         11s
tom       1         12s
# kubectl get secrets -n kube-users
NAME                  TYPE                                  DATA   AGE
default-token-4n9gr   kubernetes.io/service-account-token   3      47s
default-token-mwsjw   kubernetes.io/service-account-token   3      47s
jack-token-rmgkc      kubernetes.io/service-account-token   3      46s
jack-token-zpcjw      kubernetes.io/service-account-token   3      46s
tom-token-h258l       kubernetes.io/service-account-token   3      47s
tom-token-k7p2t       kubernetes.io/service-account-token   3      47s
```

查看tom 的 token

```SH
# kubectl describe -n kube-users secrets tom-token-h7xdt
Name:         tom-token-h7xdt
Namespace:    kube-users
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: tom
              kubernetes.io/service-account.uid: 65ff04ac-6cee-4e83-8a78-7cdec4d47f49

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1099 bytes
namespace:  10 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjZFWVN6SUZRMkJ6N1VqQ25MaC1sRG5odkM0UTBCZWZUYjJRejN2N3FBOEUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXVzZXJzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InRvbS10b2tlbi1oN3hkdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJ0b20iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2NWZmMDRhYy02Y2VlLTRlODMtOGE3OC03Y2RlYzRkNDdmNDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS11c2Vyczp0b20ifQ.LVE4K5aRZec3lUPaWncl7HrTv5GgQ4S6iWZR8l_YpxE46ug1UYL-vL5_mkQigpFK2vSGOf0GqwFjkYjWKO07oy0Jsn0sFoxIfbLMmVFnJ4yeZS8NLij1C4PfJukx2WZT4UuOUm9qTe9Q_InwZkwwCm8_v0jM_frgGaPdi4RD0u4VVgFs9IMPBs9VyHaSLRqhVE72aRPmr_CSGoXMJXHhdrgEaVBwxMBbyZI2iunWZO_i9fOkHJ1J7XG7Y53KmRxcbpKwKKuRM6vvX7QVwiqhAUi5afYUYSuL1ncVl6cIGFyLv4ysuhTkv0yl0mY-EZYJhR9cfatstDvVpxHdbvQPBA
```

先授权：绑定全局命名空间查看权限

说明：system:serviceaccounts （复数）是用于服务账户组名的前缀。

+ system:serviceaccounts:kube-users   对于kube-users 命名空间下所有用户

```SH
# cat namespace-readonly-sa.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: namespace-readonly
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: namespace-readonly
subjects:
- kind: Group
  name: system:serviceaccounts:kube-users
  apiGroup: rbac.authorization.k8s.io
```

:deciduous_tree:创建完成后，用户就有了切换 namespace的权限

但是没有 查看Pod 等其他权限 ![image-20230104120212749](images/image-20230104120212749.png)



> 用户 tom 可以查看 default、kube-system 下的 Pod 的 日志

绑定权限

```SH
kubectl create rolebinding tom-pod-log --clusterrole=pod-log --namespace=default --serviceaccount=kube-users:tom
kubectl create rolebinding tom-pod-log --clusterrole=pod-log --namespace=kube-system --serviceaccount=kube-users:tom
```

> 用户jack 可以在 default 下的 Pod中执行命令，并且可以删除Pod

绑定权限

```SH
kubectl create rolebinding jack-pod-exec --clusterrole=pod-exec --namespace=default --serviceaccount=kube-users:jack
kubectl create rolebinding jack-pod-delete --clusterrole=pod-delete --namespace=default --serviceaccount=kube-users:jack
```

