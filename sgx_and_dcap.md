# SGX SDK & DCAP

## 项目位置链接

https://github.com/intel/linux-sgx 

https://github.com/intel/SGXDataCenterAttestationPrimitives

## 归属社区SIG

云原生机密计算SIG

## 技术自身介绍

### 背景

机密计算信息安全行业内一项新兴的技术，专注于帮助保护使用中的数据。机密计算旨在加密数据在内存中进行处理，同时降低将其暴露给系统其余部分的风险，从而降低敏感数据暴露的可能性，同时为用户提供更高程度的控制和透明度。在多租户云环境中，机密计算确保敏感数据与系统堆栈的其他特权部分保持隔离。

### 问题&挑战

传统操作系统提供了进程级内存隔离，但是无法保护用户的数据不被特权进程获取； 虚拟化技术基于特权软件Hypervisor对系统资源进行分配与监控，但Hypervisor潜在的软件漏洞有可能会威胁到整个系统; 基于TPM（Trusted Platform Module）的可信架构难以保障程序运行时的可信执行；TrustZone技术为程序提供了两种隔离的执行环境, 但需要硬件厂商的签名验证才能运行在安全执行环境，开发门槛较高。

### 解决方案

英特尔软件防护扩展（Intel Software Guard Extensions，SGX）是一组安全相关的指令，它被内置于一些现代Intel 中央处理器（CPU）中。它们允许用户态及内核态代码定义将特定内存区域，设置为私有区域，此区域也被称作飞地（Enclaves）。其内容受到保护，不能被本身以外的任何进程存取，包括以更高权限级别运行的进程。CPU对受SGX保护的内存进行加密处理。受保护区域的代码和数据的加解密操作在CPU内部动态完成。因此，处理器可以保护代码不被其他代码窥视或检查。

### 结果

SGX提供了硬件指令级安全保障，保障了运行时的可信执行环境, 使恶意代码无法访问与篡改其他程序运行时的保护内容。Intel从第六代CPU开始支持SGX，SGX已经成为学术界的热点，各大云厂商也开始在云上部署基于SGX的应用。

### 技术介绍图片

![image.png](materials/imgs/sgx_dcap_overview.png)

## 应用场景

提供基础机密计算能力的支撑，在Intel SGX和TDX DCAP之上，支撑机密计算运行时，虚拟机，容器等具体的使用，最终让用户方便地将自己的workload运行到一个可信的机密计算环境当中。

## 用户情况

### 用户使用情况

对于龙蜥社区的机密计算应用，除了需要硬件上的支持，也需要软件基础架构的支持。所有的机密计算应用都会依赖于相应机密计算技术底层的软件开发包和运行时库。Intel为SGX和TDX Attestation提供了基础的软件架构支撑：主要包括SGX SDK，SGX PSW/(TDX) DCAP安装包的适配，和Anolis的集成。

### 用户使用效果

在vSGX虚拟机，TDX机密虚拟机，SGX LibOS运行时Occlum和Gramine当中，SGX SDK/PSW/(TDX) DCAP都提供了SGX，SGX 远程证明及TDX远程证明相关的软件支持（如API等）。

### 后续计划

在Anolis OS发布之前，Intel已经在CentOS和Alinux上支持了SGX SDK/PSW/DCAP软件包的构建和完整的测试。在今年第二季度之后，我们又将TDX DCAP和SGX DCAP合并到统一的代码仓库。软件主要功能上准备完毕，安装包适配到Anolis OS的过程可以分为四个主要步骤：
1. 完成相关安装包（RPM）的构建和测试
2. 发布到Intel软件仓库
3. 提供RPM Build Spec
4. 集成到Anolis软件仓库

### 用户证言

Intel在龙蜥社区中将机密计算如SGX和TDX技术落地，服务于我们的客户。 Intel目前提供了以SGX SDK/PSW/DCAP, TDX DCAP软件包为代表的TEE基础架构支撑，和基于LibOS的运行时支持Gramine。并以这些为基础支持了其他LibOS运行时如蚂蚁的Occlum，以及更高层次的机密计算应用，例如SGX虚拟化及SGX/TDX机密容器。
