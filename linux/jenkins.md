## jenkins

Pipeline 是一套运行于jenkins上的工作流框架，将原本独立运行于单个或多个节点的任务连接起来，实现单个任务难以完成的复杂流程编排与可视化。

#### 块(Blocks{})

- 由大括号括起来的语句：如 Pipeline{}, Section{}, parameters{},script{}
- 章节(Sections) 通常包括一个或者多个指令或步骤， 如 agent post stages steps
- 指令(Directives) environment options parameters triggers stage tools when
- 步骤(steps) 执行脚本式pipeline, 如script{}

#### agent

该agent部分指定整个Pipeline或特定阶段将在Jenkins环境中执行的位置，具体取决于该agent部分的放置位置。该部分必须在pipeline块的顶层定义，但阶段级使用时可选的。

简单来说，agent 部分主要作用就是告诉Jenkins,选择那台节点机器去执行pipeline代码。这个指令时必须要有的，也就在你顶层pipeline{...}的下一层，必须要一个agent{...},agent 这个指令对应的有多个可选的参数，这里注意一点，在具体某一个stage{...}里面也可以使用agent 指令。

- any 在任何可用的代理上执行Pipeline 或stage.
- None 当在pipeline块的顶层应用时，将不会为整个pipeline运行分配全局代理，并且每个stage部分将需要包含其自己的agent 部分。

```bash
pipeline {
	agent none
	stages {
		stage('Build'){
			agent {
				label '具体的节点名称'
			}
		}
	
	}
}
```

- Label 使用提供的标签在Jenkins环境中可用的代理机器上执行Pipeline 或 stage内执行。

```bash
pipeline {
	agent {
		label '具体一个节点label名称'
	}
}
```

#### writeFile

如何写。readFile 和writeFile都是简单的文本文件读写。当你拿到一个文件，不知道选择什么方式来读写，那么这个readFile 和writeFile 就是最原始的方法。但是，如果是json类型文件，那么我们可以用前面介绍插件的readJson来读取，读取到是一个json的对象，就可以直接调用json的方法。



























