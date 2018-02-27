# 分布式任务调度LTS用法示例
## 准备环境
Zookeeper集群：192.168.146.83:2181,192.168.146.84:2181,192.168.146.85:2181<br/>
JobTracker：192.168.85.72 3501 3502<br/>
LTS-Admin：http://192.168.146.83:8001/ltsAdmin/node-manager.htm (用户名密码：admin admin)<br/>
## 引入lts相关jar包
```xml:pom.xml
<!--LTS-->
    <dependency>
        <groupId>com.github.ltsopensource</groupId>
        <artifactId>lts</artifactId>
        <version>1.7.0</version>
    </dependency>
    <dependency>
        <groupId>com.github.sgroschupf</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.1</version>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.0.20.Final</version>
    </dependency>
    <dependency>
        <groupId>org.javassist</groupId>
        <artifactId>javassist</artifactId>
        <version>3.20.0-GA</version>
    </dependency>
    <dependency>
        <groupId>org.mapdb</groupId>
        <artifactId>mapdb</artifactId>
        <version>2.0-beta10</version>
    </dependency>
```
## Application启动类中添加@EnableJobClient和@EnableTaskTracker注解
## 配置文件中分别加入jobclient和tasktracker相关的配置
```properties:application.properties
#lts config
lts.jobclient.cluster-name=wecloud_cluster
lts.jobclient.registry-address=zookeeper://192.168.146.83:2181,zookeeper://192.168.146.84:2181,zookeeper://192.168.146.85:2181
lts.jobclient.node-group=wecloudDemoSchedule
lts.jobclient.use-retry-client=true
lts.jobclient.configs.job.fail.store=mapdb
testTaskSchdule.enabled=true
schedule.testTask.cron=0 */1 * * * ?

lts.tasktracker.cluster-name=wecloud_cluster
lts.tasktracker.registry-address=zookeeper://192.168.146.83:2181,zookeeper://192.168.146.84:2181,zookeeper://192.168.146.85:2181
lts.tasktracker.node-group=wecloudDemoSchedule
lts.tasktracker.configs.job.fail.store=mapdb
lts.tasktracker.dispatch-runner.enable=true
task_tracker_node_group=wecloudDemoSchedule
```
## JobClient相关类
JobStartup（任务触发类）和SchedulerService（具体任务触发配置项）。
## TaskTracker相关类
MasterNodeChangeListener（节点变化监听）和JobScheduler（具体任务定义）以及具体任务Service实现类。