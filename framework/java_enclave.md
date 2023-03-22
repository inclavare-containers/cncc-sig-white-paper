# Apache Teaclave Java TEE SDK: 面向Java生态的机密计算编程框架

## 项目位置链接

https://github.com/apache/incubator-teaclave-java-tee-sdk

## 归属社区SIG

云原生机密计算SIG

## 技术自身介绍

### 背景

数据在存储和传输状态的安全性通过加解密得到了很好的解决，但数据在运行时是以明文的方式参与计算的，很容易出现泄漏，造成不可估量的风险。机密计算技术正是为了解决运行时数据安全问题而生，它通过处理器提供一个基于芯片的可信执行环境(TEE)，将敏感数据和代码放置在该TEE内执行，TEE对整个计算过程进行严格保护，有效阻止TEE之外的组件(包括操作系统)获取或篡改TEE内的代码和数据，保证敏感代码和数据的安全性。

### 问题与挑战

Intel SGX技术提供了极高安全等级的可信执行环境，但使用该技术需要对已有的应用代码进行改造，SGX SDK只提供了对C语言生态的支持，此外用户需要用.edl文件定义服务接口，开发过程繁琐，对开发者的编程习惯冲击较大，造成开发门槛很高，阻碍了该技术的发展与应用。

### 解决方案

Teaclave Java TEE SDK提供基于Intel SGX技术的Java生态机密计算开发框架。采用Java静态编译技术将Enclave Module编译成Native代码并运行在SGX TEE中，实现对高级语言的支持并最大限度保持较低的系统TCB。屏蔽底层交互细节，用户无须定义edl接口文件。给用户提供一个Pure Java的机密计算开发框架和编译构建工具链，极大降低Intel SGX的开发门槛。

<div align=center><img src="../materials/imgs/java_enclave_technology.png"></div>

将机密计算从C/C++应用生态扩展到Java生态，在不牺牲机密安全性的前提下，极大提升开发效率和用户体验。

## 应用场景

### 场景描述

Teaclave Java TEE SDK可应用于对数据和算法敏感的领域。比如政府部门、金融、区块链、医疗和联邦计算等；

在阿里云DataTrust隐私增强计算平台，Teaclave Java TEE SDK应用在SQL安全审计和文件转加密等两个微服务模块中；

### 应用效果

基于Teaclave Java TEE SDK帮助用户开发Pure Java机密计算应用，并保证系统安全性和性能，提升机密计算应用开发效率与体验。

## 竞品分析

Intel SGX SDK与OpenEnclave仅支持C/C++生态机密计算应用开发，且开发门槛高；

Occlum LibOS技术降低了机密计算开发与部署难度，但系统TCB很大，牺牲了应用部分机密性。
