# Nepxion Discovery
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg?label=license)](https://github.com/Nepxion/Discovery/blob/master/LICENSE)
[![Maven Central](https://img.shields.io/maven-central/v/com.nepxion/discovery.svg?label=maven%20central)](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.nepxion%22%20AND%20discovery)
[![Javadocs](http://www.javadoc.io/badge/com.nepxion/discovery.svg)](http://www.javadoc.io/doc/com.nepxion/discovery)
[![Build Status](https://travis-ci.org/Nepxion/Discovery.svg?branch=master)](https://travis-ci.org/Nepxion/Discovery)

Nepxion Discovery是一款对Spring Cloud的服务注册发现的增强中间件，其功能包括多版本灰度发布，黑/白名单的IP地址过滤，限制注册等，支持Eureka、Consul和Zookeeper。现有的Spring Cloud微服务可以方便引入该插件，使用者不需要对业务代码做任何修改，只需要做三个非常容易的事情
- 引入Plugin Starter依赖到pom.xml
- 必须为微服务定义一个版本号（version），在application.properties或者yaml的metadata里
- 建议为微服务自定义一个便于为微服务归类的属性值，例如组名（group）或者应用名（application），在application.properties或者yaml的metadata里，便于灰度界面分析
- 如果采用了远程配置中心集成的话，那么只需要在那里修改规则（XML），触发推送；如果未集成，可以通过客户端工具（例如Postman）推送修改的规则（XML）
- 具体教程和示例查看最下面的“示例演示”，图形化演示视频，请访问[http://www.iqiyi.com/w_19s07thtsh.html](http://www.iqiyi.com/w_19s07thtsh.html)，视频清晰度改成720P，然后最大化播放

## 痛点
现有Spring Cloud的痛点
- 如果你是运维负责人，是否会经常发现，你掌管的测试环境中的服务注册中心，被一些不负责的开发人员把他本地开发环境注册上来，造成测试人员测试失败。你希望可以把本地开发环境注册给屏蔽掉，不让注册
- 如果你是运维负责人，生产环境的某个微服务集群下的某个实例，暂时出了问题，但又不希望它下线。你希望可以把该实例给屏蔽掉，暂时不让它被调用
- 如果你是业务负责人，鉴于业务服务的快速迭代性，微服务集群下的实例发布不同的版本。你希望根据版本管理策略进行路由，提供给下游微服务区别调用，达到多版本灰度访问控制
- 如果你是测试负责人，希望对微服务做A/B测试，那么通过动态改变版本达到该目的

## 简介
- 实现对基于Spring Cloud的微服务和Zuul网关的支持
  - 具有极大灵活性 - 支持在任何环节做过滤控制和版本灰度发布
  - 具有极小限制性 - 只要开启了服务注册发现，程序入口加了@EnableDiscoveryClient
- 实现服务注册层面的控制
  - 基于黑/白名单的IP地址（或者HostName，下同）过滤机制禁止对相应的微服务进行注册
  - 基于最大注册数的限制微服务注册。一旦微服务集群下注册的实例数目已经达到上限（可配置），将禁止后续的微服务进行注册
- 实现服务发现层面的控制
  - 基于黑/白名单的IP地址（或者HostName，下同）过滤机制禁止对相应的微服务被发现
  - 基于版本配对，通过对消费端和提供端可访问版本对应关系的配置，在服务发现和负载均衡层面，进行多版本访问控制
- 实现灰度发布
  - 通过规则改变，实现灰度发布
  - 通过版本切换，实现灰度发布
- 实现通过XML进行上述规则的定义
- 实现通过事件总线机制（EventBus）的功能，实现发布/订阅功能
  - 对接远程配置中心，异步接受远程配置中心主动推送规则信息，动态改变微服务的规则
  - 结合Spring Boot Actuator，异步接受Rest主动推送规则信息，动态改变微服务的规则
  - 结合Spring Boot Actuator，动态改变微服务的版本
  - 在服务注册层面的控制中，一旦禁止注册的条件触发，主动推送异步事件，以便使用者订阅
- 实现通过Listener机制进行扩展
  - 使用者可以自定义更多的规则过滤条件
  - 使用者可以对服务注册发现核心事件进行监听
- 实现支持Spring Boot Actuator和Swagger集成
- 实现独立控制台，支持对规则和版本集中管理，未来考虑界面实现
- 实现支持未来扩展更多的服务注册中心
- 实现图形化的灰度发布功能

## 场景
- 黑/白名单的IP地址注册的过滤
  - 开发环境的本地微服务（例如IP地址为172.16.0.8）不希望被注册到测试环境的服务注册发现中心，那么可以在配置中心维护一个黑/白名单的IP地址过滤（支持全局和局部的过滤）的规则
  - 我们可以通过提供一份黑/白名单达到该效果
- 最大注册数的限制的过滤
  - 当某个微服务注册数目已经达到上限（例如10个），那么后面起来的微服务，将再也不能注册上去
- 黑/白名单的IP地址发现的过滤
  - 开发环境的本地微服务（例如IP地址为172.16.0.8）已经注册到测试环境的服务注册发现中心，那么可以在配置中心维护一个黑/白名单的IP地址过滤（支持全局和局部的过滤）的规则，该本地微服务不会被其他测试环境的微服务所调用
  - 我们可以通过推送一份黑/白名单达到该效果
- 多版本灰度访问控制
  - A服务调用B服务，而B服务有两个实例（B1、B2），虽然三者相同的服务名，但功能上有差异，需求是在某个时刻，A服务只能调用B1，禁止调用B2。在此场景下，我们在application.properties里为B1维护一个版本为1.0，为B2维护一个版本为1.1
  - 我们可以通过推送A服务调用某个版本的B服务对应关系的配置，达到某种意义上的灰度控制，切换版本的时候，我们只需要再次推送即可
- 动态改变微服务版本
  - 在A/B测试中，通过动态改变版本，不重启微服务，达到访问版本的路径改变

## 架构
简单描述一下，本系统的核心模块“基于版本控制的灰度发布”，从网关（Zuul）开始的灰度发布操作过程
- 灰度发布前
  - 假设当前生产环境，调用路径为网关(V1.0)->服务A(V1.0)->服务B(V1.0)
  - 运维将发布新的生产环境，部署新服务集群，服务A(V1.1)，服务B(V1.1)
  - 由于网关(1.0)并未指向服务A(V1.1)，服务B(V1.1)，所以它们是不能被调用的  
- 灰度发布中
  - 新增用作灰度发布的网关(V1.1)，指向服务A(V1.1)->服务B(V1.1)
  - 灰度网关(V1.1)发布到服务注册发现中心，但禁止被服务发现，网关外的调用进来无法负载均衡到网关(V1.1)上
  - 在灰度网关(V1.1)->服务A(V1.1)->服务B(V1.1)这条调用路径做灰度测试
  - 灰度测试成功后，把网关(V1.0)指向服务A(V1.1)->服务B(V1.1)
- 灰度发布后  
  - 下线服务A(V1.0)，服务B(V1.0)，灰度成功
  - 灰度网关(V1.1)可以不用下线，留作下次版本上线再次灰度发布

架构图

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Architecture.jpg)

## 兼容
版本兼容情况
- Spring Cloud F版，请采用4.x.x版本，具体代码参考master分支
- Spring Cloud C版、D版和E版，请采用3.x.x版本，具体代码参考Edgware分支
- 4.x.x版本由于Swagger和Spring Boot 2.x.x版本的Actuator用法有冲突，故暂时不支持Endpoint功能，其他功能和3.x.x版本一致

中间件兼容情况
- Consul
  - Spring Cloud F版，最好采用Consul的1.2.1服务器版本（或者更高），从[https://releases.hashicorp.com/consul/1.2.1/](https://releases.hashicorp.com/consul/1.2.1/)获取
  - Spring Cloud C版、D版和E版，必须采用Consul的0.9.3服务器版本（或者更低），从[https://releases.hashicorp.com/consul/0.9.3/](https://releases.hashicorp.com/consul/0.9.3/)获取
- Zookeeper
  - Spring Cloud F版，必须采用Zookeeper的3.5.x服务器版本（或者更高）
  - Spring Cloud C版、D版和E版，最好采用Zookeeper的3.5.0以下服务器版本（或者更低）
- Eureka
  - 跟Spring Cloud版本保持一致

## 依赖
微服务选择相应的插件引入
```xml
<dependency>
    <groupId>com.nepxion</groupId>
    <artifactId>discovery-plugin-starter-eureka</artifactId>
    <version>${discovery.plugin.version}</version>
</dependency>

<dependency>
    <groupId>com.nepxion</groupId>
    <artifactId>discovery-plugin-starter-consul</artifactId>
    <version>${discovery.plugin.version}</version>
</dependency>

<dependency>
    <groupId>com.nepxion</groupId>
    <artifactId>discovery-plugin-starter-zookeeper</artifactId>
    <version>${discovery.plugin.version}</version>
</dependency>
```

独立控制台引入
```xml
<dependency>
    <groupId>com.nepxion</groupId>
    <artifactId>discovery-console-starter</artifactId>
    <version>${discovery.plugin.version}</version>
</dependency>
```

## 工程

| 工程名 | 描述 |
| --- | --- | 
| discovery-plugin-framework | 核心框架 |
| discovery-plugin-framework-consul | 核心框架的Consul扩展 |
| discovery-plugin-framework-eureka | 核心框架的Eureka扩展 |
| discovery-plugin-framework-zookeeper | 核心框架的Zookeeper扩展 |
| discovery-plugin-config-center | 配置中心实现 |
| discovery-plugin-admin-center | 管理中心实现 |
| discovery-console | 独立控制台，提供给UI |
| discovery-plugin-starter-consul | Consul Starter |
| discovery-plugin-starter-eureka | Eureka Starter |
| discovery-plugin-starter-zookeeper | Zookeeper Starter |
| discovery-console-starter | Console Starter |

## 规则和策略
### 规则示例
请不要被吓到，我只是把注释写的很详细而已，里面配置没几行
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <!-- 如果不想开启相关功能，只需要把相关节点删除即可，例如不想要黑名单功能，把blacklist节点删除 -->
    <register>
        <!-- 服务注册的黑/白名单注册过滤，只在服务启动的时候生效。白名单表示只允许指定IP地址前缀注册，黑名单表示不允许指定IP地址前缀注册。每个服务只能同时开启要么白名单，要么黑名单 -->
        <!-- filter-type，可选值blacklist/whitelist，表示白名单或者黑名单 -->
        <!-- service-name，表示服务名 -->
        <!-- filter-value，表示黑/白名单的IP地址列表。IP地址一般用前缀来表示，如果多个用“;”分隔，不允许出现空格 -->
        <!-- 表示下面所有服务，不允许10.10和11.11为前缀的IP地址注册（全局过滤） -->
        <blacklist filter-value="10.10;11.11">
            <!-- 表示下面服务，不允许172.16和10.10和11.11为前缀的IP地址注册 -->
            <service service-name="discovery-springcloud-example-a" filter-value="172.16"/>
        </blacklist>

        <!-- <whitelist filter-value="">
            <service service-name="" filter-value=""/>
        </whitelist>  -->

        <!-- 服务注册的数目限制注册过滤，只在服务启动的时候生效。当某个服务的实例注册达到指定数目时候，更多的实例将无法注册 -->
        <!-- service-name，表示服务名 -->
        <!-- filter-value，表示最大实例注册数 -->
        <!-- 表示下面所有服务，最大实例注册数为10000（全局配置） -->
        <count filter-value="10000">
            <!-- 表示下面服务，最大实例注册数为5000，全局配置值10000将不起作用，以局部配置值为准 -->
            <service service-name="discovery-springcloud-example-a" filter-value="5000"/>
        </count>
    </register>

    <discovery>
        <!-- 服务发现的黑/白名单发现过滤，使用方式跟“服务注册的黑/白名单过滤”一致 -->
        <!-- 表示下面所有服务，不允许10.10和11.11为前缀的IP地址被发现（全局过滤） -->
        <blacklist filter-value="10.10;11.11">
            <!-- 表示下面服务，不允许172.16和10.10和11.11为前缀的IP地址被发现 -->
            <service service-name="discovery-springcloud-example-b" filter-value="172.16"/>
        </blacklist>

        <!-- 服务发现的多版本灰度访问控制 -->
        <!-- service-name，表示服务名 -->
        <!-- version-value，表示可供访问的版本，如果多个用“;”分隔，不允许出现空格 -->
        <version>
            <!-- 表示消费端服务a的1.0，允许访问提供端服务b的1.0和1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-a" provider-service-name="discovery-springcloud-example-b" consumer-version-value="1.0" provider-version-value="1.0;1.1"/>
            <!-- 表示消费端服务b的1.0，允许访问提供端服务c的1.0和1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="1.0" provider-version-value="1.0;1.1"/>
            <!-- 表示消费端服务b的1.1，允许访问提供端服务c的1.2版本 -->
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="1.1" provider-version-value="1.2"/>
        </version>
    </discovery>
</rule>
```

### 多版本灰度规则策略
```xml
版本策略介绍
1. 标准配置，举例如下
   <service consumer-service-name="a" provider-service-name="b" consumer-version-value="1.0" provider-version-value="1.0,1.1"/> 表示消费端1.0版本，允许访问提供端1.0和1.1版本
2. 版本值不配置，举例如下
   <service consumer-service-name="a" provider-service-name="b" provider-version-value="1.0,1.1"/> 表示消费端任何版本，允许访问提供端1.0和1.1版本
   <service consumer-service-name="a" provider-service-name="b" consumer-version-value="1.0"/> 表示消费端1.0版本，允许访问提供端任何版本
   <service consumer-service-name="a" provider-service-name="b"/> 表示消费端任何版本，允许访问提供端任何版本
3. 版本值空字符串，举例如下
   <service consumer-service-name="a" provider-service-name="b" consumer-version-value="" provider-version-value="1.0,1.1"/> 表示消费端任何版本，允许访问提供端1.0和1.1版本
   <service consumer-service-name="a" provider-service-name="b" consumer-version-value="1.0" provider-version-value=""/> 表示消费端1.0版本，允许访问提供端任何版本
   <service consumer-service-name="a" provider-service-name="b" consumer-version-value="" provider-version-value=""/> 表示消费端任何版本，允许访问提供端任何版本
4. 版本对应关系未定义，默认消费端任何版本，允许访问提供端任何版本

特殊情况处理，在使用上需要极力避免该情况发生
1. 消费端的application.properties未定义版本号，则该消费端可以访问提供端任何版本
2. 提供端的application.properties未定义版本号，当消费端在xml里不做任何版本配置，才可以访问该提供端
```

### 动态改变规则策略
微服务启动的时候，由于规则（例如：rule.xml）已经配置在本地，使用者希望改变一下规则，而不重启微服务，达到规则的改变 
- 规则分为本地规则和动态规则
- 本地规则是通过在本地规则（例如：rule.xml）文件定义的，也可以从远程配置中心获取，在微服务启动的时候读取
- 动态规则是通过POST方式动态设置，或者由远程配置中心推送设置
- 规则初始化的时候，如果接入了远程配置中心，先读取远程规则，如果不存在，再读取本地规则文件
- 多规则灰度获取规则的时候，先获取动态规则，如果不存在，再获取本地规则

### 动态改变版本策略
微服务启动的时候，由于版本已经写死在application.properties里，使用者希望改变一下版本，而不重启微服务，达到访问版本的路径改变 
- 版本分为本地版本和动态版本
- 本地版本是通过在application.properties里配置的，在微服务启动的时候读取
- 动态版本是通过POST方式动态设置
- 多版本灰度获取版本值的时候，先获取动态版本，如果不存在，再获取本地版本

### 黑/白名单的IP地址注册的过滤策略
微服务启动的时候，禁止指定的IP地址注册到服务注册发现中心。支持黑/白名单，白名单表示只允许指定IP地址前缀注册，黑名单表示不允许指定IP地址前缀注册
- 全局过滤，指注册到服务注册发现中心的所有微服务，只有IP地址包含在全局过滤字段的前缀中，都允许注册（对于白名单而言），或者不允许注册（对于黑名单而言）
- 局部过滤，指专门针对某个微服务而言，那么真正的过滤条件是全局过滤+局部过滤结合在一起

### 最大注册数的限制的过滤策略
微服务启动的时候，一旦微服务集群下注册的实例数目已经达到上限（可配置），将禁止后续的微服务进行注册
- 全局配置值，只下面配置所有的微服务集群，最多能注册多少个
- 局部配置值，指专门针对某个微服务而言，那么该值如存在，全局配置值失效

### 黑/白名单的IP地址发现的过滤策略
微服务启动的时候，禁止指定的IP地址被服务发现。它使用的方式和“黑/白名单的IP地址注册的过滤”一致

### 版本属性字段定义策略
不同的服务注册发现组件对应的版本配置值
```xml
# Eureka config
eureka.instance.metadataMap.version=1.0
eureka.instance.metadataMap.group=xxx-service-group

# 奇葩的Consul配置（参考https://springcloud.cc/spring-cloud-consul.html - 元数据和Consul标签）
# Consul config（多个值用“,”分隔，例如version=1.0,value=abc）
spring.cloud.consul.discovery.tags=version=1.0,group=xxx-service-group

# Zookeeper config
spring.cloud.zookeeper.discovery.metadata.version=1.0
spring.cloud.zookeeper.discovery.metadata.group=xxx-service-group
```

### 功能开关策略
```xml
# 开启和关闭服务注册层面的控制。一旦关闭，服务注册的黑/白名单过滤功能将失效，最大注册数的限制过滤功能将失效。缺失则默认为true
spring.application.register.control.enabled=true
# 开启和关闭服务发现层面的控制。一旦关闭，服务多版本调用的控制功能将失效，动态屏蔽指定IP地址的服务实例被发现的功能将失效。缺失则默认为true
spring.application.discovery.control.enabled=true
```

## 配置中心
### 跟远程配置中心整合
使用者可以跟携程Apollo，百度DisConf等远程配置中心整合，实现规则读取和订阅
- 主动从本地或远程配置中心获取规则
- 订阅远程配置中心的规则更新

继承ConfigAdapter.java
```java
public class MyConfigAdapter extends ConfigAdapter {
    // 从本地获取规则
    @Override
    protected String getLocalContextPath() {
        // 规则文件放在resources目录下
        return "classpath:rule.xml";

        // 规则文件放在工程根目录下
        // return "file:rule.xml";
    }

    // 从远程配置中心获取规则
    @Override
    public InputStream getRemoteInputStream() throws Exception {
        InputStream inputStream = ...;

        return inputStream;
    }

    // 订阅远程配置中心的规则更新（推送策略自己决定，可以所有服务都只对应一个规则信息，也可以根据服务名获取对应的规则信息）
    @PostConstruct
    public void update() throws Exception {
        InputStream inputStream = ...;
        fireRuleUpdated(new RuleUpdatedEvent(inputStream), true);
    }
}
```

## 管理中心
> PORT端口号为server.port或者management.port都可以（management.port开放只支持3.x.x版本）
### 配置接口
### 版本接口
### 路由接口
参考Swagger界面，如下图

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Swagger-1.jpg)

## 独立控制台
为UI提供相关接口
已实现如下功能
- 一系列批量功能

待实现如下功能
- 与远程配置中心整合
- 与UI整合

> PORT端口号为server.port或者management.port都可以（management.port开放只支持3.x.x版本）
### 控制台接口
参考Swagger界面，如下图

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Swagger-2.jpg)

## 扩展和自定义更多规则或者监听
使用者可以继承如下类
- AbstractRegisterListener，实现服务注册的扩展和监听
- AbstractDiscoveryListener，实现服务发现的扩展和监听，注意，在Consul下，同时会触发service和management两个实例的事件，需要区别判断，如下图

集成了健康检查的Consul控制台
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Consul.jpg)

## 示例演示
### 场景描述
本例将模拟一个较为复杂的场景，如下图

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Version.jpg)

- 系统部署情况：
  - 网关Zuul集群部署了1个
  - 微服务集群部署了3个，分别是A服务集群、B服务集群、C服务集群，分别对应的实例数为2、2、3
- 微服务集群的调用关系为网关Zuul->服务A->服务B->服务C
- 系统调用关系
  - 网关Zuul的1.0版本只能调用服务A的1.0版本，网关Zuul的1.1版本只能调用服务A的1.1版本
  - 服务A的1.0版本只能调用服务B的1.0版本，服务A的1.1版本只能调用服务B的1.1版本
  - 服务B的1.0版本只能调用服务C的1.0和1.1版本，服务B的1.1版本只能调用服务C的1.2版本

用规则来表述上述关系
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <discovery>
        <version>
            <!-- 表示网关z的1.0，允许访问提供端服务a的1.0版本 -->
            <service consumer-service-name="discovery-springcloud-example-zuul" provider-service-name="discovery-springcloud-example-a" consumer-version-value="1.0" provider-version-value="1.0"/>
            <!-- 表示网关z的1.1，允许访问提供端服务a的1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-zuul" provider-service-name="discovery-springcloud-example-a" consumer-version-value="1.1" provider-version-value="1.1"/>
            <!-- 表示消费端服务a的1.0，允许访问提供端服务b的1.0版本 -->
            <service consumer-service-name="discovery-springcloud-example-a" provider-service-name="discovery-springcloud-example-b" consumer-version-value="1.0" provider-version-value="1.0"/>
            <!-- 表示消费端服务a的1.1，允许访问提供端服务b的1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-a" provider-service-name="discovery-springcloud-example-b" consumer-version-value="1.1" provider-version-value="1.1"/>
            <!-- 表示消费端服务b的1.0，允许访问提供端服务c的1.0和1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="1.0" provider-version-value="1.0;1.1"/>
            <!-- 表示消费端服务b的1.1，允许访问提供端服务c的1.2版本 -->
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="1.1" provider-version-value="1.2"/>
        </version>
    </discovery>
</rule>
```

上述微服务分别见discovery-springcloud-example字样的8个DiscoveryApplication，分别对应各自的application.properties。这8个应用，对应的版本和端口号如下表

| 微服务 | 服务端口 | 管理端口 | 版本 |
| --- | --- | --- | --- |
| A1 | 1100 | 5100 | 1.0 |
| A2 | 1101 | 5101 | 1.1 |
| B1 | 1200 | 5200 | 1.0 |
| B2 | 1201 | 5201 | 1.1 |
| C1 | 1300 | 5300 | 1.0 |
| C2 | 1301 | 5301 | 1.1 |
| C3 | 1302 | 5302 | 1.2 |
| Zuul | 1400 | 5400 | 1.0 |

独立控制台见discovery-springcloud-example-console，对应的版本和端口号如下表

| 服务端口 | 管理端口 |
| --- | --- |
| 2222 | 3333 |

### 开始演示
- 启动服务注册发现中心，默认是Eureka。可供选择的有Eureka，Zuul，Zookeeper。Eureka，请启动discovery-springcloud-example-eureka下的应用，后两者自行安装服务器
- 根据上面选择的服务注册发现中心，对示例下的discovery-springcloud-example/pom.xml进行组件切换
```xml
<dependency>
    <groupId>com.nepxion</groupId>
    <artifactId>discovery-plugin-starter-eureka</artifactId>
    <!-- <artifactId>discovery-plugin-starter-consul</artifactId> -->
    <!-- <artifactId>discovery-plugin-starter-zookeeper</artifactId> -->
    <version>${discovery.plugin.version}</version>
</dependency>
```
- 根据上面选择的服务注册发现中心，对控制台下的discovery-springcloud-example-console/pom.xml进行组件切换切换

### 服务注册过滤的操作演示
黑/白名单的IP地址注册的过滤
- 在rule.xml把本地IP地址写入到相应地方
- 启动DiscoveryApplicationA1.java
- 抛出禁止注册的异常，即本地服务受限于黑名单的IP地址列表，不会注册到服务注册发现中心；白名单操作也是如此，不过逻辑刚好相反

最大注册数的限制的过滤
- 在rule.xml修改最大注册数为0
- 启动DiscoveryApplicationA1.java
- 抛出禁止注册的异常，即本地服务受限于最大注册数，不会注册到服务注册发现中心

黑/白名单的IP地址发现的过滤
- 在rule.xml把本地IP地址写入到相应地方
- 启动DiscoveryApplicationA1.java和DiscoveryApplicationB1.java、DiscoveryApplicationB2.java
- 你会发现A服务无法获取B服务的任何实例，即B服务受限于黑名单的IP地址列表，不会被A服务的发现；白名单操作也是如此，不过逻辑刚好相反

### 服务发现和负载均衡控制的操作演示
#### 基于图形化方式的多版本灰度访问控制
- 启动discovery-springcloud-example下8个DiscoveryApplication，无先后顺序，等待全部启动完毕
- 启动discovery-springcloud-example-console下ConsoleApplication
- 启动discovery-console-desktop下ConsoleLauncher
  - 在主界面上的工具栏上，点击“显示服务拓扑”按钮，选择服务集群过滤（根据group过滤选择）
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console0.jpg)
  - 显示相关服务在主界面上，如果不过滤，则把在服务注册发现中心所有注册的服务都显示在界面上
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console1.jpg)
  - 在主界面上，选择“example-discovery-springcloud-example-zuul”集群下的服务，右键“执行灰度路由”
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console2.jpg)
  - 在路由界面上点击箭头指向的“添加服务”按钮，并切换下拉菜单，依次把A，B，C三个服务加进去，然后点击“执行路由”按钮，可以看到从Zuul->A服务->B服务->C服务，可以访问的路径，正是想要的结果
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console3.jpg)

进行版本切换的灰度策略
  - 回到主界面，在选择的Zuul服务上，右键“执行灰度发布”
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console4.jpg)
  - 在弹出的界面，在灰度版本的文本框输入1.1，然后点击“更新灰度版本”按钮，那么Zuul服务的版本从1.0切换到1.1，该节点会呈现黄色闪烁，表示正在执行版本灰度
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console5.jpg)
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console6.jpg)
  - 重复“执行灰度路由”的步骤，发现以Zuul为起点访问路径改变了，目的达到。通过“执行灰度发布”界面，点击“清除灰度版本”按钮，回滚到以前访问路径，这里不表述了
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console7.jpg)

进行规则改变的灰度策略
  - 在主界面上，选择“example-discovery-springcloud-example-b”集群下的服务集群，右键“执行灰度发布”，批量改变B1和B2服务的规则
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console8.jpg)
  - 在弹出的界面，在灰度文本框输入如下新规则（操作的逻辑：B服务的所有版本都只能访问C服务3.0版本，而本例中C服务3.0版本是不存在的，意味着这么做B服务不能访问C服务），然后点击“批量更新灰度规则”按钮，那么B1和B2服务的规则进行改变，两个节点会呈现青色闪烁，表示正在执行规则灰度

