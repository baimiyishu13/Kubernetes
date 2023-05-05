如果一个人真的无所事事的时候，开始学习，哪怕是一些目前用不到的东西。培养自己学习的积极性，坚持两个月，你就会发现自己脱离了手机，这个时候不要迷茫，不要觉得自己有点成就感就懈怠，坚持下去，这个阶段你的游戏好友、酒肉朋友和你会疏离。但心中的目标要坚定，警醒自己努力暗示自我，接下来将涅槃重生！

[TOC]



# DevOps #

## 第一章 基于K8s的DevOps平台 ##

### 前言 ###

k8s服务部署问题：==微服务越来越多==

如果手动管理：发版（前期几十个或者几个）还可以处理。但是多了以后会遇到一个问题

+ 手动操作、发版：占用很大的人力。
+ ==利用DevOps提高工作效率，降低复杂度==

DevOps技术栈：

涉及到：

1. 主角Jenkins CI界的大佬，虽然出现了很多种的CI工具，但是也未能撼动Jenkins的地位。
2. docker、k8s
3. 镜像仓库：HaRBOR
4. 代码管理：GitLab

###  介绍：DevOps ###

		概念：DevOps是Development和Operations的组合。是一种重视开发人员（Dev）和运维人员（Ops）之间沟通合作的文化、协作和整合。通过自动化“软件交付”和架构变更的流程，来使得构建、测试、发布软件能够更加快捷、频繁和可靠。

DevOps概念层次不齐

+ 是一种重视开发和运维之间的文化的组合。简化之间的沟通
+ 主要用途：提高工作效率、发布应用的频率

==随着发展，DevOps已经不仅仅是开发和运维的桥梁==

开发、技术运营、QA、OPS部门直接沟通、协作与整合，通过一系列自动化工具来完成软件的生命周期管理。

+ 通过自动化工具

**重要环节CI/CD**

> CI/CD 和 DevOps是不化等号的。

有人说CI/CD 就是 DevOps的使用，其实是不对的

+ CI/CD只是DevOps的一个环节，DevOps涵盖的太多了。不仅仅是技术方面的还有文化方面的。

CICD概念： 

		CI/CD是一种通过在应用开发阶段引入自动化工具，来频繁向客户交付应用的方法，主要针对在集成新代码时所引发的问题。CICD时DevOps中尤为重要的一个环节。

> 集成新代码时很方便的定位代码引发的问题是什么，本身的问题、方案是什么？

CI/CD主要分为了三个方面

1. 持续集成：属于开发人员的自动化流程 - 帮助开发人员更加频繁的将代码更改合并到共享分支或主干中。
2. 持续交付：每个阶段都涉及测试自动化、代码发布自动化。在流程结束时，运维团队可以快速、轻松地将应用部署到生产环境中。**目标：持续交付的目的是拥有一个可随时部署到生产环境的代码库**
3. 持续部署：对于一个成熟的CI/CD管道来说，最后的阶段是持续部署。作为持续交付（自动将生产就绪型构建版本发布到代码存储库）的延申，持续部署可以自动将应用发布到生产环境中。**持续部署意味着开发人员对应用的更改在编写后的几分钟内就能生效，者更加便于持续接收和整合用户反馈** ==就是将持续交付的结果，自动发布到生产环境==



部署：手动、自动（持续部署）

注意：自动发布可能比较难的，对生产环境需要有敬畏之心，能不动就不动。不能随便将一个东西扔到生产环境。==所以一般在生产环境还市手动部署而且可能还需要审核==

持续部署：

+ 大公司做的好，向谷歌（一天或者一次发布上万次），手动部署肯定是无法接收的
+ 非常烧钱（大部分公司没有自动发布化平台）



### 1.1 什么是流水线 ###

#### 1.1.1 流水线 ####

CICD：集成、交付、部署

最终目标：将代码部署到某个产品，供别人使用（与生产电子产品线很像）

+ 提交：开发提交代码
+ 编译：镜像
+ 部署：产物部署到环境

整个过程有很多小小的单元，合在一起！

每个单元只负责自己的事情，将复杂的事情分成很多的单元。



#### 1.1.2 Jenkins 流水线入门 ####

流水线：Pipeline

> 非常灵活，可以实现企业大部分的需求。

Jenkins流水线（两种方式）

1. 声明式流水线（主推）：直接使用声明步骤实现逻辑，==可以直接使用图像化界面生成==
2. 脚本式流水线：需要些Grooay语法，相对比较复杂，需要实现逻辑。==以node开头（在任何的代理上执行流水线）==

流水线清晰明了，可以将每个过程单独列出来，比如

+ Build 构建过程
+ Test 测试过程
+ Deploy 部署过程

```yaml
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```



**agent**

整个流水线运行在哪里，代理节点或者非代理（指令和阶段都可以指定）

+ agent any ：可以运行在任意的代理节点上
  agent none ：不设置节点，每个阶段自己指定
  agent {label “label”} ：选择带有标签的代理节点
  agent {docker “image”} ：基于docker运行流水线

**stages** ：代表阶段的一个整体块

+ stage：单个阶段，格式：stage(名称)，这个名称会显示在流水线流程图上
  steps ：实际要执行的DSL步骤，支持echo、git、sh、mail等等有效语句，支持条件结构（与、或、非）
  post ：可选项，同时可以在阶段和流水线中使用

**stage解释**

在stages内部有一个或者多个单独的阶段，都至少包含了一个或者多个DSL步骤，还可以引入指令定义一些变量、参数，全局变量等等

> jenkins大部分都可以通过自带的工具生成代码



### 1.2 声明式Pipeline语法 ###

声明式流水线必须包含在一个Pipeline块中，比如以下是一个Pipeline块的格式：

```groovy
pipeline {

}
```

在声明式流水线中有效的基本语句和表达式遵循与Groovy的语法同样的规则，但有以下例外：

+ 流水线顶层必须是一个block，即 pipeline {} ；
+ 分隔符可以不需要号 ，但是 每条语句都 必须 在自己的行上；
+ 块只能由 块只能由 Sections 、Directive 、Steps或 assignment statements组成 ；
+ 属性引用语句被当做是无参数的方法调用，比如input会被当做input（）。

#### 1.2.1 Section ####

	声明式流水线中的Sections不是一个关键字或指令，而是包含一个或多个Agent、Stages、post、Directives和Steps的代码区域块。

##### 1.2.1.1 Agent #####

		Aggent 表示整个流水线或特定阶段中的步骤和命令执行的位置，该部分必须在pipeline块的顶层被定义，也可以在stage中再次被定义，但是stage级别是可选的。

+ any：在任何可用的代理上执行流水线，配置语法

