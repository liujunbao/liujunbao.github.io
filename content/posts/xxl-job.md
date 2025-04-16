---
title: Xxl Job
date: 2025-01-10T18:30:15+08:00
draft: false
toc: false
images: 
tags:
  - untagged
---

## xxl-job

第一步去看官方文档：[分布式任务调度平台XXL-JOB](https://www.xuxueli.com/xxl-job)，照着文档看一遍。
我来简单重复一下文档的过程
### 1. 部署调度中心

去[Releases · xuxueli/xxl-job](https://github.com/xuxueli/xxl-job/releases)下载你想要的版本的源码。调度中台是xxl-job-admin模块，数据库位置是：doc/db/tables_xxl_job.sql，然后修改一下application.properties中的数据库的部分，部署一下就完成了。

### 2. 执行器

执行器和我们自己的任务看xxl-job-executor-samples下的xxl-job-executor-sample-springboot，然后集成到我们的项目里，这里注意：**最好是完全按照sample中的配置文件来，不要漏写不要多写。还有就是执行器的版本最好和调度中心的版本保持一致**。
大概有，我这里列一下2.1.1版本的：
```java
package com.xxl.job.executor.core.config;  
  
import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
/**  
 * xxl-job config * * @author xuxueli 2017-04-28 */@Configuration  
public class XxlJobConfig {  
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);  
  
    @Value("${xxl.job.admin.addresses}")  
    private String adminAddresses;  
  
    @Value("${xxl.job.executor.appname}")  
    private String appName;  
  
    @Value("${xxl.job.executor.ip}")  
    private String ip;  
  
    @Value("${xxl.job.executor.port}")  
    private int port;  
  
    @Value("${xxl.job.accessToken}")  
    private String accessToken;  
  
    @Value("${xxl.job.executor.logpath}")  
    private String logPath;  
  
    @Value("${xxl.job.executor.logretentiondays}")  
    private int logRetentionDays;  
  
  
    @Bean  
    public XxlJobSpringExecutor xxlJobExecutor() {  
        logger.info(">>>>>>>>>>> xxl-job config init.");  
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();  
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);  
        xxlJobSpringExecutor.setAppName(appName);  
        xxlJobSpringExecutor.setIp(ip);  
        xxlJobSpringExecutor.setPort(port);  
        xxlJobSpringExecutor.setAccessToken(accessToken);  
        xxlJobSpringExecutor.setLogPath(logPath);  
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);  
  
        return xxlJobSpringExecutor;  
    }  
  
    /**  
     * 针对多网卡、容器内部署等情况，可借助 "spring-cloud-commons" 提供的 "InetUtils" 组件灵活定制注册IP；  
     *  
     *      1、引入依赖：  
     *          <dependency>  
     *             <groupId>org.springframework.cloud</groupId>  
     *             <artifactId>spring-cloud-commons</artifactId>  
     *             <version>${version}</version>  
     *         </dependency>  
     *     *      2、配置文件，或者容器启动变量  
     *          spring.cloud.inetutils.preferred-networks: 'xxx.xxx.xxx.'  
     *     *      3、获取IP  
     *          String ip_ = inetUtils.findFirstNonLoopbackHostInfo().getIpAddress();     */  
  
}
```

```yml
# web port  
server.port=8081  
  
# log config  
logging.config=classpath:logback.xml  
  
  
### xxl-job admin address list, such as "http://address" or "http://address01,http://address02"  
xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin  
  
### xxl-job executor address  
xxl.job.executor.appname=xxl-job-executor-sample  
xxl.job.executor.ip=  
xxl.job.executor.port=9999  
  
### xxl-job, access token  
xxl.job.accessToken=  
  
### xxl-job log path  
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler  
### xxl-job log retention days  
xxl.job.executor.logretentiondays=30
```

```java
package com.xxl.job.executor.service.jobhandler;  
  
import com.xxl.job.core.biz.model.ReturnT;  
import com.xxl.job.core.handler.IJobHandler;  
import com.xxl.job.core.handler.annotation.JobHandler;  
import com.xxl.job.core.log.XxlJobLogger;  
import org.springframework.stereotype.Component;  
  
import java.util.concurrent.TimeUnit;  
  
  
/**  
 * 任务Handler示例（Bean模式）  
 *  
 * 开发步骤：  
 * 1、继承"IJobHandler"：“com.xxl.job.core.handler.IJobHandler”；  
 * 2、注册到Spring容器：添加“@Component”注解，被Spring容器扫描为Bean实例；  
 * 3、注册到执行器工厂：添加“@JobHandler(value="自定义jobhandler名称")”注解，注解value值对应的是调度中心新建任务的JobHandler属性的值。  
 * 4、执行日志：需要通过 "XxlJobLogger.log" 打印执行日志；  
 *  
 * @author xuxueli 2015-12-19 19:43:36 */@JobHandler(value="demoJobHandler")  
@Component  
public class DemoJobHandler extends IJobHandler {  
  
    @Override  
    public ReturnT<String> execute(String param) throws Exception {  
       XxlJobLogger.log("XXL-JOB, Hello World.");  
  
       for (int i = 0; i < 5; i++) {  
          XxlJobLogger.log("beat at:" + i);  
          TimeUnit.SECONDS.sleep(2);  
       }  
       return SUCCESS;  
    }  
  
}
```