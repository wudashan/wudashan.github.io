---
layout:     post
title:      "Spring Batch批处理框架介绍"
subtitle:   "一款轻量的、全面的批处理框架，用于开发强大的批处理应用程序。"
date:       2017-07-23 22:30:00
author:     "Wudashan"
header-img: "img/post-bg-spring-batch-analysis.jpg"
catalog: true
tags:
    - Spring
    - Batch
    - 批处理
    - 开源框架
---

> 本篇博客基于Spring Batch的3.0.8版本。

# 前言

在大型的企业应用中，或多或少都会存在大量的任务需要处理，如邮件批量通知所有将要过期的会员等等。而在批量处理任务的过程中，又需要注意很多细节，如任务异常、性能瓶颈等等。那么，使用一款优秀的框架总比我们自己重复地造轮子要好得多一些。

我所在的物联网云平台部门就有这么一个需求，需要实现批量下发命令给百万设备。为了防止枯燥乏味，下面就让我们先通过Spring Batch框架简单地实现一下这个功能，再来详细地介绍这款框架！

---

# 小试牛刀

## 引入依赖

首先我们需要引入对`Spring Batch`的依赖，在`pom.xml`文件加入下面的代码：

```
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-core</artifactId>
    <version>3.0.8.RELEASE</version>
</dependency>
```

## 装载Bean

其次，我们需要在resources目录下，创建`applicationContext.xml`文件，用于自动注入我们需要的类：

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">


    <!-- 事务管理器 -->
    <bean id="transactionManager" class="org.springframework.batch.support.transaction.ResourcelessTransactionManager"/>

    <!-- 任务仓库 -->
    <bean id="jobRepository" class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean">
        <property name="transactionManager" ref="transactionManager"/>
    </bean>

    <!-- 任务加载器 -->
    <bean id="jobLauncher" class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
        <property name="jobRepository" ref="jobRepository"/>
    </bean>

</beans>
```

有了上面声明的transactionManager、jobRepository、jobLauncher，我们就可以执行批量任务啦！不过，我们还需要创建一个任务。在Spring Batch框架中，一个任务Job由一个或者多个步骤Step，而步骤又由读操作Reader、处理操作Processor、写操作Writer组成，下面我们分别创建它们。

## 创建Reader

既然是读操作，那么肯定要有能读的数据源，方便起见，我们直接在resources目录下创建一个`batch-data.csv`文件，内容如下：

```
1,PENDING
2,PENDING
3,PENDING
4,PENDING
5,PENDING
6,PENDING
7,PENDING
8,PENDING
9,PENDING
10,PENDING
```

非常简单，其中第一列代表着命令的id，第二列代表着命令的当前状态。也就是说，现在有10条缓存的命令，需要下发给设备。

读操作需要实现`ItemReader<T>`接口，框架提供了一个现成的实现类`FlatFileItemReader`。使用该类需要设置`Resource`和`LineMapper`。Resource代表着数据源，即我们的batch-data.csv文件；LineMapper则表示如何将文件的每行数据转成对应的DTO对象。

### 创建DTO对象

由于我们的数据源是命令数据，所以我们需要创建一个`DeviceCommand.java`文件，代码如下：

```
public class DeviceCommand {

    private String id;

    private String status;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }
}
```

### 自定义LineMapper

我们需要自己实现一个LineMapper实现类，用于将batch-data.csv文件的每行数据，转成程序方便处理的DeviceCommand对象。

```
public class HelloLineMapper implements LineMapper<DeviceCommand> {

    @Override
    public DeviceCommand mapLine(String line, int lineNumber) throws Exception {

        // 逗号分割每一行数据
        String[] args = line.split(",");
        
        // 创建DeviceCommand对象
        DeviceCommand deviceCommand = new DeviceCommand();
        
        // 设置id值到对象中
        deviceCommand.setId(args[0]);
        
        // 设置status值到对象中
        deviceCommand.setStatus(args[1]);
        
        // 返回对象
        return deviceCommand;

    }

}
```

## 创建Processor

读完数据后，我们就需要处理数据了。既然我们前面从文件里读取了待下发的命令，那么在这里下发命令给设备是最好的时机。处理操作需要实现`ItemProcessor<I, O>`接口，我们自己实现一个`HelloItemProcessor.java`即可，代码如下：

```
public class HelloItemProcessor implements ItemProcessor<DeviceCommand, DeviceCommand> {

