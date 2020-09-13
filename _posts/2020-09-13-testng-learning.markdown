---
layout:     post
title:      "《TestNG》学习笔记"
subtitle:   ""
date:       2020-09-13 22:00:00
author:     "Wudashan"
header-img: "img/post-bg-testng-learning.jpg"
catalog: true
tags:
    - 开源框架
    - 源码分析
---

# 执行时序图

```
@startuml
User -> TestNG : main(argv)
activate TestNG
		TestNG -> Configuration : new Configuration()
		TestNG -> JCommander : parse(argv)
    TestNG -> TestNG : validateCommandLineParameters()
    TestNG -> TestNG : configure(CommandLineArgs)
		group run()
				group initializeEverything() 
					TestNG -> TestNG : initializeSuitesAndJarFile()
					TestNG -> TestNG : initializeConfiguration()
					TestNG -> TestNG : initializeDefaultListeners()
					TestNG -> TestNG : initializeCommandLineSuites()
					TestNG -> TestNG : initializeCommandLineSuitesParams()
					TestNG -> TestNG : initializeCommandLineSuitesGroups()
					note right: 主要是在初始化List<XmlSuite>
				end
				TestNG -> TestNG : sanityCheck()
				TestNG -> IExecutionListener : onExecutionStart()
				TestNG -> IAlterSuiteListener : alter()
				TestNG -> TestNG : runSuites()
				note right: 框架执行细节
				TestNG -> TestNG : generateReports()
				TestNG -> IExecutionListener : onExecutionFinish()
		end
User <-- TestNG : void
deactivate TestNG
@enduml
```