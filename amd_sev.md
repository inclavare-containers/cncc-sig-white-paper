# AMD SEV

## 项目位置链接

https://github.com/AMDESE/AMDSEV

## 项目归属SIG

云原生机密计算SIG

## 技术自身介绍

### 背景

机密计算指的是在Secure or Trusted执行环境中进行计算操作，以防止正在做计算的敏感数据被泄露或者修改。机密计算的核心是Trusted Execution Environment (TEE)，TEE即可基于硬件实现、也可基于软件实现，本文主要专注于基于硬件的实现，因为基于硬件的实现可以做到更小的TCB、更高的性能和可靠性
可信计算（TEE） 是基于硬件和密码学原理的隐私计算方案，相比于纯软件解决方案，具有较高的通用性、易用性和较优的性能。其缺点是需要引入可信方，即信任芯片厂 商。此外由于CPU相关实现属于TCB，侧信道攻击也成为不可忽视的攻击向 量，需要关注相关漏洞和研究进展。

### 问题&挑战

随着越来越多的业务上云，端到端的全链路可信或机密正在慢慢成为公有云基础设施的默认要求而不再是一个特性，需要综合利用加密存储、安全网络传输、机密计算来实现对用户敏感数据全生命周期的保护。机密计算是当前业界正在补齐的环节，主流的硬件平台已经部分提供或正在实现对机密计算的支持，包括AMD SEV，Intel TDX和SGX，Arm CCA，IBM SE和RISC-V的KeyStone等。

### 解决方案

通过显式内存分区或建立映射表等机制，将VM和Monitor放到完全独立于Host Kernel/VMM的内存区间 （物理上防止不同区间的内存相互访问）。

### 结果

Confidential Virtual Machine也可以称作Secure Virtual Machine (SVM)，是最终实现机密计算的实体，本身作为一个可信域存在，SVM自身和在其中运行的用户程序可以不受来自SVM之外的非可信特权软件和硬件的的攻击，从而实现对用户的机密数据和代码的保护。
因为是整个虚拟机作为一个可信域，所以对运行于其中的用户程序可以做到透明，无需重构用户程序，对最终用户而言可以实现零成本可信 （Trust Native）

### 技术介绍图片

![image.png](materials/imgs/amd_sev.png)

## 用户情况

- Azure 已经利用AMD SEV-SNP 技术支持了机密计算
https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-vm-overview
- Google GCP 已经利用 AMD SEV 技术发布了两代机密计算实例
https://cloud.google.com/blog/products/identity-security/introducing-google-cloud-confidential-computing-with-confidential-vms
