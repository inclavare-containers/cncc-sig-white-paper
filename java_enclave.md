# JavaEnclave

## 项目位置链接

http://gitlab.alibaba-inc.com/java-tee/JavaEnclave/tree/master

## 归属社区SIG

云原生机密计算SIG

## 技术自身介绍

### 背景

机密计算是通过在基于硬件的受信任执行环境(TEE) 中执行计算来保护使用中的数据。TEE是强制仅执行已授权代码的环境。 TEE外部的任何代码都无法读取或篡改该环境中的任何数据。 机密计算威胁模型旨在消减云提供商和运营商以及租户域中的其他行动者访问正在执行的代码和数据的能力。

### 问题与挑战

基于Intel SGX Enclave硬件的SGX SDK是一种partition编程模型, 它将一个应用分割成Host与Enclave两部分，只将Enclave部分的代码和数据运行在SGX TEE内运行，保证了TCB足够小，可有效降低敏感数据的攻击面。但这种编程模型需要对已有的应用进行分割改造，且需要定义复杂的配置文件等， 导致应用门槛比较高，阻碍了该技术的广泛使用。

### 解决方案

JavaEnclave采用Java静态编译技术将机密计算从C/C++生态扩展到Java生态，并继承了SGX SDK所定义的Host-Enclave Partition编程模型，最大限度的降低了系统TCB，给用户提供一个Pure Java的机密计算开发界面和构建工具链。

### 结果

将机密计算从C/C++应用生态扩展到Java生态，在不牺牲机密安全性的前提下，极大提升开发效率和用户体验。

### 技术介绍图片

![image.png](materials/imgs/java_enclave_technology.png)

## 应用场景

### 场景描述

JavaEnclave可应用在对敏感算法和数据有保护诉求的领域，比如金融、区块链、医疗与联邦计算等；有效防止平台提供商或黑客对敏感数据的非法窃取。

在阿里云DataTrust隐私增强计算平台，JavaEnclave应用在SQL安全审计和加密文件转加密等两个微服务模块。

### 应用效果

采用JavaEnclave可以让用户开发Pure Java的机密计算服务，同时不牺牲整个系统的机密性与性能。

## 竞品分析

Intel SGX SDK与OpenEnclave只支持C/C++机密计算应用，开发难度较大，开发效能比低；

Occlum LibOS技术降低开发难度，兼容性高，但TCB相对较大，牺牲了部分应用机密性；