```groovy
pipeline{
	agent any
}
```

+ none：表示该pipeline脚本没有全局的agent配置，当顶层的agent配置为none时，每个stage部分都需要包含自己的agent。配置方法：

```groovy
pipeline{
	agent none
    stage {
        stage('Stage Fro Build'){
            agent any
        }
    }
}
```

+ lanel：选择将某个具体的节点执行Pileline命令，例如：agent { label ’my-default-label‘ } 配置语法：

注意：Jenkins是主从架构，node节点可以打上标签，通过标签实现选择具体节点选择pipeline节点

```groovy
pipeline{
	agent none
    stage {
        stage('Stage Fro Build'){
            agent { label 'MY-SLAVE-LABEL' }
        }
    }
}
```

+ node：和label配置类似，只不过是可以添加一些额外的配置，比如 customWorkspace；

node就是直接指定某一个节点去执行流水线，和label不同，直接使用node名称，有额外的配置工作目录！

+ dockerfile：使用从源码中包含的Dockerfile 所构建的容器执行流水线或stage。此时对应的agent写法如下

**注意**：一般用的不是很多，流水线就是要求做的快。一套下来时间不能太长，==一般使用docker==

```groovy
agent {
    dockerfile {
        filename 'dockerfile.build'
        dir 'build'
        label 'my-defned-label'
        additionalBuildArgs '--build-arg version=1.0.2'
    }
}
```

+ docker：相当于dockerfile，可以直接使用docker字段指定外部镜像即可，可以省去构建的时间，比如使用maven镜像进行打包，同时可以指定args：

**注意：**非常常用的选项,，镜像-一般都在公司的私有仓库，适用于静态Slave

```groovy
agent{
    docker{
        image：'maven:3-alpine'
        label 'my-defined-label'
        args '-v /tmp:/tmp'
    }
}
```

比docker更好的一个选择是k8s

+ kubernetes：Jenkins也支持使用Kubernetes创建Slave，也就是常说的动态Slave。配置示例如下

**注意**：

1. 节点宕机不会影响slave，静态需要手动去选择另一个节点
2. 动态：其实也不是非常动态，通常是固定几个节点去使用
3. yaml：jenkins会控制k8s创建Pod

```groovy
agent {
    kubernetes {
        label podlabel
        yaml """ 
kind: Pod 
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
 """
    }
}
```

##### 1.2.1.2 配置示例 #####

**示例一：**假设有一个java项目，需要用mvn命令进行编译，此时可以使用maven的镜像作为agent，配置如下

```groovy
Jenkinsfile (Declarative Pipeline) //可以不要此行
pipelien {
    agent { docker 'maven:3-apline' } //版本需要和开发确认
    stages {
        stage( 'Examplc Bulid' ) {
            steps {
                sh ' mvn -B clean verify '
            }
        }
    }
}
```

