**Spring 框架提供的 RestTemplate 类可用于在应用中调用 rest 服务，它简化了与http服务的通信方式，统一了Restful的标准，封装了http 链接， 我们只需要传入url 及返回值类型即可。**

使用 RestTemplate ，来搭建一个微服务项目。现在有两个模块：

1. 服务提供模块（模块名：`a01-cloud-provider-payment8001`）
2. 服务消费模块（模块名：`a02-cloud-consumer-order80`）

