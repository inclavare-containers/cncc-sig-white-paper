# Inclavare Containers: 面向机密计算场景的开源容器运行时技术栈

## 项目位置链接

https://github.com/inclavare-containers

## 归属社区SIG

云原生机密计算SIG

## 技术自身介绍

### 背景

机密计算是一种能够通过软件加密算法和硬件HW-TEE保护用户数据和程序的技术。 在云原生场景中， 机密级算能够对计算中的数据提供机密性和完整性保护， 并能够有效防止云提供商和任何高权限的第三方对执行环境中的数据进行窃取和篡改。

### 问题&挑战

对于数据隐私和数据安全度关注高的用户， 不愿意冒风险，把自己的业务部署在公有云上， 因为云提供商具有查看用户隐私数据和业务的可能性， 这也阻止了一些企业上云的脚步。随着机密计算蓬勃发展，硬件厂商提供了HW-TEE硬件加密的方案的涌现， 可以有效解决安全敏感度较高的用户上云顾虑。 对于公有云人们开发和部署很多是以容器的方式进行， 如何让HW-TEE和容器技术相结合，成为云厂商迫切要解决的问题。

### 解决方案

Alibaba和Intel把Intel-SGX和容器技术相结合， 创新性的开发出了机密容器Inclavare containers, 完美兼容容器标准，并能够为客户的数据和程序进行保驾护航， 并且Inclavare containers 容器成为CNCF的第一个机密容器。

![image.png](../materials/imgs/inclavare_overview.png)

如下图所示，Inclavare Containers 中包含多个组件，这些组件可以利用基于硬件支持的 Enclave 技术使可信应用运行在容器中。包含的组件有 rune、shim-rune、Enclave Runtime等。
- rune：rune 是一个命令行工具，用于根据 OCI 规范在容器中生成和运行 Enclave。rune 是在 runc 代码基础上开发的，既可以运行普通 runc 容器也可以运行 Enclave 容器；rune已经写入 OCI 运行时实现列表：
https://github.com/opencontainers/runtimespec/blob/master/implementations.md。
- shim-rune：为容器运行时 rune 提供的 shim，主要负责管理容器的生命周期、把普通镜像转换成 TEE 镜像；管理容器的生命周期，与 rune 配合完成容器的创建、启动、停止、删除等操作。
- Enclave Runtime：负责在 Enclave 内加载和运行受信任和受保护的应用程序。rune 和 Enclave Runtime 之间的接口是 Enclave Runtime PAL API，它允许通过定义良好的函数接口来调用 Enclave Runtime。机密计算应用通过这个接口与云原生生态系统进行交互。

一类典型的 Enclave Runtime 实现基于库操作系统。目前，推荐的与 rune 交互的 Enclave Runtime 是 Occlum，这是一种内存安全、多进程 Libos。另一类典型的 Enclave Runtime是带有 Intel® SGX WebAssembly Micro Runtime (WAMR)，这是一个占用空间很小的独立 WebAssembly (WASM) 运行时，包括一个 VM 核心、一个应用程序框架和一个 WASM 应用程序的动态管理。

此外，您可以使用您喜欢的任何编程语言和 SDK（例如英特尔 SGX SDK）编写自己的Enclave Runtime，只要它实现了 Enclave Runtime PAL API。


Inclavare Contianers主要有以下特点：

1、将 Intel® SGX 技术与成熟的容器生态结合，将用户的敏感应用以 Enclave 容器的形式部署和运行；Inclavare Contianers 的目标是希望能够无缝运行用户制作的普通容器镜像，这将允许用户在制作镜像的过程中，无需了解机密技术所带来的复杂性，并保持与普通容器相同的使用体感。

2、Intel® SGX 技术提供的保护粒度是应用而不是系统，在提供很高的安全防护手段的同时，也带来了一些编程约束，比如在 SGX enclave 中无法执行 syscall 指令；因此我们引入了 LibOS 技术，用于改善上述的软件兼容性问题，避免开发者在向 Intel® SGX Enclave 移植软件的过程中，去做复杂的软件适配工作。然后，虽然各个 LibOS 都在努力提升对系统调用的支持数量，但这终究难以企及原生 Linux 系统的兼容性，并且即使真的达成了这个目标，攻击面过大的缺点又会暴露出来。因此，Inclavare Containers 通过支持 Java 等语言Runtime 的方式，来补全和提升 Enclave 容器的泛用性，而不是将 Enclave 容器的泛用性绑定在“提升对系统调用的支持数量” 这一单一的兼容性维度上；此外，提供对语言 Runtime 的支持，也能将像 Java 这样繁荣的语言生态引入到机密计算的场景中，以丰富机密计算应用的种类和数量。

3、通过定义通用的 Enclave Runtime PAL API 来接入更多类型的 Enclave Runtime，比如 LibOS 就是一种 Enclave Runtime 形态；设计这层 API 的目标是为了繁荣 Enclave Runtime 生态，允许更多的 Enclave Runtime 通过对接 Inclavare Containers 上到云原生场景中，同时给用户提供更多的技术选择。

## 应用场景

作为业界首个面向机密计算场景的开源容器运行时，Inclavare Containers 为ACK-TEE（ACK-Trusted Execution Environment）提供了使用机密容器的最佳实践。ACK-TEE 依托 Inclavare Containers，能够无缝地运行用户制作的机密容器镜像，并保持与普通容器相同的使用体感。ACK-TEE 可被广泛应用于各种隐私计算的场景，包括：区块链、安全多方计算、密钥管理、基因计算、金融安全、AI、数据租赁。
