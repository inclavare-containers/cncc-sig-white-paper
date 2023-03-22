# CSV机密容器

## 介绍

[Kata Containers ](https://github.com/confidential-containers/kata-containers-CCv0)是一个使用虚拟化来提供隔离层的开源安全容器项目。Kata支持机密计算硬件技术，通过利用可信执行环境(Trusted Execution Environments)来保护客户的高度敏感的工作负载，以防止不受信任的实体（例如：云服务商）访问租客的敏感数据。

在kata container中，您能够在在机密虚拟机中运行POD和容器，将主机/owner/管理员/CSP软件栈从kata 的TCB中移除，从而构建一个更强大的云原生多租户架构。具体做法是：在为每个租户使用的容器加密镜像远程Provisioning解密密钥前，先认证pod或容器是否已经运行在了经过认证的环境中。

海光CPU支持安全虚拟化技术CSV(China Secure Virtualization)，CSV的设计目标是通过CSV虚拟机提供可信执行环境，适用的场景包括云计算、机密计算等。海光2号支持CSV1技术，提供虚拟机内存加密能力，采用国密SM4算法，不同的CSV虚拟机使用不同的加密密钥，密钥由海光安全处理器管理，主机无法解密虚拟机的加密内存。

海光CSV加密虚拟机支持两种远程认证方式：

- pre-attestation指需要在启动TEE之前能够执⾏行行attestation过程。在云上的CSV虚拟机启动的时候，对加载的ovmf，kernel，initrd，cmdline进行静态度量，生成measurement。线下的远程证明验证者对measurement进行验证，确保启动的CSV虚拟机是符合预期的。
- runtime-attestation：在云上的CSV虚拟机运行的时候，产生带有硬件可执行环境的Quote的TLS证书。线下的远程证明验证者对TLS证书进行验证，确保CSV虚拟机运行在TEE中。

