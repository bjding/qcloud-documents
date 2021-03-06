# 1. 测试目的
HTTPS服务拥有身份验证，信息加密及完整性校验等优势，但通过新增SSL协议实现安全通信，必然会产生一定的性能损耗，主要包括延时的增加及加解密消耗CPU资源等方面。本文测试了腾讯云https服务在SSL加解密情况下的极限性能数据，供用户与https传统性能数据进行比对和参考。

# 2. 测试环境
- 压力工具：wrk 4.0.2
- 腾讯云底层服务环境：Nginx 1.1.6_1.9.9 + Openssl 1.0.2h
- 安装Nginx机器操作系统信息：Linux TENCENT64.site 3.10.94-1-tlinux2-0036.tl2 #1 SMP Thu Jan 21 03:40:59 CST 2016 x86_64 x86_64 x86_64 GNU/Linux
- 其他压力机器操作系统：Linux TENCENT64.site 2.6.32.43-tlinux-1.0.17-default #1 SMP Tue Nov 17 18:03:12 CST 2015 x86_64 x86_64 x86_64 GNU/Linux

# 3. WebServer集群测试方案
由于单台压力机无法发送足够大的压力测试腾讯云https服务的极限性能，需要采用多台压力机来发送压力，整个测试包含三部分：

1) 压力机集群。用来发送http/https压力，并输出单台机器的压力测试结果 。

2) 中控机，同步控制压力机集群的启动和结束，获取各台机器的压力数据并汇总输出。

3) 测试机，即承载腾讯云https服务的云机器，测试webserver性能，直接返回页面，不需要连接upstream。
连接关系如下表示：

![](https://mc.qcloudimg.com/static/img/45cc4191947ca44b37c47499138c8669/image.jpg)

# 4. HTTPS WebServer测试性能数据

| 连接类型 | Session cache | 包体(bytes) | 加密套件 | 性能(qps) |
|---------|---------|---------|---------|---------|
| 长 | 打开 | 230 | ECDHE-RSA-AES128-GCM-SHA256 | 1617180 |
| 短 | 关闭 | 230 | ECDHE-RSA-AES128-GCM-SHA256 | 65993 |

# 5. 测试结论
由上表可知，腾讯云HTTPS支持SSL加解密，其后端拥有多个服务器集群，单集群的单台云服务器完全握手性能可达到65000 qps，长连接时性能可以达到1600000qps。

普通情况下，HTTPS协议由于使用SSL协议，增加了至少一次完整握手的过程，因此增加延时为2*RTT。此外，SSL对称/非对称加密将消耗大量CPU资源，RSA的解密能力是困扰HTTPS接入的主要难题。

使用腾讯云负载均衡的HTTPS服务，用户无需为SSL加解密单独部署服务，且腾讯云不收取任何额外费用，让用户轻松拥有极强的业务承载能力和防攻击能力。
