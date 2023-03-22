# Intel SGX SDK/PSW/DCAP: Intel SGX软件开发套件和平台软件服务

Intel SGX Platform Software and Datacenter Attestation Primitives （PSW/DCAP） 在龙蜥生态中为数据中心和云计算平台提供 Intel SGX 技术所需的平台软件服务，如远程证明等。Intel SGX SDK在龙蜥生态中为开发者提供使用 Intel SGX 技术所需的软件开发套件，帮助开发者高效便捷地开发机密计算程序和解决方案。本文介绍如何在Anolis OS 8.6当中基于Intel SGX SDK/PSW/DCAP构建SGX机密计算环境，并演示如何运行示例代码以验证SGX功能。

## 背景信息

Intel® SGX以硬件安全保障信息安全，不依赖固件和软件的安全状态，为用户提供物理级的机密计算环境。Intel® SGX通过新的指令集扩展与访问控制机制实现SGX程序的隔离运行，保障关键代码和数据的机密性与完整性不受恶意软件的破坏。不同于其他安全技术，Intel® SGX的可信根仅包括硬件，避免了基于软件的可信根可能自身存在安全漏洞的缺陷，极大地提升了系统安全保障。

## 检查 SGX 使能状态

构建SGX机密计算环境前，您可以通过cpuid检查SGX使能状态。

1. 安装cpuid。

```sh
sudo yum install -y cpuid
```

2. 检查SGX使能状态。
```sh
cpuid -1 -l 0x7 | grep SGX
```

**说明**  SGX被正确使能后，运行SGX程序还需要SGX驱动。

3. 检查SGX驱动安装情况。
```sh
ls -l /dev/{sgx_enclave,sgx_provision}
```

## 开始构建SGX机密计算环境

