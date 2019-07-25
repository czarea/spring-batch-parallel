## 1.简介
Spring Batch是一个轻量级，可扩展且全面的批处理框架，可以大规模处理数据。Spring Batch构建在spring框架之上，为执行批处理应用程序提供直观，简单的配置。Spring Batch提供了可处理大量记录所必需的可重用功能，包括日志记录/跟踪，事务管理，作业处理统计，作业重启，跳过和资源管理等交叉问题。

详细介绍可参考：[http://czarea.com/2019/06/10/Spring%20Batch%E7%AE%80%E4%BB%8B/](http://czarea.com/2019/06/10/Spring%20Batch%E7%AE%80%E4%BB%8B/)

本文代码地址：[https://github.com/czarea/spring-batch-parallel.git](https://github.com/czarea/spring-batch-parallel.git)

## 2.parallel处理

### 2.1 gradle配置

```
plugins {
    id 'org.springframework.boot' version '2.1.6.RELEASE'
    id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.czarea'
version = '1.0'
sourceCompatibility = '1.8'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-batch'
    runtimeOnly 'org.hsqldb:hsqldb'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.batch:spring-batch-test'
}

```

这里需要说明下，为什么要加上hsqldb，因为spring batch会保存批处理信息到数据库，所以要加上数据库。


parallel配置
```
@Configuration
@EnableBatchProcessing
public class BatchConfiguration {

    Logger logger = LoggerFactory.getLogger(BatchConfiguration.class);

    @Autowired
    JobBuilderFactory jobBuilderFactory;

    @Autowired
    StepBuilderFactory stepBuilderFactory;


    @Bean
    public Job parallelStepsJob() {

        Flow masterFlow = new FlowBuilder<Flow>("masterFlow").start(taskletStep("step1")).build();


        Flow flowJob1 = new FlowBuilder<Flow>("flow1").start(taskletStep("step2")).build();
        Flow flowJob2 = new FlowBuilder<Flow>("flow2").start(taskletStep("step3")).build();
        Flow flowJob3 = new FlowBuilder<Flow>("flow3").start(taskletStep("step4")).build();

        Flow slaveFlow = new FlowBuilder<Flow>("splitflow")
                .split(new SimpleAsyncTaskExecutor()).add(flowJob1, flowJob2, flowJob3).build();

        return (jobBuilderFactory.get("parallelFlowJob")
                .incrementer(new RunIdIncrementer())
                .start(masterFlow)
                .next(slaveFlow)
                .build()).build();

    }


    private TaskletStep taskletStep(String step) {
        return stepBuilderFactory.get(step).tasklet((contribution, chunkContext) -> {
            IntStream.range(1, 100).forEach(token -> logger.info("Step:" + step + " token:" + token));
            return RepeatStatus.FINISHED;
        }).build();

    }
}    
```

- 加EnableBatchProcessing开启batch
- 新增一个简单的TaskletStep工厂，简单打印step步骤和1-100
- 创建一个masterFlow，一个slaveFlow，masterFlow只包含一个step，slaveFlow包含3个并行处理的step
- 创建Job，以masterFlow开始，再启动slaveFlow


运行打印日志，会先打印step1，做执行slaveFlow，其中3个step为并行处理

```
2019-07-25 15:18:58.260  INFO 17452 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step1]
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:1
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:2
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:3
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:4
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:5
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:6
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:7
2019-07-25 15:18:58.271  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:8
2019-07-25 15:18:58.272  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:9
2019-07-25 15:18:58.274  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:10
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:11
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:12
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:13
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:14
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:15
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:16
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:17
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:18
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:19
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:20
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:21
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:22
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:23
2019-07-25 15:18:58.275  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:24
2019-07-25 15:18:58.276  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:25
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:26
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:27
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:28
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:29
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:30
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:31
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:32
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:33
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:34
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:35
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:36
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:37
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:38
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:39
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:40
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:41
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:42
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:43
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:44
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:45
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:46
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:47
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:48
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:49
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:50
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:51
2019-07-25 15:18:58.278  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:52
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:53
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:54
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:55
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:56
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:57
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:58
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:59
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:60
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:61
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:62
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:63
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:64
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:65
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:66
2019-07-25 15:18:58.279  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:67
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:68
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:69
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:70
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:71
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:72
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:73
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:74
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:75
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:76
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:77
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:78
2019-07-25 15:18:58.283  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:79
2019-07-25 15:18:58.286  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:80
2019-07-25 15:18:58.286  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:81
2019-07-25 15:18:58.286  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:82
2019-07-25 15:18:58.286  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:83
2019-07-25 15:18:58.287  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:84
2019-07-25 15:18:58.287  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:85
2019-07-25 15:18:58.287  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:86
2019-07-25 15:18:58.287  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:87
2019-07-25 15:18:58.287  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:88
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:89
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:90
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:91
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:92
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:93
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:94
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:95
2019-07-25 15:18:58.293  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:96
2019-07-25 15:18:58.294  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:97
2019-07-25 15:18:58.294  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:98
2019-07-25 15:18:58.294  INFO 17452 --- [           main] c.c.s.p.config.BatchConfiguration        : Step:step1 token:99
2019-07-25 15:18:58.318  INFO 17452 --- [cTaskExecutor-2] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step3]
2019-07-25 15:18:58.320  INFO 17452 --- [cTaskExecutor-3] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step4]
2019-07-25 15:18:58.323  INFO 17452 --- [cTaskExecutor-1] o.s.batch.core.job.SimpleStepHandler     : Executing step: [step2]
2019-07-25 15:18:58.325  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:1
2019-07-25 15:18:58.325  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:2
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:3
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:4
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:5
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:6
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:7
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:8
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:9
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:10
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:1
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:11
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:12
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:2
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:3
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:4
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:5
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:6
2019-07-25 15:18:58.326  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:13
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:14
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:15
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:16
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:17
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:18
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:19
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:20
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:21
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:22
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:23
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:24
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:25
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:26
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:27
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:28
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:29
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:30
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:31
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:7
2019-07-25 15:18:58.327  INFO 17452 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : Step:step2 token:1
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:32
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : Step:step2 token:2
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:33
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : Step:step2 token:3
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:8
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:34
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:9
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : Step:step2 token:4
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:10
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : Step:step4 token:35
2019-07-25 15:18:58.329  INFO 17452 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : Step:step3 token:11
......
```

## 3.partition处理

Spring Batch中还有另一个并行处理用例，它是通过分区实现的。比如处理一个大的文件，因为IO问题，大文件的读取性能会比较差。在这种情况下，我们将单个文件拆分为多个文件，并且可以在同一个线程中处理每个文件。在我们的示例中，包含50条记录的单个文件person.txt已拆分为10个文件，每个文件包含5条记录。这可以通过使用split命令来实现

```
split -l 5 person.txt person
```

![partition处理流程](https://docs.spring.io/spring-batch/4.1.x/reference/html/images/partitioning-spi.png)

配置：

```
@Configuration
@EnableBatchProcessing
public class BatchConfiguration {

    Logger logger = LoggerFactory.getLogger(BatchConfiguration.class);

    @Autowired
    JobBuilderFactory jobBuilderFactory;

    @Autowired
    StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job partitioningJob() throws Exception {
        return jobBuilderFactory.get("parallelJob")
                .incrementer(new RunIdIncrementer())
                .flow(masterStep())
                .end()
                .build();
    }

    @Bean
    public Step masterStep() throws Exception {
        return stepBuilderFactory.get("masterStep")
                .partitioner(slaveStep())
                .partitioner("partition", partitioner())
                .gridSize(10)
                .taskExecutor(new SimpleAsyncTaskExecutor())
                .build();
    }

    @Bean
    public Partitioner partitioner() throws Exception {
        MultiResourcePartitioner partitioner = new MultiResourcePartitioner();
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        partitioner.setResources(resolver.getResources("classpath:files/persona*"));
        return partitioner;

    }

    @Bean
    public Step slaveStep() throws Exception {
        return stepBuilderFactory.get("slaveStep")
                .<Map<String, String>, Map<String, String>>chunk(1)
                .reader(reader(null))
                .writer(writer())
                .build();
    }

    @Bean
    @StepScope
    public FlatFileItemReader<Map<String, String>> reader(@Value("#{stepExecutionContext['fileName']}") String file) throws MalformedURLException {
        FlatFileItemReader<Map<String, String>> reader = new FlatFileItemReader<>();
        reader.setResource(new UrlResource(file));

        DefaultLineMapper<Map<String, String>> lineMapper = new DefaultLineMapper<>();
        DelimitedLineTokenizer tokenizer = new DelimitedLineTokenizer(":");
        tokenizer.setNames("key", "value");

        lineMapper.setFieldSetMapper((fieldSet) -> {
            Map<String, String> map = new LinkedHashMap<>();
            map.put(fieldSet.readString("key"), fieldSet.readString("value"));
            return map;
        });
        lineMapper.setLineTokenizer(tokenizer);
        reader.setLineMapper(lineMapper);

        return reader;
    }

    @Bean
    public ItemWriter<Map<String, String>> writer() {
        return (items) -> items.forEach(item -> {
            item.entrySet().forEach(entry -> {
                logger.info("key->[" + entry.getKey() + "] Value ->[" + entry.getValue() + "]");
            });
        });
    }

}

```
- 创建一个Job带有单个StepmasterStep的parallelJob。
- MasterStep有两个分区 - 一个提供数据作为分区，另一个处理分区数据。
- MultiResourcePartitioner用于提供分区数据。它查找当前目录中的persona文件，并将每个文件作为单独的分区返回
- gridSize用于指定要创建的分区数的估计值，实际可以超过。
- chunkSize提供为1以确保在读取每条记录后调用writer。理想情况下，最好指定一个更大的数字，因为每次传递都会处理大量的记录。
- 我们已经指定了执行程序AsyncTaskExecutor，开始创建的线程数等于分区数。


执行记录：

```
2019-07-25 15:58:54.494  INFO 11544 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [partition-masterStep]
2019-07-25 15:58:54.653  INFO 11544 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : key->[Khuwaja] Value ->[Tumarae]
2019-07-25 15:58:54.654  INFO 11544 --- [cTaskExecutor-6] c.c.s.p.config.BatchConfiguration        : key->[Kinroku] Value ->[Mazzy]
2019-07-25 15:58:54.655  INFO 11544 --- [cTaskExecutor-4] c.c.s.p.config.BatchConfiguration        : key->[Amantha] Value ->[Umiji]
2019-07-25 15:58:54.656  INFO 11544 --- [cTaskExecutor-5] c.c.s.p.config.BatchConfiguration        : key->[Kazuhiko] Value ->[Melso]
2019-07-25 15:58:54.653  INFO 11544 --- [cTaskExecutor-7] c.c.s.p.config.BatchConfiguration        : key->[Bijith] Value ->[Seikaly]
2019-07-25 15:58:54.657  INFO 11544 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : key->[Jonathon] Value ->[Lott]
2019-07-25 15:58:54.657  INFO 11544 --- [TaskExecutor-10] c.c.s.p.config.BatchConfiguration        : key->[Irán] Value ->[Boonyasomphop]
2019-07-25 15:58:54.658  INFO 11544 --- [cTaskExecutor-9] c.c.s.p.config.BatchConfiguration        : key->[Rong] Value ->[Sianturi]
2019-07-25 15:58:54.659  INFO 11544 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : key->[Dorice] Value ->[Izuit]
2019-07-25 15:58:54.660  INFO 11544 --- [cTaskExecutor-8] c.c.s.p.config.BatchConfiguration        : key->[Karunesh] Value ->[Dring]
2019-07-25 15:58:54.665  INFO 11544 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : key->[Fyodr] Value ->[Grondhal]
2019-07-25 15:58:54.667  INFO 11544 --- [cTaskExecutor-6] c.c.s.p.config.BatchConfiguration        : key->[Nucu] Value ->[Marfiak]
2019-07-25 15:58:54.668  INFO 11544 --- [cTaskExecutor-4] c.c.s.p.config.BatchConfiguration        : key->[Mikyung] Value ->[Salvidar]
2019-07-25 15:58:54.669  INFO 11544 --- [cTaskExecutor-5] c.c.s.p.config.BatchConfiguration        : key->[Jojie] Value ->[Sztil]
2019-07-25 15:58:54.675  INFO 11544 --- [cTaskExecutor-7] c.c.s.p.config.BatchConfiguration        : key->[Tamussie] Value ->[Maquignaz]
2019-07-25 15:58:54.676  INFO 11544 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : key->[Palmiero] Value ->[Albone]
2019-07-25 15:58:54.678  INFO 11544 --- [cTaskExecutor-9] c.c.s.p.config.BatchConfiguration        : key->[Milla] Value ->[Soner]
2019-07-25 15:58:54.681  INFO 11544 --- [TaskExecutor-10] c.c.s.p.config.BatchConfiguration        : key->[Chonthicha] Value ->[Obrzud]
2019-07-25 15:58:54.683  INFO 11544 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : key->[Junieth] Value ->[Alazan]
2019-07-25 15:58:54.685  INFO 11544 --- [cTaskExecutor-8] c.c.s.p.config.BatchConfiguration        : key->[Helianta] Value ->[Brletich]
2019-07-25 15:58:54.686  INFO 11544 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : key->[Oliver] Value ->[Krylova]
2019-07-25 15:58:54.691  INFO 11544 --- [cTaskExecutor-4] c.c.s.p.config.BatchConfiguration        : key->[Dakotah] Value ->[Schmitgall]
2019-07-25 15:58:54.693  INFO 11544 --- [cTaskExecutor-6] c.c.s.p.config.BatchConfiguration        : key->[Jaikrishan] Value ->[Klintberg]
2019-07-25 15:58:54.696  INFO 11544 --- [cTaskExecutor-5] c.c.s.p.config.BatchConfiguration        : key->[Anusheh] Value ->[Karlas]
2019-07-25 15:58:54.697  INFO 11544 --- [cTaskExecutor-7] c.c.s.p.config.BatchConfiguration        : key->[Messaoud] Value ->[Robuck]
2019-07-25 15:58:54.698  INFO 11544 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : key->[Yamilet] Value ->[Rohácová]
2019-07-25 15:58:54.700  INFO 11544 --- [cTaskExecutor-9] c.c.s.p.config.BatchConfiguration        : key->[Ortwin] Value ->[Conev]
2019-07-25 15:58:54.701  INFO 11544 --- [TaskExecutor-10] c.c.s.p.config.BatchConfiguration        : key->[Demaine] Value ->[Bety]
2019-07-25 15:58:54.702  INFO 11544 --- [cTaskExecutor-8] c.c.s.p.config.BatchConfiguration        : key->[Keshini] Value ->[Geerts]
2019-07-25 15:58:54.703  INFO 11544 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : key->[Eulogio] Value ->[Battilana]
2019-07-25 15:58:54.706  INFO 11544 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : key->[Tiam] Value ->[Jared]
2019-07-25 15:58:54.707  INFO 11544 --- [cTaskExecutor-4] c.c.s.p.config.BatchConfiguration        : key->[Kristy] Value ->[Maog]
2019-07-25 15:58:54.708  INFO 11544 --- [cTaskExecutor-6] c.c.s.p.config.BatchConfiguration        : key->[Pushkar] Value ->[Miccolis]
2019-07-25 15:58:54.718  INFO 11544 --- [cTaskExecutor-5] c.c.s.p.config.BatchConfiguration        : key->[Turgun] Value ->[Giovenco]
2019-07-25 15:58:54.719  INFO 11544 --- [cTaskExecutor-7] c.c.s.p.config.BatchConfiguration        : key->[Marni] Value ->[Ibáñez]
2019-07-25 15:58:54.720  INFO 11544 --- [cTaskExecutor-9] c.c.s.p.config.BatchConfiguration        : key->[Crescen] Value ->[Astley]
2019-07-25 15:58:54.721  INFO 11544 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : key->[Adiayl] Value ->[Viseu]
2019-07-25 15:58:54.723  INFO 11544 --- [TaskExecutor-10] c.c.s.p.config.BatchConfiguration        : key->[Shandra] Value ->[Ladr]
2019-07-25 15:58:54.723  INFO 11544 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : key->[Irja] Value ->[Juhandi]
2019-07-25 15:58:54.724  INFO 11544 --- [cTaskExecutor-8] c.c.s.p.config.BatchConfiguration        : key->[Wakayama] Value ->[Descendre]
2019-07-25 15:58:54.726  INFO 11544 --- [cTaskExecutor-1] c.c.s.p.config.BatchConfiguration        : key->[Rajaram] Value ->[Acimovic]
2019-07-25 15:58:54.728  INFO 11544 --- [cTaskExecutor-6] c.c.s.p.config.BatchConfiguration        : key->[Adnia] Value ->[Nettan]
2019-07-25 15:58:54.730  INFO 11544 --- [cTaskExecutor-5] c.c.s.p.config.BatchConfiguration        : key->[Dima] Value ->[Benbow]
2019-07-25 15:58:54.731  INFO 11544 --- [cTaskExecutor-4] c.c.s.p.config.BatchConfiguration        : key->[Aundra] Value ->[Nijeboer]
2019-07-25 15:58:54.732  INFO 11544 --- [cTaskExecutor-7] c.c.s.p.config.BatchConfiguration        : key->[Manuela] Value ->[Echevarria]
2019-07-25 15:58:54.734  INFO 11544 --- [cTaskExecutor-9] c.c.s.p.config.BatchConfiguration        : key->[Behzad] Value ->[Pahlevi]
2019-07-25 15:58:54.734  INFO 11544 --- [cTaskExecutor-2] c.c.s.p.config.BatchConfiguration        : key->[Babah] Value ->[Hoseman]
2019-07-25 15:58:54.736  INFO 11544 --- [TaskExecutor-10] c.c.s.p.config.BatchConfiguration        : key->[Andrielle] Value ->[Sermona]
2019-07-25 15:58:54.737  INFO 11544 --- [cTaskExecutor-3] c.c.s.p.config.BatchConfiguration        : key->[Lincia] Value ->[Weatherbee]
2019-07-25 15:58:54.738  INFO 11544 --- [cTaskExecutor-8] c.c.s.p.config.BatchConfiguration        : key->[Mécheal] Value ->[Madaule]
2019-07-25 15:58:54.765  INFO 11544 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=parallelJob]] completed with the following parameters: [{run.id=1}] and the following status: [COMPLETED]
```



## 4. 总结
Spring Batch提供了两种并行处理，一种是并行多线程处理数据，一种是把一个原本是一个任务的分拆为多个进行处理。
