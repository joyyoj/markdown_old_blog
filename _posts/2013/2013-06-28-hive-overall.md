---
layout: post
title: HIVE基本框架
categories:
- 分布式查询
tags:
- HIVE
---

### HIVE基本流程

1.主要流程包括，语法解析（抽象语法树，AST,采用antlr），语义分析(sematic Analyzer生成查询块)，逻辑计划生成（OP tree），逻辑计划优化,物理计划生成（Task tree），以及物理计划执行组成。
2.使用antlr定义sql语法，（详细见hive.g），由antlr工具将hive.g编译为两个java文件：HiveLexer.java    HiveParser.java，可以将输入的sql解析为ast树
3.org.apache.hadoop.hive.ql.Driver对ast树进行初步的解析（combile），调用相应的语法分析器进行分析处理（包括DDl，Explain，Load等，其中最重要的是：SemanticAnalyzer）
4.SemanticAnalyzer的主要分析过程：调用analyzeInternal函数
     1）doPhase1过程：主要是将sql语句中涉及到的各种信息存储起来，存到QB中去，供后续调用
     2）getMetaData：这个过程主要是获取元数据信息，主要是sql中涉及到的表到元数据的关联
     3）genPlan：这是最重要的过程之一，主要是生成算子树（operator tree）
     4）optimize：优化，对算子树进行一些优化操作，例如列剪枝等
     5）genMapRedTasks：这个步骤是最关键的步骤，将算子树通过一定的规则生成若干相互以来的MR任务
5.Driver编译完成以后，开始进入执行阶段（execute），执行过程按照任务树从roottask开始依次执行，直至结束。

说明清楚各个模块的衔接点，接口交互即可。

	CliDriver.main
  	  processLine
	  	processCmd
    	  Driver.run(cmd)
			compile
			  BaseSemanticAnalyzer.analyze
			    SemanticAnalyzer.analyzeInternal
				  doPhase
				  getMetaData
				  genPlan
				  Optimizer.optimize
				  genMapRedTasks
            execute
			  launchTask
				TaskRunner.run
				  Task.executeTask
				    ExecDriver.execute
				      submitJob
		 getResults

CliDriver.main > processLine > processCmd >> Driver.run(cmd) > compile >> BaseSemanticAnalyzer >> xxxSemanticAnalyzer（常规select走SemanticAnalyzer） > analyze(sem.analyze) >> SemanticAnalyzer的analyzeInternal方法 >> new Optimizer.optimize（进行列剪裁等优化后生成Task） > genMapRedTasks >> 返回到Driver.run(cmd) >>ret = execute() >> launchTask >> TaskRunner.run > Task.executeTask > ExecDriver.execute > 执行MR（submitJob） >> getResults.

