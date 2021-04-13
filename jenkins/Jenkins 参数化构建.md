## Jenkins 参数化构建

参数化构建是用于和用户提供交互的一种方式，通过选择传入不同的参数，来执行不同的任务，如：传入不同的项目名称，让构建项目根据传入参数进行下一步构建。

### pipeline模式Jenkinsfile中使用

首先在Jenkinsfile中定义参数：

```bash
pipeline {
    agent { label 'xxx' }
    parameters {
        string(name: 'BASEOS_FILENAME', defaultValue: '', description: 'baseos output filename')
        string(name: 'TRGIER_JOBNAME', defaultValue: '', description: 'trigger job name')
    }
```

如上代码中，定义了字符串参数BASEOS_FILENAME 和 字符串参数TRGIER_JOBNAME。

在pipeline中定义了参数之后，在jenkins 构建job时，体现在如下两种形式：

1. 在开始构建的时候，需要填写构建需要的参数

![image-20210127172820091](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210127172820091.png)

2. 点击“参数“菜单，可以查看本次构建使用的参数

![image-20210127173019936](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210127173019936.png)

### 多pipeline job之间传递参数

如果想在多个job之间传递参数，需要在build job中设置参数：

```bash
stage ('trigger out arm  build') {
       steps {
           build job: 'tvos-arm-build/release', wait: false,parameters:[
               [$class: "StringParameterValue", name: "BASEOS_FILENAME", value: "${env.fileOutName}"],
               [$class: "StringParameterValue", name: "TRGIER_JOBNAME", value: "fullos-build/raspberrypi3-release"],
                    ]
            }
        }
```

如上代码，设置了在构建完成本项目，会触发下游项目“tvos-arm-build/release”构建，同时传递参数BASEOS_FILENAME 和 TRGIER_JOBNAME 。









