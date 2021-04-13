## Jenkins指令

### pipeline简介

pipeline 简单来说，就是一套运行在Jenkins 上的工作流框架，将原来独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。

### triggers

该triggers指令定义了Pipeline 应重新触发的自动化方式。目前有三个可用的触发器是cron和pollSCM 和 upstream。

```bash
pipeline {
    agent any
    stages {
        stage('拉取代码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ef0f4ce0-c68a-4607-9e25-21c38c50442f', url: '此处填写你的git地址']]])
                echo '拉取代码成功'
            }
        }
        stage('构建项目') {
            steps {
                sh 'mvn clean package'
                echo '构建项目成功'
            }
        }
        stage('项目部署') {
            steps {
                deploy adapters: [tomcat8(credentialsId: '40de4b9b-71f8-4514-8694-2307daa9963f', path: '', url: 'http://192.168.31.101:8080')], contextPath: null, war: 'target/*.war'
                echo '项目部署成功'
            }
        }
    }
    post {
         //always 无论成功或者失败都执行下面的脚本
         always {
            emailext(
               //subject邮件标题    PROJECT_NAME:项目名    BUILD_NUMBER:构建次数 BUILD_STATUS 构建状态 
               subject: '构建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!',
               //body内容,可以直接在body写,也可以写成成下面的形式进行读取
               body: '${FILE,path="email.html"}',
               //to 收件人,多个用逗号隔开
               to: '收件人邮箱'
            )
         }
    }
}		

```

指定构建后操作

解释：

- Always {}: 总是执行脚本片段
- success{}; 成功后执行
- failure{}; 失败后执行
- aborted{}:取消后执行

- currentBuild 是一个全局变量
  - description 构建描述

```bash

#option 指定运行选项
options {
	timestamps {}
	
}
#构建任务后操作
post {
	always {
		script {
			println('always')
		}
	}
	success {
		script {
			println('always')
		}
	}
	failure {
		script {
			println('always')
		}
	}
	aborted {
		script {
			println('always')
		}
	}
}
```

指定node节点/workspace

指定指定运行选项(可以省略)

```java
agent {
	node {
		label 'master' // 指定运行节点的标签或者名称
		customWorkspace "${workspace}" //指定运行工作目录(可选)
	}
}
option {
  timestamps() //日志会有时间
  skipDefaultCheckout() //删除隐士checkout scm语句
   timeout(time:1,unit: 'HOURS') //流水线超时设置1h
}
```

指定stages 阶段(一个或多个)

解释：在这里我添加了三个阶段

```bash
stages {
	stage("GetCode") { //阶段名称
		steps {
			timeout(time:5,unit:"MINUTES") { //步骤超时时间
				script {//填写运行代码
					println('获取代码')
				}
			}
		}
	}
	stage('Build') {
		steps {
			timeout(time:20, unit:"MINUTES"){
				script {
					println('应用打包')
				}
			}
		}
	}
}
```

[参考文档](https://zeyangli.github.io/)

### 触发下游项目执行

如果我们开发的项目想触发另一个顶级jenkins job,可以使用如下声明式语法。构建步骤默认等待触发的下游项目，但是有一个wait可以设置为false,准许在你的项目pipeline管道中触发并忘记。

```bash
pipeline {
    agent any
    stages {
        stage ('Compile Stage') {
            steps {
                build job: 'test_path', wait: false
            }
        }
    }
}

```

![image-20210228082418753](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210228082418753.png)



