新XML规则
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <discovery>
        <version>
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="" provider-version-value="3.0"/>
        </version>
    </discovery>
</rule>
```
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console9.jpg)
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console10.jpg)
  - 重复“执行灰度路由”的步骤，发现以Zuul为起点访问路径改变了，目的达到。通过“执行灰度发布”界面，点击“清除灰度规则”按钮，回滚到以前访问路径，这里不表述了
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Console11.jpg)

刷新灰度状态
  - 通过选择一个服务集群，查看它下面的服务有没有在进行版本灰度或者规则灰度，如果有，在界面上会有不同的颜色闪烁，上面已经说明了

#### 基于Rest方式的多版本灰度访问控制
基于服务的操作过程和效果
- 启动discovery-springcloud-example下7个DiscoveryApplication（除去Zuul），无先后顺序，等待全部启动完毕
- 下面URL的端口号，可以是服务端口号，也可以是管理端口号
- 通过版本切换，达到灰度访问控制，针对A服务
  - 1.1 通过Postman或者浏览器，执行POST [http://localhost:1100/routes](http://localhost:1100/routes)，填入discovery-springcloud-example-b;discovery-springcloud-example-c，查看路由路径，如图1，可以看到符合预期的调用路径
  - 1.2 通过Postman或者浏览器，执行POST [http://localhost:1100/version/update](http://localhost:1100/version/update)，填入1.1，动态把服务A的版本从1.0切换到1.1
  - 1.3 通过Postman或者浏览器，再执行第一步操作，如图2，可以看到符合预期的调用路径，通过版本切换，灰度访问控制成功
- 通过规则改变，达到灰度访问控制，针对B服务
  - 2.1 通过Postman或者浏览器，执行POST [http://localhost:1200/config/update-sync](http://localhost:1200/config/update-sync)，发送新的规则XML（内容见下面）
  - 2.2 通过Postman或者浏览器，执行POST [http://localhost:1201/config/update-sync](http://localhost:1201/config/update-sync)，发送新的规则XML（内容见下面）
  - 2.3 上述操作也可以通过独立控制台，进行批量更新，见图5。操作的逻辑：B服务的所有版本都只能访问C服务3.0版本，而本例中C服务3.0版本是不存在的，意味着这么做B服务不能访问C服务
  - 2.4 重复1.1步骤，发现调用路径只有A服务->B服务，如图3，通过规则改变，灰度访问控制成功
- 负载均衡的灰度测试
  - 3.1 通过Postman或者浏览器，执行POST [http://localhost:1100/invoke](http://localhost:1100/invoke)，这是example内置的访问路径示例（通过Feign实现）
  - 3.2 重复“通过版本切换，达到灰度访问控制”或者“通过规则改变，达到灰度访问控制”操作，查看Ribbon负载均衡的灰度结果，如图4
- 其它更多操作，请参考“管理中心”和“路由中心”，不一一阐述了

新XML规则
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <discovery>
        <version>
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="" provider-version-value="3.0"/>
        </version>
    </discovery>
</rule>
```

图1

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Result1.jpg)

图2

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Result2.jpg)

图3

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Result3.jpg)

图4

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Result4.jpg)

图5

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Result5.jpg)

基于网关的操作过程和效果
- 在上面基础上，启动discovery-springcloud-example下DiscoveryApplicationZuul
- 因为Zuul是一种特殊的微服务，所有操作过程跟上面完全一致

图6

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Result6.jpg)

图7

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-plugin-doc/Result7.jpg)