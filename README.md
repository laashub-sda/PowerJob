# 简介
### 产品介绍
OhMyScheduler是基于Akka架构的一款分布式调度平台与分布式计算框架（对标 Alibaba SchedulerX2.0），其主要功能特性如下：

* 使用简单：提供前端Web界面，允许开发者可视化地完成调度任务的管理（增、删、改、查）、任务运行状态监控和运行日志查看等功能。
* 定时策略完善：支持CRON表达式、固定频率、固定延迟和API四种定时调度策略。
* 执行模式丰富：支持单机、广播、Map、MapReduce四种执行模式，其中Map/MapReduce处理器能使开发者寥寥数行代码便获得集群分布式计算的能力。
* 执行器支持广泛：支持Spring Bean、普通Java对象、Shell、Python等处理器，应用范围广（比如广播执行+Shell脚本用来清理日志）
* 运维便捷：支持在线日志功能，执行器产生的日志可以在前端控制台页面实时显示，降低debug成本，极大地提高开发效率。
* 依赖精简：最小仅依赖关系型数据库（MySQL/Oracle/MS SQLServer...），扩展依赖为MongoDB（用于存储庞大的在线日志）。
* 高可用&高性能：调度服务器经过精心设计，一改其他调度框架基于数据库锁的策略，实现了无锁化调度。部署多个调度服务器可以同时实现高可用和性能的提升（支持无限的水平扩展）。
* 故障转移与恢复：任务执行失败后，可根据配置的重试策略完成重试，只要执行器集群有足够的计算节点，任务就能顺利完成。

### 适用场景
* 有定时执行需求的业务场景：如每天凌晨全量同步数据、生成业务报表等。
* 有需要全部机器一同执行的业务场景：如日志清理。
* 有需要分布式处理的业务场景：比如需要更新一大批数据，单机执行耗时非常长，可以使用Map/MapReduce处理器完成任务的分发，调动整个集群加速计算。

### 同类产品对比
||QuartZ|xxl-job|SchedulerX 2.0|OhMyScheduler|
|----|----|----|----|----|
|定时类型|CRON|CRON|CRON、固定频率、固定延迟、OpenAPI|CRON、固定频率、固定延迟、OpenAPI|
|任务类型|内置Java|内置Java、GLUE Java、Shell、Python等脚本|内置Java、外置Java（FatJar）、Shell、Python等脚本|内置Java、外置Java（容器）、Shell、Python等脚本|
|分布式任务|无|静态分片|MapReduce动态分片|MapReduce动态分片|
|在线任务治理|不支持|支持|支持|支持|
|日志白屏化|不支持|支持|不支持|支持|
|调度方式及性能|基于数据库锁，有性能瓶颈|基于数据库锁，有性能瓶颈|不详|无锁化设计，性能强劲无上限|
|报警监控|无|邮件|短信|邮件，提供接口允许开发者自定义开发|
|系统依赖|MySQL|MySQL|人民币（公测期间免费，哎，帮打个广告吧）|任意Spring Data Jpa支持的关系型数据库（MySQL、Oracle...）|
|DAG工作流|不支持|不支持|支持|暂不支持，有明确开发计划|



# 接入流程（文档不要太详细，简单强大兼得说的就是在下～）
1. [项目部署及初始化](./others/doc/SystemInitGuide.md)
2. [处理器开发](./others/doc/ProcessorDevGuide.md)
3. [任务配置与在线查看](./others/doc/ConsoleGuide.md)
4. [(强大灵活的扩展——OpenAPI)](./others/doc/OpenApiGuide.md)

# 开发日志
### 已完成
* 定时调度功能：支持CRON表达式、固定时间间隔、固定频率和API四种方式。
* 任务执行功能：支持单机、广播、Map和MapReduce四种执行方式。
* 执行处理器：支持SpringBean、普通Java对象、Shell脚本、Python脚本的执行。
* 在线日志：分布式日志方案。
* 高可用与水平扩展：调度服务器可以部署任意数量的节点，不存在调度的性能瓶颈。
* 不怎么美观但可以用的前端界面。
* OpenAPI：通过OpenAPI可以允许开发者在自己的应用上对OhMyScheduler进行二次开发，比如开发自己的定时调度策略，通过API的调度模式触发任务执行。

### 下阶段目标
* 日志的限流 & 本地分表提升在线日志最大吞吐量
* 工作流（任务编排）：当前版本勉强可以用MapReduce代替，不过工作流挺酷的，等框架稳定后进行开发。
* [应用级别资源管理和任务优先级](https://yq.aliyun.com/articles/753141?spm=a2c4e.11153959.teamhomeleft.1.696d60c9vt9lLx)：没有机器资源时，进入排队队列。不过我觉得SchedulerX的方案不太行，SchedulerX无抢占，一旦低优先级任务开始运行，那么只能等他执行完成才能开始高优先级任务，这明显不合理。可是考虑抢占的话又要多考虑很多东西...先放在TODO列表吧。
* 保护性判断（这个太繁琐了且意义不大，毕竟面向开发者，大家不会乱填参数对不对～）

# 参考
>Alibaba SchedulerX 2.0

* [Akka 框架](https://yq.aliyun.com/articles/709946?spm=a2c4e.11153959.teamhomeleft.67.6a0560c9bZEnZq)：不得不说，akka-remote简化了相当大一部分的网络通讯代码。
* [执行器架构设计](https://yq.aliyun.com/articles/704121?spm=a2c4e.11153959.teamhomeleft.97.371960c9qhB1mB)：这篇文章反而不太认同，感觉我个人的设计更符合Yarn的“架构”。
* [MapReduce模型](https://yq.aliyun.com/articles/706820?spm=a2c4e.11153959.teamhomeleft.83.6a0560c9bZEnZq)：想法很Cool，大数据处理框架都是处理器向数据移动，但对于传统Java应用来说，数据向处理器移动也未尝不可，这样还能使框架的实现变得简单很多。
* [广播执行](https://yq.aliyun.com/articles/716203?spm=a2c4e.11153959.teamhomeleft.40.371960c9qhB1mB)：运行清理日志脚本什么的，也太实用了8～

# 后记
* 产品永久开源（Apache License, Version 2.0），免费使用，且目前开发者@KFCFans有充足的时间进行项目维护和提供无偿技术支持（All In 了解一下），欢迎各位试用！
* 欢迎共同参与本项目的贡献，PR和Issue都大大滴欢迎（求求了）～
* 觉得还不错的话，可以点个Star支持一下哦～ =￣ω￣=
* 联系肥宅兄@KFCFans -> `tengjiqi@gmail.com`