为开发SGX程序，您需要在您的机器上安装运行时（runtime）、SDK，并配置远程证明服务。本文以Anolis OS 8.6为例演示构建过程，您也可以直接参考Intel官方提供的[Intel® SGX软件安装指南](https://download.01.org/intel-sgx/latest/linux-latest/docs/Intel_SGX_SW_Installation_Guide_for_Linux.pdf)安装所需的驱动、PSW等。

### 1. 安装SGX运行时
```sh
mkdir -p $HOME/sgx && \
    cd $HOME/sgx && \
    wget https://download.01.org/intel-sgx/latest/linux-latest/distro/Anolis86/sgx_rpm_local_repo.tgz --no-check-certificate && \
    tar zxvf sgx_rpm_local_repo.tgz && \
    sudo yum install -y yum-utils && \
    sudo yum-config-manager --add-repo file://$HOME/sgx/sgx_rpm_local_repo/ && \
    sudo yum install --nogpgcheck -y sgx-aesm-service libsgx-launch libsgx-urts && \
    rm -rf sgx_rpm_local_repo.tar.gz
```

可按需安装更多库：

```sh
sudo yum install --nogpgcheck -y libsgx-ae-le libsgx-ae-pce libsgx-ae-qe3 libsgx-ae-qve \
    libsgx-aesm-ecdsa-plugin libsgx-aesm-launch-plugin libsgx-aesm-pce-plugin \
    libsgx-aesm-quote-ex-plugin libsgx-dcap-default-qpl libsgx-dcap-ql \
    libsgx-dcap-quote-verify libsgx-enclave-common libsgx-launch libsgx-pce-logic \
    libsgx-qe3-logic libsgx-quote-ex libsgx-ra-network libsgx-ra-uefi \
    libsgx-uae-service libsgx-urts sgx-ra-service sgx-aesm-service
```

**说明**  SGX AESM（Architectural Enclave Service Manager）负责管理启动Enclave、密钥配置、远程认证等服务，默认安装路径为/opt/intel/sgx-aesm-service。

### 2. 安装SGX SDK

```sh
cd $HOME/sgx && \
    export SGX_VERSION="2.18.100.3" && \
    wget https://download.01.org/intel-sgx/latest/linux-latest/distro/Anolis86/sgx_linux_x64_sdk_$SGX_VERSION.bin --no-check-certificate && \
    chmod +x sgx_linux_x64_sdk_$SGX_VERSION.bin && \
    echo -e 'n\n\/opt/intel\n' | ./sgx_linux_x64_sdk_$SGX_VERSION.bin && \
    rm -rf sgx_linux_x64_sdk_$SGX_VERSION.bin && \
    source /opt/intel/sgxsdk/environment
```

安装Intel® SGX SDK后，您可以参见[Intel® SGX Developer Reference](https://download.01.org/intel-sgx/latest/linux-latest/docs/)开发SGX程序。

**说明**  Intel® SGX SDK的默认安装目录为`/opt/intel/sgxsdk/`。

### 3. 配置SGX远程证明服务

[Intel® SGX ECDSA远程证明服务](https://software.intel.com/content/www/us/en/develop/topics/software-guard-extensions/attestation-services.html)通过远程证明来获得远程提供商或生产者的信任，为SGX SDK提供下列信息：

  1. SGX certificates：SGX证书。
  2. Revocation lists：已被撤销的SGX证书列表。
  3. Trusted computing base information：可信根信息。

**说明**  Intel Ice Lake仅支持基于Intel SGX DCAP的远程证明方式，不支持基于Intel EPID的远程证明方式，您可能需要适配程序后才能正常使用远程证明功能。更多远程证明的信息，请参见[attestation-service](https://software.intel.com/content/www/us/en/develop/topics/software-guard-extensions/attestation-services.html)。

配置QPL及PCK缓存服务：

1. 安装Quote Provider Library (QPL)。您可以使用自己定制的 QPL 或使用 Intel 提供的默认 QPL (libsgx-dcap-default-qpl)
2. 安装 PCK 缓存服务。 PCK Caching Service的安装配置请参考Intel官方PCCS文档：[SGXDataCenterAttestationPrimitives](https://github.com/intel/SGXDataCenterAttestationPrimitives/tree/master/QuoteGeneration/pccs)
3. 确保 PCK 缓存服务由本地管理员或数据中心管理员正确设置。还要确保引用提供程序库的配置文件（/etc/sgx_default_qcnl.conf）与真实环境一致，例如：

```sh
PCS_URL=https://your_pcs_server:8081/sgx/certification/v3/
```

## 验证SGX功能示例一：启动Enclave

Intel® SGX SDK中提供了SGX示例代码用于验证SGX功能，默认位于`/opt/intel/sgxsdk/SampleCode`目录下。

本节演示其中的启动Enclave示例（SampleEnclave），效果为启动一个Enclave，以验证是否可以正常使用安装的SGX SDK。

### 1. 安装编译工具及相关依赖

```sh
yum install -y gcc-c++
```

### 2. 设置SGX SDK相关的环境变量

```sh
source /opt/intel/sgxsdk/environment
```

### 3. 编译示例代码SampleEnclave

- 进入SampleEnclave目录
```sh
cd /opt/intel/sgxsdk/SampleCode/SampleEnclave
```

- 编译SampleEnclave
```sh
make
```

- 运行编译出的可执行文件
```sh
./app
```
预期的结果为：
```sh
Checksum(0x0x7ffd989a81e0, 100) = 0xfffd4143
Info: executing thread synchronization, please wait...
Info: SampleEnclave successfully returned.
Enter a character before exit ...
```

## 验证SGX功能示例二：SGX远程证明示例

Intel® SGX SDK中提供了SGX示例代码用于验证SGX功能，默认位于/opt/intel/sgxsdk/SampleCode目录下。

本节演示其中的SGX远程证明示例（QuoteGenerationSample、QuoteVerificationSample），效果为生成和验证Quote。该示例涉及被挑战方（在SGX实例中运行的SGX程序）和挑战方（希望验证SGX程序是否可信的一方），其中QuoteGenerationSample为被挑战方生成Quote的示例代码，QuoteVerificationSample为挑战方验证Quote的示例代码。

### 1. 安装编译工具及相关依赖
```sh
yum install -y git
```

### 2. 设置SGX SDK相关的环境变量
```sh
source /opt/intel/sgxsdk/environment
```

### 3. 安装SGX远程证明依赖的包
```sh
yum install --nogpgcheck -y libsgx-dcap-ql-devel libsgx-dcap-quote-verify-devel
```

### 4. 编译被挑战方示例代码QuoteGenerationSample

- 进入QuoteGenerationSample目录

```sh
git clone https://github.com/intel/SGXDataCenterAttestationPrimitives -b DCAP_1.15
cd SGXDataCenterAttestationPrimitives/SampleCode/QuoteGenerationSample
```

- 编译QuoteGenerationSample

```sh
make
```

- 运行编译出的可执行文件生成Quote

```sh
./app
```

预期的结果为：

```sh
sgx_qe_set_enclave_load_policy is valid in in-proc mode only and it is optional: the default enclave load policy is persistent:
set the enclave load policy as persistent:succeed!

Step1: Call sgx_qe_get_target_info:succeed!
Step2: Call create_app_report:succeed!
Step3: Call sgx_qe_get_quote_size:succeed!
Step4: Call sgx_qe_get_quote:succeed!cert_key_type = 0x5
sgx_qe_cleanup_by_policy is valid in in-proc mode only.

 Clean up the enclave load policy:succeed!
```

### 5. 编译挑战方示例代码QuoteVerificationSample

- 进入QuoteVerificationSample目录

```sh
git clone https://github.com/intel/SGXDataCenterAttestationPrimitives -b DCAP_1.15
cd SGXDataCenterAttestationPrimitives/SampleCode/QuoteVerificationSample
```

- 编译QuoteVerificationSample

```sh
make
```

- 生成签名密钥

```sh
openssl genrsa -out Enclave/Enclave_private_sample.pem -3 3072
```

- 对QuoteVerificationSample Enclave进行签名。

```sh
/opt/intel/sgxsdk/bin/x64/sgx_sign sign -key Enclave/Enclave_private_sample.pem -enclave enclave.so -out enclave.signed.so -config Enclave/Enclave.config.xml
```

- 运行编译出的可执行文件验证Quote

```sh
./app
```
预期的结果为：

```sh
Info: ECDSA quote path: ../QuoteGenerationSample/quote.dat

Trusted quote verification:
	Info: get target info successfully returned.
	Info: sgx_qv_set_enclave_load_policy successfully returned.
	Info: tee_get_quote_supplemental_data_version_and_size successfully returned.
	Info: latest supplemental data major version: 3, minor version: 1, size: 336
	Info: App: tee_verify_quote successfully returned.
	Info: Ecall: Verify QvE report and identity successfully returned.
	Info: App: Verification completed successfully.
	Info: Supplemental data Major Version: 3
	Info: Supplemental data Minor Version: 1

===========================================

Untrusted quote verification:
	Info: tee_get_quote_supplemental_data_version_and_size successfully returned.
	Info: latest supplemental data major version: 3, minor version: 1, size: 336
	Info: App: tee_verify_quote successfully returned.
	Info: App: Verification completed successfully.
	Info: Supplemental data Major Version: 3
	Info: Supplemental data Minor Version: 1
```