    @Override
    public DeviceCommand process(DeviceCommand deviceCommand) throws Exception {

        // 模拟下发命令给设备
        System.out.println("send command to device, id=" + deviceCommand.getId());

        // 更新命令状态
        deviceCommand.setStatus("SENT");

        // 返回命令对象
        return deviceCommand;
        
    }
    
}
```

## 创建Writer

处理完数据后，我们需要更新命令状态到文件里，用于记录我们已经下发。与读文件类似，我们需要实现`ItemWriter<T>`接口，框架也提供了一个现成的实现类`FlatFileItemWriter`。使用该类需要设置`Resource`和`LineAggregator`。Resource代表着数据源，即我们的batch-data.csv文件；LineAggregator则表示如何将DTO对象转成字符串保存到文件的每行。

### 自定义LineAggregator

我们需要自己实现一个LineAggregator实现类，用于将DeviceCommand对象转成字符串，保存到batch-data.csv文件。

```
public class HelloLineAggregator implements LineAggregator<DeviceCommand> {

    @Override
    public String aggregate(DeviceCommand deviceCommand) {

        StringBuffer sb = new StringBuffer();
        sb.append(deviceCommand.getId());
        sb.append(",");
        sb.append(deviceCommand.getStatus());
        return sb.toString();

    }

}
```

## 主程序

那么，完事具备，只欠东风！接下面我们在主程序`Main.java`里实现我们的批量命令下发功能！代码如下：

```
public class Main {


    public static void main(String[] args) throws Exception {

        // 加载上下文
        String[] configLocations = {"applicationContext.xml"};
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(configLocations);

        // 获取任务启动器
        JobLauncher jobLauncher = applicationContext.getBean(JobLauncher.class);
        JobRepository jobRepository = applicationContext.getBean(JobRepository.class);
        PlatformTransactionManager transactionManager = applicationContext.getBean(PlatformTransactionManager.class);

        // 创建reader
        FlatFileItemReader<DeviceCommand> flatFileItemReader = new FlatFileItemReader<>();
        flatFileItemReader.setResource(new FileSystemResource("src/main/resources/batch-data.csv"));
        flatFileItemReader.setLineMapper(new HelloLineMapper());

        // 创建processor
        HelloItemProcessor helloItemProcessor = new HelloItemProcessor();

        // 创建writer
        FlatFileItemWriter<DeviceCommand> flatFileItemWriter = new FlatFileItemWriter<>();
        flatFileItemWriter.setResource(new FileSystemResource("src/main/resources/batch-data.csv"));
        flatFileItemWriter.setLineAggregator(new HelloLineAggregator());

        // 创建Step
        StepBuilderFactory stepBuilderFactory = new StepBuilderFactory(jobRepository, transactionManager);
        Step step = stepBuilderFactory.get("step")
                                      .<DeviceCommand, DeviceCommand>chunk(1)
                                      .reader(flatFileItemReader)       // 读操作
                                      .processor(helloItemProcessor)    // 处理操作
                                      .writer(flatFileItemWriter)       // 写操作
                                      .build();

        // 创建Job
        JobBuilderFactory jobBuilderFactory = new JobBuilderFactory(jobRepository);
        Job job = jobBuilderFactory.get("job")
                                   .start(step)
                                   .build();

        // 启动任务
        jobLauncher.run(job, new JobParameters());

    }

}
```

执行main方法之后，屏幕将会输出下面信息：

```
send command to device, id=1
send command to device, id=2
send command to device, id=3
send command to device, id=4
send command to device, id=5
send command to device, id=6
send command to device, id=7
send command to device, id=8
send command to device, id=9
send command to device, id=10
```

再查看`batch-data.csv`文件，将会发现命令状态全部更新为SENT：

```
1,SENT
2,SENT
3,SENT
4,SENT
5,SENT
6,SENT
7,SENT
8,SENT
9,SENT
10,SENT
```

至此，我们的批量命令下发全部成功！可以发现，使用Spring Batch框架来实现批处理非常的轻量，当然这只是它所有功能里的冰山一角。

---

# 正式介绍

Spring Batch在官网是这样一句话介绍自己的：A lightweight, comprehensive batch framework designed to enable the development of robust batch applications vital for the daily operations of enterprise systems.（一款轻量的、全面的批处理框架，用于开发强大的日常运营的企业级批处理应用程序。）

框架主要有以下功能：

 - Transaction management（事务管理）
 - Chunk based processing（基于块的处理）
 - Declarative I/O（声明式的输入输出）
 - Start/Stop/Restart（启动/停止/再启动）
 - Retry/Skip（重试/跳过）
 
如果你的批处理程序需要使用上面的功能，那就大胆地使用它吧！

## 框架全貌

![](http://docs.spring.io/spring-batch/trunk/reference/html/images/spring-batch-reference-model.png.pagespeed.ce.TrtTC751hI.png)

框架一共有4个主要角色：`JobLauncher`是任务启动器，通过它来启动任务，可以看做是程序的入口。`Job`代表着一个具体的任务。`Step`代表着一个具体的步骤，一个Job可以包含多个Step（想象把大象放进冰箱这个任务需要多少个步骤你就明白了）。`JobLauncher`是存储数据的地方，可以看做是一个数据库的接口，在任务执行的时候需要通过它来记录任务状态等等信息。