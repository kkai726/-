# gRPC
<center> 日期: 2022/7/26 </center>

## gRPC是什么
gRPC 是什么可以用官网的一句话来概括 **“A high-performance, open-source universal RPC framework”**

- 多语言：语言中立，支持多语言
- 轻量级、高性能：序列化支持 PB（Protocol Buffer） 和 Json ， PB 是一种语言无关的高级序列化框架。
- 可插拔
- IDL： 基于文件定义服务，通过 proto3 工具生成指定语言的数据结构、服务接口以及客户端Stub。
- 设计理念
- 移动端：基于标准的 HTTP/2 设计，支持双向流、消息头压缩，单 TCP 的多路复用、服务端推送等特性，，这些特性使得 gRPC 在移动端设备上更加省电和节省网络流量。  
  
## gRPC - HealthCheck
  gRPC 有一个标准的健康检测协议， 在 gRPC 的所有语言实现中基本都提供了生成代码和用于设置运行状态的功能。  
***
主动健康检测 health check，， 可以在服务提供者服务不稳定时，被消费者所感知， 临时从负载均衡中摘除，减少错误请求。当服务提供者重新稳定后， health check 成功， 重新加入到消费者的负载均衡，恢复请求。 health check， 同样也被用于外挂方式的容器健康检测， 或者流量检测（k8s liveness & readiness）。  

1. Kubernetes 向 discovery 发起注销请求。
2. Kubernetes 向 App 发送 Sigter 信号，进入优雅推出过程。
3. 其他客户端在2个心跳周期内（最差， 一般是实时的） 退出。
4. Kubernetes 退出超时（一般在10-60s内）， 强制退出 Sigkill。