**示例二：**本实例在流水线顶层将agent定义为node，那么此时stage部分就需要必须包含它自己的agent部分，在agent（ ‘Example Bulid’ ）部分使用maven:3-apline执行该阶段步骤，在stage（ ‘Example Test’ ）部分使用openjdk:8-jre 执行该阶段步骤，此时PiPeline如下：

	在每个stage去指定一个不同的镜像就可以实现不同的操作，==不需要再slave装一堆的依赖==

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent none
    stages {
        stage( 'Example Build' ) {
            agent { docker 'maven:3-apline' }
            steps {
                echo 'hello,Maven'
                sh 'mvn --version'
            }
        }
        stage('Example Test') {
            agent { docker 'openjdk:8-jre' }
            steps {
                echo 'hello,JDK'
                sh 'java --version'
            }
        }
    }
}
```

**示例三：**上述的示例可以用于基于Kubernetes的agent实现，比如定义具有三个容器的Pod，分别是

+ jnlp：负责和Jenkines Master通信 （固定写法，不需要做变更）
+ build：负责执行构建命令
+ kuberctl：负责执行kubernetes相关命令

在stage中通过containers字段，选择在某个容器执行命令：

**注意**：

1. tty=true : 保证容器不会退出（有一个前台运行的进程）
2. 镜像版本需要和开发确认

```groovy
pipeline {
    agent {
        kebernetes {
            cloud 'kubernetes-default'
            slaveConnectTimeout 1200
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - args: [\'$(JENKINS_SECRET)\',\'$(JENKINS_NAME)\']
      image: 'mirantis/jnlp-slave:latest'
      name: jnlp
      imagePullPolicy: IfNotPresent
    - command:
        - "cat"
      image: 'maven:3.5.3'
      name: "build"
      imagePullPolicy: IfNotPresent
      tty: true    
    - command:
        - "cat"
      image: 'kubectl'
      name: "build"
      imagePullPolicy: IfNotPresent
      tty: true
            """
        }
    }
    stages {
        stage('Building') {
            steps {
                container(name: 'build') {
                sh """
                  mvm clean install
                """
                }
            }
        }
        stage('Deploy') {
            steps {
                container(name: 'kubectl')
                sh """
                  kubectl get node
                """
            }
        }
    }
}
```

##### 1.2.1.3 Post #####

	post一般用于流水线结束后的进一步处理，比如错误通知等。Post可以针对流水线不同的结果做出不同的处理，就像开发程序的错误处理，比如Python语言的try catch。Post可以定义在Pipeline或stage中，目前支持一下条件：

+ always：无论Pipeline或stage的完成状态如何，都运行该post中定义的指令
+ changed：只有当前Pipeline或stage的完成状态与它之前的运行不同时，才允许在该post部分允许该步骤
+ fixed：当本次Pipeline或stage成功，且上一次构建是失败或者不稳定时，允许运行该post中定义的指令；
+ regression：当本次Pipeline或stage的完成状态的状态为失败、不稳定或终止，且上一次构建的状态为成功时，允许允许该post中定义的指令；
+ failure：只有当前Pipeline或stage的完成状态为失败（fiailure），才允许在ppost部分运行该步骤，通常在这时Web界面显示为红色；
+ success：当前状态为成功（success），执行post步骤，通常Web界面为蓝色或绿色；
+ unstable：当前状态为不稳定（unstable），执行post步骤，通常由于测试失败或代码违规等造成，在web界面中显示为黄色；
+ aborted：当前状态为终止（aborted），执行post步骤，通常由于流水线被手动终止触发，在这时Web界面中显示为灰色；
+ unsuccessful：当前状态不是success时，执行该Post步骤，
+ cleanup：无论pipeline或stage状态如何，都允许执行该post命令，和always区别在于，cleanup会在其他执行后执行

**最常用的：**

1. always：用于邮件通知
2. failure：一般配置在stage中，根据stage不同的执行结果，打印信息或生成配置、改通知内容

**示例：**一般情况下post部分放在流水线的底部，比如本实例，无论stage状态如何，都会输出一条 ” I will always say Hello again！“

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
               echo 'Hello Word'
            }
        }
    }
    post {
        always {
            echo 'I will always say Hello again'
        }
    }
}
```

  也可以将post写在stage：

+ 对stage生效，stage可能失败

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
               echo 'Hello Word'
            }
            post {
                failure {
                    echo "Pipeline Testing failure ..."
                }
            }
        }
    }
}
```

##### 1.2.1.4 Stages #####

	Stage包含一个或多个stage命令，同时可以在stage块中定义真正执行的指令。
	
	比如创建一个流水线，stage包含一个名为Example的stage，该stage执行 echo “hello Word

命令输出Hello Word字符串：

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Hello World ${env.BUIlD_ID}""
            }
        }
    }
}
```

**注意：**

1. 双引号：引用环境变量
2. 单引号：不能引用环境变量

##### 1.2.1.5 Steps #####

	Steps 部分在给定的stage指令中执行一个或多个步骤，比如在steps定义执行一条shell命令：

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
      stage('Example') {
        steps {
          echo 'Hello Word'
        }
      }
    }
}
```

或者是使用sh字段执行多条指令：

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stages('Example') {
            steps {
            sh """
            	echo 'Hello Word'
            	mv clean install
            """
        	}
        }  
    }
}
```

#### 1.2.2 Directives ####

指令学习

	Directives可用于执行stage时的条件判断或预处理一些数据，和secions一致，Directives不是一个关键字或指令，而是包含了environment、options、parameters、triggers、stage、tools、input、when等配置。

##### 1.2.2.1 Environment #####

环境变量的配置

	Environment主要用于在流水线中配置的一些环境变量，根据配置的位置决定环境变量的作用域：

+ 全局：可以定义在pipeline中作为全局变量
+ 局部：也可以配置在stage中作为该stage的环境变量。

	该指令支持一个特殊的放啊 credentials（），该方法可用于在Jenkins环境中通过标识符访问预定义的凭证。对于类型为 Secret Text的凭证，credentials（）可以将该Secret中的文本内容赋值给环境变量，对于类型为标准账号密码型的凭证，指定环境变量为username和password，并且会定义两个额外的环境变量，分别为 MYVARNAME_USR和MYVARNAME_PSW。

**注意**：Jenkins有一个凭证的关系系统；如连接hub等

	假如需要定义个变量名CC的全局变量和一个名为AN_ACCESS_KEY的全局变量，并且用credwntials读取一个 Secret文本，可以通过以下方式定义：

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    environment { // Pipeline中定义，属于全局变量
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment { //定义在stage中，属于局部变量
                AN_ACCESS_KEY = crednetials('my-prefined-secret-test')
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

##### 1.2.2.2 Options #####

	Jenkins 流水线支持很多的内置命令，比如 retry 可以对失败的步骤进行重复执行n次，可以根据不同的指令实现不同的效果。比如常见的指令如下：

+ buildDiscarder：保留多少个流水线的构建记录，比如 options { buildDiscarder(logRotator(numToKeepStr: ‘1’ ))}; 推荐在流水线配置，也可以在界面配置
+ disableConcurrentBuilds:禁止流水线并行执行，防止并行流水线同时访问共享资源导致流水线失败，比如：options {disableConcurrentBuilds() } . ==常用：因为在安装可能产生缓存文件，新的执行可能导致缓存文件冲突，一般都是禁止流水线并行==
+ disableResume：如果控制器重启（master重启），禁止流水线自动恢复。比如：options { disableResume() }。更具实际情况看
+ newContainerPerStage：agent为docker或者dockerfile时、每个阶段将在同一个节点的新容器中运行，而不是所有的阶段都在一个容器中运行，比如：options { newContainerPerStage() }。同一个容器速度更快，很少配置此参数
+ quietPeriod：流水线静默期，也就是流水线被触发后等待一会儿再执行。比如：options {quietPeriod(30) }。一般用于，这个流水线需要等待上一个流水线的结果、报告再执行
+ retry：流水线失败后重试的次数。比如：options { retry(3) }；
+ timeout：设置流水线的超时时间，超过流水线时间，job会自动终止，比如：options { timeout(time:1,unit:‘HOURS’) }
+ timrstamps：为控制台输出时间戳，比如：options { timestamps() }。输出日志加时间戳。

配置示例如下：只需要添加options字段即可

```groovy
pipeline {
    agent any
    options {
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

option除了写在pipeline顶层，还可以写在stage中，但是写在stage中的option仅支持retry、timeout、timestamps，或者是和stage相关声明式选项，比如 skipDefaultCheckout,处于stage级别的options写法如下：

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            options {
                timeout(time: 1, unit: 'HOURS')
            }
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

Jenkins界面操作显示：

  ![image-20221016160319056](C:\Users\dxx\Desktop\Kubernetes\images\image-20221016160319056.png)

`save`保存 点击`Build now`

 ![image-20221016160605822](C:\Users\dxx\Desktop\Kubernetes\images\image-20221016160605822.png)

**测试超时**

```groovy
timeout(time: 3, unit: 'SECONDS')
```

显示：叉 （Cancelling nested steps due to timeout）



##### 1.2.2.3 Parameters #####

	paramenters提供一个用户在触发流水线时应该提供的参数列表，这些用户指定参数的值可以通过params对象提供给流水线的step（步骤）。

目前支持的参数类型如下：

+ string：字符串，如 parameters { string(name:’DEPLOY_ENV’,defualtValue:’staging’,description:)},表示定义一个名为DEPLOY_ENV的字符型变量，默认值为staging。
+ text：文本型参数，一般用于定义多行文本内容的变量，如paramters { test(name:’DEPLOY_TEXT’,defualtValue:’One\nTwo\nThree\n’,description’‘’)} ,表示定义一个名为DEPLOY_TEXT的变量，默认值是’One\nTwo\nThree\n‘
+ booleanParam：布尔值，paramters { booleanParam(name:’DEPLOY_BUILD’,defualtValue:true,description’‘’)} 
+ choice：选择型参数，一般用于给定的几个可选的值，然后选择其中一个进行赋值，例如：paramters { choice(name:’CHOICES’,choices:[‘noe’、‘tow’、’three‘],description’‘’)} .表示可选值为one、two、three。
+ password：密码型变量，一般用于定义敏感型变量，再Jenkins控制台会输出*。例如 paramters { password(name:’PASSWORD’,defualtValue:’SECRET‘,description’A secret password‘)}，表示定义一个名为PASSWORD的变量，其默认值为SECRET。

Parameters用法如下：



```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PERSON',defaultValue: 'Mr Jenkins',description: "Who should I say hello to?")
        text(name: 'BIOGRAPHY',defaultValue: '',description: "Enter some information about the person")
        booleanParam(name: 'TOGGLE',defaultValue: true,description: "Toggle this value")
        choice(name: 'CHOICE',choices: ['One','Tow','Three'],description: "Pick something")
        password(name: 'PASSWORD',defaultValue: 'SECRET',description: "ENTER a password")
    }
    stages {
        stage('Example') {
            steps {
                echo "string:Hello ${params.PERSON}"
                echo "Biography: ${params.BIOGRAPHY}"
                echo "Toggle: ${params.TOGGLE}"
                echo "Choice: ${params.CHOICE}"
                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
```

注意：

+ 一般都是配置全局
+ 变量必须使用 ”双引号“
+ 如果再pipeline中定义了变量，有一个小bug（无法避免的，`SAVE`后不会让你选择参数列表，只有第一次构建后成功读取到参数配置才会显示使用参数化构建）

**构建一次后就读取到变量的配置**

 ![image-20221017120649826](C:\Users\dxx\Desktop\Kubernetes\images\image-20221017120649826.png)



##### 1.2.2.4 Triggers #####

在Pipeline中可以用triggers实现自动触发流水执行任务，可以通过 **Webhook:用的最多**、Cron、pollSCM和upstream等方式触发流水线；

假设某流水线构建时间比较长，或者某个流水线需要定期在某个时间段执行构建，可以使用cron配置触发器，比如周一到周五每个四个小时执行一次。

**注意**：手动点可能存在的问题

1. 流水线构建时间过长
2. 流水线过多

导致手动触发耽误自己时间太多，job有很多的扫描下载生成一个报告，经过几个小时才能结束，手动触发等结束肯定不行；

```groovy
pipeline {
    agent any
    triggers {
        cron('H * */4 * * 1-5')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

注意：H的意思不是hours的意思，而是Hash的缩写，主要是为了解决很多流水线在同一时间运行带来的系统负载压力。

 ![image-20221017123739045](C:\Users\dxx\Desktop\Kubernetes\images\image-20221017123739045.png)

使用cron字段可以定期执行流水线，如果代码更新想要重新触发流水线，可以使用pollSCM字段

```groovy
pipeline {
    agent any
    triggers {
        pollSCM('H */4 * * 1-5')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

Upstream 可以根据上游job的执行结果决定是否触发该流水线，比如当job1或job2执行成功时触发该流水线。

```groovy
pipeline {
    agent any
    triggers {
        upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS)
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

目前支持的状态：SYCCESS、UNSTABLE、FAILUARE、NOT_BUILD、ABORTED等；



##### 1.2.2.5 Input #####

	input 字段可以实现在流水线中进行交互式操作，比如选择要部署的环境、是否继续执行某个阶段等。

配置input支持以下选项：

+ message：必选，需要用过进行input的提示信息，比如“是否发布到生产环境”
+ id：可选、input的标识符，默认为stage的名称
+ ok：可选，确认信息按钮的显示信息，比如：“确定”、“允许”；
+ submitter：可选、允许提交input操作的用户或组的名称，如果空，如何登录的用户均可提交input
+ parameters：提供一个参数列表供input使用。

假设需要配置一个提示消息为“还继续么”、确认按钮为“继续”、提供一个PERSON的变量的参数，并且只能由登录用户为alice和bob提交的流水线

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            input {
                message "还继续么？"
                ok "继续"
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                }
            }
            steps {
                echo "Hello, ${PERSON}, nice to meet you."
            }
        }
    }
}
```

 ![image-20221017131302292](C:\Users\dxx\Desktop\Kubernetes\images\image-20221017131302292.png)







##### 1.2.2.6 When #####

用的频率非常非常高

	When指令允许流水线更具给定的条件决定是否应该执行该stage，when指令必须包含至少一个条件。如果when包含多个条件，所有的子条件必须返回True、stage才能执行。
	
	When也可以结合not、allOf、anyOf语法达到更灵活的条件匹配。

目前常用的内置条件如下：

+ branch：当正在构建的分支与给定的分支匹配时，执行这个stage，例如：when {branch ‘master’ }。注意，brabch只适用于多分支流水线
+ changelog：匹配提交的changeLog决定是否构建，例如：when { changelog ‘。*^\\[DEPENDENCY\\].+$’};
+ environment：当指定的环境变量和给定的变量匹配时，执行这个stage，例如：when{ environment name:’DEPLOY_TO’,calue:‘production’}
+ equals：当期望值和实际值相同时，执行stage，例如：when { equals expected:2,actual:currentBuild.bunmer}
+ expression：当指定的Groovy表达式评估为true，执行stage、例如：when { expression {return params.DEBUG_BUILD}}
+ tag：如果 TAG_NAME 的值和给定的条件匹配，执行这个stage，例如：when { tag “release-*”}
+ not：当嵌套条件出现错误时，执行这个stage，必须包含一个条件，例如：when { not { branch ’master‘ }}
+ allOf：当所有的嵌套条件正确时，执行这个stage、必须包含至少一个条件，例如：when { allOf {branch ’master‘；environment:’DEPLY_TO’,value:’production’}};
+ anyOf：当至少有一个嵌套条件为true时，执行这个satge，例如：when { anyOf {branch ’master‘； branch ’staging‘}}

示例一：当分支为production时，执行Example Deploy步骤：

```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

执行结果：跳过了stage('Example Deploy') 。 因为branch不是production

也可以同时配置多个条件

```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
                environment name: 'DEPLOY_TO', value: 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

两个条件都匹配才会执行satge

也可以使用anyOf进行匹配其中一个条件即可，比如分支为production，DEPLOY_TO为production或staging时执行Deploy：

```groovy
pipeline {
    agent any
        parameters {
        string(name: 'DEPLOY_TO',defaultValue: 'production',description: "Deploy")
        }
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo "Deploy to ${DEPLOY_TO}"
            }
        }
    }
}
```

执行结果：

 ![image-20221017141204150](C:\Users\dxx\Desktop\Kubernetes\images\image-20221017141204150.png)

也可以使用expression进行正则匹配，比如当BRANCH_NAME为production 或staging，并且DEPLOY_TO为production或staging时才会执行Example Deploy。

```groovy
pipeline {
    agent any
        parameters {
        string(name: 'DEPLOY_TO',defaultValue: 'production',description: "Deploy")
        }
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                expression { BRANCH_NAME ==~/(production|staging)}
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo "Deploy to ${DEPLOY_TO}"
            }
        }
    }
}
```

默认情况下，如果定义了某个stage的agent，在进入该stage的agent后，该stage的when条件才会被评估，但是可用通过一些选项更改次选项。比如在进入stage的agent前评估when，可以使用beforeAgent，当when为true时才进行该stage。

	目前支持的前置条件如下：

+ beforeAgent：如果beforeAgent为true，则会先评估when条件，在when条件为true时，才会进入strage；
+ deforeInput：如果beforeAgent为true，则会先评估when条件，在when条件为true时，才会进入input阶段；
+ beforeOptions：如果beforeAgent为true，则会先评估when条件，在when条件为true时，才会进入options阶段；

**注意：**beforeOptions优先级大于deforeInput大于beforeAgent

beforeAgent示例：

```groovy
pipeline {
    agent none
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            agent {
                label "some-label"
            }
            when {
                beforeAgent true
                branch 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

非常好用

用途：条件执行false直接跳过

#### 1.2.3 Parallel ####

并行执行

	在声明式流水线中可以使用Parallel字段，即可很方便的实现并发构建，比如对分支A、B、C进行处理：

假如提交了一次代码，在构建时候也要执行代码扫描、测试。先构建再测试不合理。

+ 一般将构建和代码扫描做一个并行处理，再构建的过程也去执行代码扫描（剩下很多的时间）
+ 实际生产环境，代码构建和扫描可能不是一条线。代码扫描集成到了构建（java的单元测试扫描）

```groovy
pipeline {
    agent any
    stages {
        stage('Non-Paraller Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            when {
                branch 'master'
            }
            failFast true //其他的stage失败，立即关闭
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
                stage('Branch C') {
                    agent {
                        label "for-branch-c"
                    }
                    stages {
                        stage('Nested 1') {
                          steps {
                                echo "In stage Nested 1 within Branch C"
                            }
                        }
                        stage('Nested 2') {
                            steps {
                                echo "In stage Nested 2 within Branch C"
                            }
                        }
                    }
                }
            }
        }
    }
}
```



### 1.3 Jenkinsfile ###

和pipeline时对等的，对pipeline'对版本的控制（回滚、提交记录、等）



pipeline保存的文件名称叫：Jenkinsfile（不是写死的）。

pipeline脚本放在了远端的SCM 如git仓库：那就时Jenkinsfile



流水线支持两种语法，即声明式和脚本式，者两种语法都支持构建持续交付流水线，并且都可以用来再Web UI 或Jenkinsfile中定义流水线，不过通常将Jenkinsfile放置于代码仓库中（当然也可以放在单独的代码仓库中进行管理）。

	创建一个Jenkinsfile 并将其放置于代码仓库中，有以下好处：

+ 方便对流水线上的代码进行复查/迭代：
+ 对管道进行审计追踪；
+ 流水线真正的源代码能够被项目的多个成员查看和编辑。



#### 1.3.1 环境变量 ####

##### 1.3.1.1 静态变量 #####

	Jenkins 有许多内置变量可以直接在Jenkinsfile中使用，可以通过 JENKINS_URL/pipclinc-syntax/globals#env获取完整的列表，目前比较常用的环境变量如下：

+ BUILD_ID：当前构建的ID，与Jenkins版本1.597+中的BUILD_NUMBER完全相同。
+ BUILD_NUMBER：当前构建的ID，和UBILD_ID 一致
+ BUILD_TAG：用来标识构建的版本号，格式为：jenkins-${JOB_NAME}-${BUILD_NUMBER},可以对产物进行命名，比如生产的jar包名字、镜像的TAG等；
+ BUILD_URL：本次构建的完整URL，比如：https://buildserver/jenkins/job/MyJobName/17/;
+ JOB_NAME：本次构建项目名称
+ NODE_NAME：当前构建节点名称
+ JENKINS_URL：Jenkins完整的URL，需要在system Configuration设置；
+ WORKSPACE执行构建的工作目录。

上述变量会保存在一个map中，可以使用env.BUILD 或 env.JENKINS_URL引用某个内置变量：

```groovy
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                sh "env"
            }
        }
    }
}
```

**输出：**running 20 on null

==BUILD_TAG=jenkins-input-28==：是唯一的

用途：构建镜像tag（每次tag都是唯一的，名称是唯一的，但是ID是递增）

对应脚本式流水线：

```groovy
node {
	echo "running ${env.BUILD_ID} on ${env.JENKINS_URL}"
}
```

除了上述的环境变量，也可以手动设置一些环境变量：

```groovy
pipeline {
    agent any
    environment {
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment {
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

上述配置了两个环境变量，一个CC值为clang，另一个是DEBUG_FLAGS值为-g。但是两者定义的位置不一样，CC位于顶层，适用于整个流水线，而DEBUG_FLAGS位于stage中，只适用于当前stage。

##### 1.3.1.2 动态变量 #####

	动态变量是根据某个指令的结果进行动态赋值，变量的值根据指令的执行结果而不同。如下所示：

```groovy
pipeline {
    agent any
    environment {
        CC = """${sh(
        	returnStdout: true //输出的结果赋值给CC
            script: 'echo "clang"'
        	)}"""
        EXIT_STATUS = """${sh(
        	returnStatus: true //输出的结果赋值给CC
            script: 'exit 1'
        	}"""
    }
    stages {
        stage('Example') {
            environment {
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

+ returnStdout：将命令的执行结果赋值给变量，比如上述的命令返回值是clang，此时CC的值为”clang “ . 注意后面多了一个空格，可以用.trim()将其删除
+ returnStatus：将命令的执行状态赋值给变量，比如上述命令的执行状态为1，此时EXIT_STATUS的值为1



#### 1.3.2 凭证 ####

jenkins的声明式流水线语法有一个credentials()函数，它支持secret text（加密文本）、username 和password（用户和密码）以及secret file（加密文件）等。接下来看一下一些常用的凭证处理方法。

##### 1.3.2.1 加密文本 #####

本实例演示将两个Secret文本凭证分配给单独的环境变量来访问Amazon Web服务，需要提前创建者两个文件的credentials（实践的章节会有演示），Jenkinsfile文件内容如下：

将k8s的证书和gitlab的ssk，以及hub的账号密码都是在jenkins管理的！

```groovy
pipeline {
    agent {
        
    }
    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }
    stages {
        stage('Example stage 1') {
            steps {
                //
            }
        }
        stage('Example stage 2') {
            steps {
                //
            }
        }
    }
}
```

说明：

	上述示例定义了两个全局变量AWS_ACCESS_KEY_ID和AWS_SECRET_ACCESS_KEY。这两个变量引用的是credentials的两个加密文件，并且这了ing个变量均可以在stages直接引用（通过 $AWS_SECRET_ACCESS_KEY 和 AWS_ACCESS_KEY_ID）

注意：如果在steps中使用echo $AWS_ACCESS_KEY_ID，则返回的是****，加密的内容不会被显示出来 

##### 1.3.2.2 用户名密码 #####

本实例用来演示credent账号密码的使用，比如使用一个公有账号访问Bitbucket、GitLab、Harbor等。假设已经设置配置完成了用户名密码形式的credentials，

	可以用以下方式设置凭证环境变量（BITBUCKET_COMMON_CREDS 名称可以自定义）：

```groovy
    environment {
        bitbucket_common_creds = credentials('jenkins-bitbucket-common-creds')
    }
```

上述的配置会自动生成三个环境变量：

+ BITBUCKET_COMMON_CREDS：包含一个一冒号分割的用户名和密码，格式为username:password
+ BITBUCKET_COMMON_CREDS_USR：仅包含用户名的附加变量
+ BITBUCKET_COMMON_CREDS_PSW：仅包含密码的附加变量

此时，调用用户名密码的Jenkinsfile如下：

```groovy
pipeline {
    agent any
    environment {
        BITBUCKET_COMMON_CREDS = credentials('STADY_USERINFO')
    }
    stages {
        stage('Example stage 1') {
            steps {
                echo "${BITBUCKET_COMMON_CREDS}"
            }
        }
        stage('Example stage 2') {
            steps {
                echo "${BITBUCKET_COMMON_CREDS_USR}"
                echo "${BITBUCKET_COMMON_CREDS_PSW}"
            }
        }
    }
}
```

注意：此时环境变量的凭证仅作用在stage 1，也可以配置在顶层对全局生效。



账号密码一般不会配置在全局

+ 一般在构建时使用

 ![image-20221018113749850](C:\Users\dxx\Desktop\Kubernetes\images\image-20221018113749850.png)



##### 1.3.2.3 加密文件 #####

	需要加密保存的文件，也可以使用credential，比如链接到Kuberneter集群的kubeconfig文件等。
	
	假如已经配置好了一个kubeconfig文件，此时可以在Pipeline中引用该文件：

```groovy
pipeline {
    agent {
        //
    }
    environment {
        MY_KUBECONFIG = credentials('my-kubeconfig')
    }  
    stages {
        stage('Example stage 1') {
            steps {
                sh("kubectl --kubeconfig $MY_KUBECONFIG get pods")
            }
        }
    }
}
```

#### 1.3.3 参数处理 ####

#### 1.3.4 使用多个代理 ####





### 1.4  DevOps平台建设 ###

首先学习在kubernetes中进行CICD的过程，一般步骤如下



下图为：非生产环境的发版样式

+ 有时候需要开发环境构建出来的镜像直接用于测试环境或者生成环境；==一次构建多次部署==

 ![image-20221018143723014](C:\Users\dxx\Desktop\Kubernetes\images\image-20221018143723014.png)

1. 在GitLab中创建对应的项目
2. 配置jenkins集成kubernetes集群，后期Jenkins和Slave将为在kubernetes中动态创建slave
3. jenkins创建对应的任务（JOB），集成该项目的Git地址和kubernetes集群
4. 开发者将代码提交到GitLab
5. 如有配置钩子，推送（push）代码会自动触发Jenkins构建，入门没有配置，需要手动构建
6. Jenkins控制Kubernetes（使用的时kubernetes插件）创建jenkins Slave（Pod形式）
7. Jenkins Slave根据流水线（pipeline）定义的步骤执行构建
8. 通过Dockerfile生成镜像
9. 将镜像推送到（push）私有Harbor（或者其他镜像仓库）
10. Jenkins再次控制kubernetes进行最新的镜像部署
11. 流水线结束删除Jenkins Slave







### 1.4.1 Jenkins安装 ###



##### 1.4.1.1 Jenkins 安装 ##### 

首先需要一个linux服务器，配置不低于2C4G和40G硬盘，首先安装docker：

```sh
yum install -y yum-utils device-,apper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce-19.03.* docekr-ce-cli-19.03.*
systemctl daemon-reload && systemctl enable --now docker
```

创建Jenkins的数据目录，防止容器重启后数据丢失：

```SH
mkdir /data/jenkins_data -p
chmod 777 /data/jenkins_data/
```

启动Jenkins，并配置管理员张哈皮密码 admin/admin123“

```SH
docker run --name myjenkins -p 8080:8080 -p 50000:50000 -e JENKINS_PASSWORD=admin123 -e JENKINS_USERNAME=admin -e JENKINS_HTTP_PORT_NUMBER=8080 -v /data/jenkins_data:/bitnami/jenkins bitnami/jenkins:2.361.2-debian-11-r4

```

其中8080端口为Jenkins Web界面端口、50000是jnlp使用的端口、后期jenkins需要使用50000端口和jenkins主节点通信。

查看jenkins日志：

```SH
docker logs -f myjenkins 
```

##### 1.4.1.2 插件安装 #####

如果时公司环境，在安装插件之前一定要备份

 ![image-20221018150614361](C:\Users\dxx\Desktop\Kubernetes\images\image-20221018150614361.png)

在安装之前首先配置国内的插件源，点击Advanced，将插件源更改为国内插件源

（https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json）

实验环境需要用到的插件

+ Git
+ Git parameter
+ Git Pipeline for Vlue Ocean 
+ GitLab 
+ Credentials
+ Credentials Binding 
+ Active Choices 
+ Image Tag Parameter 
+ Kubernetes Credentials 
+ Kubernetes CLI 
+ Kubernetes 
+ Pipeline: Declarative 
+ Pipeline 
+ List Git Branches Parameter 
+ Extended Choice Parameter 
+ Dynamic Parameter Plug-in 
+ Dynamic Extended Choice Parameter Plug-In 
+ Build With Parameters 
+ Dashboard for Blue Ocean 
+ Pipeline SCM API for Blue Ocean 
+ Blue Ocean 

 ![image-20221019123657910](C:\Users\dxx\Desktop\Kubernetes\images\image-20221019123657910.png)

### 1.4.2 GitLab安装 ###

GitLab 在企业内经常用于代码的版本控制，也是DevOps 平台中尤为重要的一个工具，接下来在另一台服务器（4C4G40G 以上）上安装 GitLab（如果同样有可用的 GitLab，也可无需安装）。 
首先在 GitLab 国内源下载 GitLab 的安装包：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/



将下载后的rpm 包，上传到服务器，之后通过 yum 直接安装即可： 

[gitlab-ce-15.4.2-ce.0.el7.x86_64.rpm](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-15.4.2-ce.0.el7.x86_64.rpm)

```SH
yum install -y gitlab-ce-15.4.2-ce.0.el7.x86_64.rpm
```

安装完成后需要修改几处配置

```SH
[root@k8s-node02 ~]# vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.31.205'
prometheus['enable'] = false
```

+ 将external_url更改为自己发布的地址，可以是服务器IP，也可以是一个被解析的域名

	大部分公司内可能已经有了 Prometheus 监控平台，所以GitLab 自带的Prometheus 可以无需安装，后期可以安装GitLab Exporter 即可进行监控。关闭 Prometheus 插件（可选）：



更改完成后需要重新加载配置文件： 

加载配置完成以后，可以看到如下信息： 

```SH
```

密码

```SH
[root@k8s-node02 ~]# cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: xVIjGd7B0WrVmGh4Sk4SsiVB3epnkXHFvNeAalF1Omo=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.

```

登录后，可以创建一个测试项目，进行一些简单的测试。首先创建一个组

 ![image-20221020120719762](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020120719762.png)

之后再去创建一个项目：（空白的）

> 先创建组、再创建项目

 ![image-20221020120904283](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020120904283.png)

 ![image-20221020121010565](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020121010565.png)

 ![image-20221020121105104](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020121105104.png)



比如使用master01作为git客户端，==可以生成一个key==

之后可以将 Jenkins 服务器的 key 导入到 GitLab，首先生成密钥（如有可以无需生成）： 

```SH
[root@k8s-master01 ~]# ssh-keygen -t rsa -C "baimiyishu13@163.com"
[root@k8s-master01 ~]# cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDEazMKIvKApCm1xVWyic3Ywp262sYeJuEuPrc4qM9oaC2pw4g5IjU2D3lYZuyteRu54HzB1ZSUC4KMtpW37XFSe/yPvKhwhdZ0D3DQaL2lJiG+Gxl0bOA9voTDs7ttlI+nbaJcf0sido3VqKmVEMQRlGwisPfphLlRKqULrrK/mmC88PAzdZqU6mb1eTL2JW4R1tDlvdDfNluL1CZ+V8YtV4qpXOPYIafcWTPlOrkqangPuLv67FWvT825GAY4xhL6LxT3/jLWX/EknvPNbh5qY72TDQ36JP3TQVHqoFURqyKwmFac3BM2yKKXKc1lo8/ZJKjLjHEYAJihgVSwVMyx baimiyishu13@163.com
```

将公钥的内容放在 GitLab 中即可： 

就可以拉取仓库了

**在 GitLab 找到 Profile： **

 ![image-20221020121605239](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020121605239.png)

选择左侧ssh Key

 ![image-20221020121713992](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020121713992.png)



  ![image-20221020121954902](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020121954902.png)

添加后就可以在 Jenkins 服务器拉取代码： 



```SH
[root@k8s-master01 test-project]# ls
README.md
[root@k8s-master01 test-project]# echo "Frist Commit For DevOps" > README.md
[root@k8s-master01 test-project]# echo "Frist Commit For DevOps" > first.md
[root@k8s-master01 test-project]# ls
first.md  README.md
[root@k8s-master01 test-project]# git add .
[root@k8s-master01 test-project]# git config --global user.email "baimiyishu13@163.com"
[root@k8s-master01 test-project]# git config --global user.name "baimiyishu13"
[root@k8s-master01 test-project]# git commit -am "first commint"
[main bbc06cb] first commint
 2 files changed, 2 insertions(+), 92 deletions(-)
 rewrite README.md (100%)
 create mode 100644 first.md
[root@k8s-master01 test-project]# git push origin main
Username for 'http://192.168.31.205': root
Password for 'http://root@192.168.31.205':
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 274 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To http://192.168.31.205/kubernetes/test-project.git
   07e33c4..bbc06cb  main -> main
You have new mail in /var/spool/mail/root
```



### 1.4.3 Harbor安装 ###

首先在 GitHub 下载最新的 Harbor 离线包，并上传至 Harbor 服务器，官方下载地址：https://github.com/goharbor/harbor/releases/ 
由于Harbor 是采用docker-compose 一键部署的，所以Harbor 服务器也需要安装Docker： 

首先需要安装docker：



安装完成后，将下载的Harbor离线包减压并载入Harbor镜像

```SH
[root@k8s-master03 ~]# tar -xf harbor-offline-installer-v2.6.1.tgz
You have new mail in /var/spool/mail/root
[root@k8s-master03 ~]# cd harbor
[root@k8s-master03 harbor]# docker load -i harbor.v2.6.1.tar.gz
```

之后安装 Compose： 

```SH
[root@k8s-master03 harbor]# curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 12.1M  100 12.1M    0     0  47686      0  0:04:27  0:04:27 --:--:-- 54809
You have new mail in /var/spool/mail/root
[root@k8s-master03 harbor]# chmod +x /usr/local/bin/docker-compose
You have new mail in /var/spool/mail/root
[root@k8s-master03 harbor]# docker-compose -v
docker-compose version 1.29.2, build 5becea4c
```

Harbor 默认提供了一个配置文件模板，需要更改如下字段： 

```SH
[root@k8s-master03 harbor]# ls
common.sh  harbor.v2.6.1.tar.gz  harbor.yml.tmpl  install.sh  LICENSE  prepare
[root@k8s-master03 harbor]# cp harbor.yml.tmpl harbor.yml
[root@k8s-master03 harbor]# vim harbor.yml
hostname: 192.168.31.203
#https
data_volume: /data/harbor
```

+ hostname：Harbor 的访问地址，可以是域名或者 IP，生产推荐使用域名，并且带有证书；
+ https：域名证书的配置，生产环境需要配置权威证书供 Harbor 使用，否则需要添加insecure-registry 配置，由于是学习环境，所以本示例未配置证书；
+ 账号密码按需修改即可，默认为 admin:Harbor12345。
  之后修改 Harbor 的数据目录：

创建Harbor 数据目录并进行预配置： 

```SH
[root@k8s-master03 harbor]# mkdir /data/harbor /var/log/harbor -p
[root@k8s-master03 harbor]# ./prepare
```

执行安装： 

```SH
[root@k8s-master03 harbor]# ./install.sh
```

成功启动后，即可通过配置的地址或域名访问： 

 ![image-20221020185707885](C:\Users\dxx\Desktop\Kubernetes\images\image-20221020185707885.png)

		如果配置不是https 协议，所有的Kubernetes 节点的Docker（如果是containerd 作为Runtime，可以参考下文配置 insecure-registry）都需要添加 insecure-registries 配置： 

 ```sh
 # vim /etc/docker/daemon.json
 {
   "exec-opts": ["native.cgroupdriver=systemd"],
   "insecure-registries": ["192.168.31.203"]
 }
 # 保存后重启docker
 [root@k8s-master01 ~]#  systemctl daemon-reload
 [root@k8s-master01 ~]# systemctl restart  docker
 ```

如果 Kubernetes 集群采用的是 Containerd 作为的 Runtime，配置 insecure-registry 只需要在Containerd 配置文件的 mirrors 下添加自己的镜像仓库地址即可： 

```SH
[root@k8s-master03 ~]# vim /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."192.168.31.203"]
        endpoint = ["http://192.168.31.203"]
[root@k8s-master03 ~]#  systemctl restart containerd
```

测试: 

+ tag
+ login 登录
+ push 上传

```SH

[root@k8s-node02 ~]# docker images
REPOSITORY                         TAG                     IMAGE ID       CREATED         SIZE
bitnami/jenkins                    2-debian-11             1c4c71478873   4 days ago      647MB
bitnami/jenkins                    2.361.2-debian-11-r4    1c4c71478873   4 days ago      647MB
192.168.31.203/kubernetes/alpine   3.16                    9c6f07244728   2 months ago    5.54MB
alpine                             3.16                    9c6f07244728   2 months ago    5.54MB
bitnami/jenkins                    2.303.1-debian-10-r29   01f2ad9a5d5a   13 months ago   575MB

[root@k8s-node02 ~]# docker login 192.168.31.203
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded

[root@k8s-node02 ~]# docker push 192.168.31.203/kubernetes/alpine:3.16
The push refers to repository [192.168.31.203/kubernetes/alpine]
994393dc58e7: Pushed
3.16: digest: sha256:1304f174557314a7ed9eddb4eab12fed12cb0cd9809e4c28f29af86979a3c870 size: 528

```

ctr上传

```SH
[root@k8s-node02 ~]# ctr -n k8s.io image pull 192.168.31.203/kubernetes/alpine:3.16 --plain-http --user admin:Harbor12345
```



### 1.4.4 Jenkins 凭证Credentials ###



Harbor 的账号密码、GitLab 的私钥、Kubernetes 的证书均使用 Jenkins 的Credentials 管理。 

#### 1.4.4.1 匹配Kubernetes证书 ####

	首先需要找到集群中的 KUBECONFIG，一般是 kubectl 节点的~/.kube/config 文件，或者是KUBECONFIG 环境变量所指向的文件。 

```SH
[root@k8s-master01 ~]# ls .kube/config
.kube/config
```

> 操作集群的配置文件，导出该文件

	接下来只需要把证书文件放置于 Jenkins 的 Credentials 中即可。首先点击Manage Jenkins，之后点击 Manage Credentials：

 ![image-20221021163256766](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021163256766.png)

	然后在点击 Jenkins，之后点击 Add Credentials： 

+ File：KUBECONFIG 文件或其它加密文件；
+ ID：该凭证的 ID；
+ Description：证书的描述。



#### 1.4.4.2 配置Harbor 账号密码 ####

	对于账号密码和 Key 类型的凭证，配置步骤是一致的，只是选择的凭证类型不一样。接下来通过 Jenkins 凭证管理 Harbor 的账号密码。 

在同样的位置点击 Add Credentials： 

选择类型为Username with password： 

 ![image-20221021164031763](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021164031763.png)

➢	Username：Harbor 或者其它平台的用户名；
➢	Password：Harbor 或者其它平台的密码；
➢	ID：该凭证的 ID；
➢	Description：证书的描述。



#### 1.4.4.3 配置 Gailab Key ####

点击 Add Credentials，类型选择为 SSH Username with private key： 

 ![image-20221021165310873](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021165310873.png)

需要注意：

Private Key：Jenkins 服务器的私钥，一般位于~/.ssh/id_rsa。

在gitlab上放的是公钥。公钥可以放在很多地方，

私钥：放在jenkins中（用于解密）

 ![image-20221021165840284](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021165840284.png)



### 1.4.5 配置Agent ###

通常情况下，Jenkins Slave 会通过Jenkins Master 节点的 50000 端口与之通信，所以需要开启 Agent 的 50000 端口。 

点击Manage Jenkins，然后点击Configure Global Security： 

 ![image-20221021171254766](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021171254766.png)

实际使用时，没有必要把整个Kubernetes 集群的节点都充当创建Jenkins Slave Pod 的节点，可以选择任意的一个或多个节点作为创建 Slave Pod 的节点。 

假设k8s-node01 作为 Slave 节点： 

```SH
[root@k8s-master01 ~]# kubectl label node k8s-node01 build=true
node/k8s-node01 labeled
```

如果集群并非使用 Docker 作为 Runtime，但是由于构建镜像时，需要使用Docker，所以该节点需要安装 Docker： 

如果镜像仓库未配置证书，需要配置 insecure-registry： 





#### 1.4.1 Jenkins配置Kubernetes多集群 ####

首先点击 Manage Jenkins，之后点击Configure Clouds： 

 ![image-20221021172448696](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021172448696.png)

 ![image-20221021173848589](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021173848589.png)

	最后点击 Save 即可，添加完 Kubernetes 后，在 Jenkinsfile 的 Agent 中，就可以选择该集群作为创建 Slave 的集群。 

如果想要添加多个集群，重复上述的步骤即可。首先添加 Kubernetes 凭证，然后添加Cloud即可。 

 ![image-20221021174032374](C:\Users\dxx\Desktop\Kubernetes\images\image-20221021174032374.png)





## 第二章 DevOps篇-DevOps实践 ##

### 1.5 自动化构建Java应用 ###

以下构建方法为一种通用的方法

	在实际生成环境中做的时候，基本框架都一样，可能会该一下构建的镜像、命令（一般情况）。

所以说：在做容器化时并不是非要找到案例才可以做

+ 对应k8s和jenkins而言：本身自己就是一个应用，不会了解你的应用是什么，什么框架、具体的其他东西
+ 部署方式不同，发版方式一样

 ![image-20221022120301056](C:\Users\dxx\Desktop\Kubernetes\images\image-20221022120301056.png)



选择git

#### 1.5.1 创建Java测试用例 ####

	示例项目可以从 https://gitee.com/dukuan/spring-boot-project.git 找到该项目（也可以使用公司的Java 项目也是一样的）。



接下来将该项目导入到自己的 GitLab 中。首先找到之前创建的 Kubernetes 组，然后点击