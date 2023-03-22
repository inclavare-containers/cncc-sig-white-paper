# 云原生机密计算SIG概述

随着通信、网络和计算机技术的持续演进与广泛应用，数据资源的开放共享、交换流通成为推动“万物互联、智慧互通”的重要趋势。与此同时，近年来数据安全事件频发、数据安全威胁日趋严峻，数据的安全处理和流通受到了国内外监管部门的广泛重视。**如何在保障安全的前提下最大程度发挥数据的价值，是当前面临的重要课题。**

在日益严苛的隐私保护相关法律法规约束下，作为当前数据处理基础设施的云计算也正在经历一次重大的范式转换，即从默认以 CSP 为信任基础的计算范式走向信任链与 CSP 解耦的新范式。我们将此范式称为隐私保护云计算，而机密计算是实现隐私保护云计算的必由之路。

为拥抱隐私保护云计算新范式，促进隐私保护云计算生态发展，云原生机密计算SIG应运而生：

![image.png](materials/imgs/overview.png)

## 愿景

云原生机密计算SIG致力于通过开源社区合作共建的方式，为业界提供开源和标准化的机密计算技术以及安全架构，推动云原生场景下机密计算技术的发展。工作组将围绕下述核心项目构建云原生机密计算开源技术栈，降低机密计算的使用门槛，简化机密计算在云上的部署和应用步骤，拓展使用场景及方案。

云原生机密计算SIG的愿景是：

1）构建安全、易用的机密计算技术栈

2）适配各种常见机密计算硬件平台

3）打造典型机密计算产品和应用案例

## 项目介绍

### 海光 CSV 机密容器

CSV 是海光研发的安全虚拟化技术。CSV1 实现了虚拟机内存加密能力，CSV2 增加了虚拟机状态加密机制，CSV3 进一步提供了虚拟机内存隔离支持。CSV 机密容器能够为用户提供虚拟机内存加密和虚拟机状态加密能力，主机无法解密获取虚拟机的加密内存和加密状态信息。CSV 虚拟机使用隔离的 TLB、Cache 等硬件资源，支持安全启动、代码验证、远程认证等功能。

- 主页：https://openanolis.cn/sig/coco/doc/533508829133259244

### Intel Confidential Computing Zoo

Intel 发起并开源了 Confidential Computing Zoo (CCZoo)，CCZoo 基于 Intel TEE（SGX,TDX）技术，提供了不同场景下各种典型端到端安全解决方案的参考案例，增加用户在机密计算方案实现上的开发体验，并引导用户结合参考案例快速设计自己特定的机密计算解决方案。 CCZoo 目前提供了基于 Libos + Intel TEE + OpenAnolis 容器的 E2E 安全解决方案参考案例，后续，CCZoo 计划基于 OpenAnolis ，提供更多的机密计算参考案例，为用户提供相应的容器镜像，实现敏捷部署。

- 主页：https://cczoo.readthedocs.io
- 代码库：https://github.com/intel/confidential-computing-zoo

### Intel HE Toolkit

Intel HE Toolkit 旨在为社区和行业提供一个用于实验、开发和部署同态加密应用的平台。目前 Intel HE Toolkit 包括了主流的 Leveled HE 库，如 SEAL、Palisade和 HELib，基于使能了英特尔最新指令集加速的的 Intel HEXL 库，在英特尔至强处理器平台上为同态加密业务负载提供了卓越的性能体验。同时，Intel HE Toolkit即将集成半同态 Paillier 加速库 IPCL，为半同态加密应用提供加速支持。此外，Intel HE Toolkit 还提供了示例内核、示例程序和基准测试 HEBench。这些示例程序演示了利用主流的同态加密库构建各种同态加密应用保护用户隐私数据的能力。HEBench 则为各类第三方同态加密应用提供了公允的评价基准，促进了同态加密领域的研究与创新。

- 主页：https://www.intel.com/content/www/us/en/developer/tools/homomorphic-encryption/
- 代码库：
    - Intel HE Toolkit: https://github.com/intel/he-toolkit
    - Intel HEXL: https://github.com/intel/hexl
    - Intel Paillier Cryptosystem Library (IPCL): https://github.com/intel/pailliercryptolib
    - HE Bench: https://github.com/hebench

### Intel SGX Platform Software and Datacenter Attestation Primitives

在龙蜥生态中为数据中心和云计算平台提供 Intel SGX 技术所需的平台软件服务，如远程证明等。
- RPM包：https://download.01.org/intel-sgx/latest/linux-latest/distro/Anolis86/
- 代码库：https://github.com/intel/SGXDataCenterAttestationPrimitives

### Intel SGX SDK

在龙蜥生态中为开发者提供使用 Intel SGX 技术所需的软件开发套件，帮助开发者高效便捷地开发机密计算程序和解决方案。
- RPM包：https://download.01.org/intel-sgx/latest/linux-latest/distro/Anolis86/
- 代码库：https://github.com/intel/linux-sgx

### Occlum

Occlum 是一个 TEE LibOS，是机密计算联盟（CCC, Confidential Computing Consortium）的官方开源项目。目前 Occlum 支持 Intel SGX 和 HyperEnclave 两种 TEE。Occlum 在 TEE 环境中提供了一个兼容 Linux 的运行环境，使得 Linux 下的应用可以不经修改就在 TEE 环境中运行。Occlum 在设计时将安全性作为最重要的设计指标，在提升用户开发效率的同时保证了应用的安全性。Occlum 极大地降低了程序员开发 TEE 安全应用的难度，提升了开发效率。
- 主页：https://occlum.io/
- 代码库：https://github.com/occlum/occlum

### KubeTEE Enclave Services

提供 TEE 有关的 Kubernetes 基础服务 (如集群规模的密钥分发和同步服务、集群远程证明服务等），使得用户可以方便地将集群中多台 TEE 机器当作一个更强大的 TEE 来使用。
- 代码库：https://github.com/SOFAEnclave/KubeTEE

### Apache Teaclave Java TEE SDK

Apache Teaclave Java TEE SDK(JavaEnclave)是一个面向 Java 生态的机密计算编程框架，它继承Intel SGX SDK所定义的Host-Enclave机密计算分割编程模型。JavaEnclave提供一种十分优雅的模式，对一个完整的Java应用程序进行分割与组织。它将一个Java项目划分成三个子模块，Common子模块定义SPI服务接口，Enclave子模块实现SPI接口并以Provider方式提供服务，Host子模块负责TEE环境的管理和Enclave机密服务的调用。整个机密计算应用的开发与使用模式符合Java经典的SPI设计模式，极大降低了Java机密计算开发门槛。此外，本框架创新性应用Java静态编译技术，将Enclave子模块Java代码编译成Native形态并运行在TEE环境，极大减小了Enclave攻击面，杜绝了Enclave发生注入攻击的风险，实现了极致安全的Java机密计算运行环境。
- 主页: https://teaclave.apache.org
- 代码库: https://github.com/apache/incubator-teaclave-java-tee-sdk

### Gramine

Gramine 是一个轻量级的 LibOS，旨在以最小的主机要求运行单个应用程序。Gramine 可以在一个隔离的环境中运行应用程序。其优点是可定制，易移植，方便迁移，可以媲美虚拟机。
在架构上 Gramine 可以在任何平台上支持运行未修改的 Linux 二进制文件。目前，Gramine 可以在 Linux 和 Intel SGX enclave 环境中工作。
- 主页：https://gramine.readthedocs.io/
- 代码库：https://github.com/gramineproject/gramine