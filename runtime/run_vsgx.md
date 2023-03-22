# Intel SGX虚拟机最佳实践

本文主要为您介绍如何在 SGX 裸金属/物理机环境中启动一个 SGX 虚拟机，以及如何在 SGX 虚拟机中构建 SGX 加密计算环境，并演示如何运行示例代码以验证 SGX 功能。

## 背景信息

Intel® SGX 是一组用于内存访问的指令和机制，以便为敏感应用程序和数据提供安全访问。 SGX 允许应用程序将其特定的地址空间用作 enclave，这是一个受保护的区域，即使在存在特权恶意软件的情况下也能提供机密性和完整性。 阻止任何不驻留在 enclave 中的软件访问 enclave 内存区域，包括来自特权软件的访问。

用户可以使用 Linux 内核中 KVM 虚拟化模块和 QEMU 在支持英特尔 SGX 的硬件上创建一个虚拟机，该虚拟机可以在 Guest 操作系统中使用Intel SGX and Intel SGX Data Center Attestation Primitives (Intel SGX DCAP) 。
 
## 前提条件

### 1. 检查 BIOS 是否使能 SGX

请使用 BIOS 使能 SGX 功能的 SGX 裸金属。

以浪潮 NF5280M6 SGX 机型为例，请参考下图的步骤检查 SGX 是否使能。

#### 1.1. Total Memory Encription

Main menu->Socket Configuration->Processor Configuration->Total Memory Encryption
->Enabled


#### 1.2. SW Guard Extensions(SGX)

Main menu->Socket Configuration->Processor Configuration-> SW Guard Extensions(SGX)
->Enabled

#### 1.3. PRMRR Size

Main menu->Socket Configuration->Processor Configuration-> SW Guard Extensions(SGX)
->PRMRR Size 推荐设置为 32 G （PRMRR 决定单 socket 上预留 EPC 的大小）

### 2. 安装 Anolis 8.6 操作系统

请参考 [Anolis 8.6 GA 说明文档](https://mirrors.openanolis.cn/anolis/8.6/isos/ReadMe.txt)安装 Anolis 8.6 GA。

### 3. 升级 kernel 到 5.10

- 安装 kernel 5.10 


```sh
yum-config-manager --add-repo https://mirrors.openanolis.cn/anolis/8/kernel-5.10/x86_64/os/ && \
  yum update kernel-5.10.134-12.1.an8
```

> 提示： 若提示`-bash: yum-config-manager: command not found` ，请参考以下命令安装yum-utils。

```sh
yum install -y yum-utils
```

- 重启机器之后，请输入以下命令查看内核版本。

```sh
uname -r
```

### 4. 检查 Host 侧的 SGX 使能状态

- 检查 SGX 使能状态。

```sh
dmesg | grep -i sgx
```

以下输出表示 SGX 已经被正确使能。

```sh
[    2.326326] sgx: EPC section 0x80c000000-0xfff7fffff
[    2.391710] sgx: EPC section 0x180c000000-0x1fffffffff
[    2.456785] sgx: EPC section 0x280c000000-0x2fffffffff
[    2.522183] sgx: EPC section 0x380c000000-0x3fffffffff
```

- 检查 SGX 驱动安装情况。

```sh
ls /dev/sgx*
```

以下输出表示已经安装 SGX 驱动。

```sh
/dev/sgx_enclave  /dev/sgx_provision  /dev/sgx_vepc
```

## 步骤一：安装QEMU

如果系统 QEMU 的版本低于 v6.2.0，请执行以下命令安装 QEMU v6.2.0。更多详细信息请参考[qemu](https://github.com/qemu/qemu/blob/master/docs/system/i386/sgx.rst?spm=ata.21736010.0.0.51d463c2XdwdzN&file=sgx.rst)。

```sh
export version=6.2.0-11.0.1.module+an8.6.0+10830+f21fb638.5
yum install -y qemu-img-$version \
	qemu-guest-agent-$version \
	qemu-kvm-$version \
	qemu-kvm-common-$version \
	qemu-kvm-core-$version \
	qemu-kvm-hw-usbredir-$version \
	qemu-guest-agent
```

## 步骤二：下载 Guest OS镜像

- 请参考以下命令从龙蜥社区下载 Anolis 8.6 的 Guest OS 镜像。

```sh
cd $HOME/vsgx/ && \
 	wget https://mirrors.openanolis.cn/anolis/8.6/isos/GA/x86_64/AnolisOS-8.6-x86_64-ANCK.qcow2
```

- 修改 Guest 镜像的登陆密码.

> 说明：以下命令行将密码设置为`123456`， 请根据需求自行设置登陆密码。

```sh
yum install -y libguestfs-tools-c
export LIBGUESTFS_BACKEND=direct
virt-customize -a AnolisOS-8.6-x86_64-ANCK.qcow2 --root-password password:123456
```

- 结果显示：

```sh
[   0.0] Examining the guest ...
[   6.5] Setting a random seed
[   6.5] Setting passwords
[   7.6] Finishing off

```

## 步骤三：启动 SGX Guest

### 启动方式一：使用 QEMU 命令行启动

- 请输入以下 QEMU 命令来启动 SGX Guest。

```sh
/usr/libexec/qemu-kvm \
 	-enable-kvm \
	-cpu host,+sgx-provisionkey -smp 8,sockets=1 -m 16G -no-reboot \
 	-drive file=$HOME/vsgx/AnolisOS-8.6-x86_64-ANCK.qcow2,if=none,id=disk0,format=qcow2 \
	-device virtio-scsi-pci,id=scsi0,disable-legacy=on,iommu_platform=true -device scsi-hd,drive=disk0 -nographic \
	-monitor pty -monitor unix:monitor,server,nowait \
	-object memory-backend-epc,id=mem1,size=64M,prealloc=on \
	-M sgx-epc.0.memdev=mem1,sgx-epc.0.node=0
```

- 请输入步骤二设置的用户名和密码，进入SGX Guest。

```sh
localhost login: root
Password: 123456
```

### 启动方式二：使用 virsh 启动 sgx guest

virsh 是用于管理 虚拟化环境中的客户机和 Hypervisor 的命令行工具，与 virt-manager 等工具类似，它也是通过 libvirt API 来实现虚拟化的管理。virsh 是完全在命令行文本模式下运行的用户态工具，它是系统管理员通过脚本程序实现虚拟化自动部署和管理的理想工具之一。

#### 安装 libvirt

> 说明：目前 Anolis 的 libvirt 不支持 sgx 功能，需要打上 intel 的 6 个 patch。目前基于[intel 提供的代码](https://github.com/hhb584520/libvirt/commits/v13)自行打包，由 inclavare-containers repo 进行管理。

```sh
cd $HOME/vsgx && \
	wget https://mirrors.openanolis.cn/inclavare-containers/bin/anolis8.6/libvirt-8.5.0/libvirt-with-vsgx-support.tar.gz && \
	tar zxvf libvirt-with-vsgx-support.tar.gz && \
	cd libvirt-with-vsgx-support && \
	yum localinstall -y *.rpm
```

#### 在 libvirt 中配置 QEMU

在 QEMU 中构建英特尔 SGX 环境，需要访问以下设备：

- `/dev/sgx_enclave` 启动enclave
- `/dev/sgx_provision` 启动供应认证enclave (PCE)
- `/dev/sgx_vepc `分配 EPC 内存页

libvirt 默认启用的 cgroup 控制器将拒绝访问这些设备文件。编辑`/etc/libvirt/qemu.conf `并更改`cgroup_device_acl` 列表已包括所有三个：

```sh
cgroup_device_acl = [
    "/dev/null", "/dev/full", "/dev/zero",
    "/dev/random", "/dev/urandom",
    "/dev/ptmx", "/dev/kvm",
    "/dev/rtc","/dev/hpet",
    "/dev/sgx_enclave", "/dev/sgx_provision", "/dev/sgx_vepc"
]
```

QEMU 还需要读取和写入`/dev/sgx_vepc` 设备，该设备由 root 拥有，文件模式为 600。这意味着您必须将 QEMU 配置为以 root 身份运行。 请编辑 `/etc/libvirt/qemu.conf`，并设置用户参数。

```sh
user =“root”
```

- 进行这些更改后，您需要重新启动 libvirtd 服务：

```sh
systemctl restart libvirtd
```

#### 配置 NAT 网络

重启 libvirtd 服务之后，请检查当前的网络设置。

```sh
# virsh net-list --all
 名称        状态     自动开始   持久
---------------------------------------
 default     活动     否         否
```

libvirt 默认使用了一个名为 `default` 的 `NAT` 网络，这个网络默认使用 virbr0 作为桥接接口，使用 dnsmasq 来为使用 nat 网络的虚拟机提供 dns 及 dhcp 服务。


如果您需要自定义 libvirt 虚拟网络，请参考[libvirt 网络管理](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/chap-network_configuration#doc-wrapper)。

#### 创建 SGX Guest xml 文件

以下是 SGX 虚拟机的参考 xml 文件 `vsgx.xml`, 在使用过程中，请根据实际需求，修改对应的配置字段。

> 说明：假设 guest image 的位置为`/root/vsgx/AnolisOS-8.6-x86_64-ANCK.qcow2`, 使用名为`default` 的 `NAT` 网络。

```yaml
<domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
  <name>vsgx</name>
  <memory unit='KiB'>16777216</memory>
  <currentMemory unit='KiB'>16777216</currentMemory>
  <vcpu placement='static'>8</vcpu>
  <os>
    <type arch='x86_64'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <pae/>
  </features>
  <clock offset='localtime'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <qemu:commandline>
     <qemu:arg value='-cpu'/>
     <qemu:arg value='host,+sgx-provisionkey'/>
     <qemu:arg value='-object'/>
     <qemu:arg value='memory-backend-epc,id=mem1,size=64M,prealloc=on'/>
     <qemu:arg value='-M'/>
     <qemu:arg value='sgx-epc.0.memdev=mem1,sgx-epc.0.node=0'/>
  </qemu:commandline>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/root/vsgx/AnolisOS-8.6-x86_64-ANCK.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </disk>
    <controller type='ide' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
    </controller>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </memballoon>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <interface type='network'>
      <source network='default'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
  </devices>

<feature>
  <sgx supported='yes'>
    <flc>yes</flc>
    <epc_size unit='KiB'>1048576</epc_size>
  </sgx>
</feature>
</domain>
```

#### 启动虚拟机

```sh
# virsh create vsgx.xml
Domain 'vsgx' created from vsgx.xml
```

成功启动虚拟机之后，请输入一下命令列出虚拟机实例。

```sh
# virsh list
 Id   名称   状态
----------------------
 1    vsgx   running
```

#### 进入虚拟机控制台

```sh
# virsh  console vsgx
Connected to domain 'vsgx'
Escape character is ^] (Ctrl + ])
# 这里要敲一下回车键
Anolis OS 8.6
Kernel 4.19.91-26.an8.x86_64 on an x86_64

Activate the web console with: systemctl enable --now cockpit.socket

localhost login:
```

- 请输入步骤二设置的用户名和密码，进入SGX guest。

```sh
localhost login: root
Password: 123456
```

### 检查 Guest 的 SGX 使能状态

不管是用 QEMU 命令行直接启动的 SGX Guest，还是使用 virsh 启动的 SGX Guest，在启动之后，都需要检查 Guest 中对 SGX 是否支持。

在 Guest 中使用 SGX 需要支持 SGX 的内核/操作系统。可以通过以下方式在 Guest 中确定支持：

- 检查 SGX 使能状态。

```sh
dmesg | grep -i sgx
```

以下输出表示 SGX 已经被正确使能。

```sh
[    0.489460] sgx: EPC section 0x440000000-0x443ffffff
```

- 检查 SGX 驱动安装情况。

```sh
ls /dev/sgx_*
```

以下输出表示已经安装 SGX 驱动。

```sh
/dev/sgx_enclave  /dev/sgx_provision
```

## 步骤四：构建 SGX 加密计算环境

为开发 SGX 程序，您需要在 SGX 虚拟机上安装 SGX SDK，PSW 和 DCAP 进而构建 SGX 加密计算环境。

- 安装 SGX SDK

```sh
mkdir -p $HOME/vsgx && \
	wget https://mirrors.openanolis.cn/inclavare-containers/bin/anolis8.6/sgx-2.17/sgx_linux_x64_sdk_2.17.100.3.bin && \
	chmod +x sgx_linux_x64_sdk_2.17.100.3.bin && \
	echo -e 'n\n\/opt/intel\n' | ./sgx_linux_x64_sdk_*.bin && \
	rm -rf sgx_linux_x64_sdk_*.bin
```

- 安装 SGX PSW/DCAP

```sh
cd $HOME/vsgx && \
	wget https://mirrors.openanolis.cn/inclavare-containers/bin/anolis8.6/sgx-2.17/sgx_rpm_local_repo.tar.gz && \
	tar zxvf sgx_rpm_local_repo.tar.gz && \
	yum install -y yum-utils && \
	yum-config-manager --add-repo file://$HOME/vsgx/sgx_rpm_local_repo/ && \
	yum install --nogpgcheck -y sgx-aesm-service libsgx-launch libsgx-urts && \
	rm -rf sgx_rpm_local_repo.tar.gz
```

## 步骤五：验证 SGX 功能示例一：启动 Enclave

Intel SGX SDK 中提供了 SGX 示例代码用于验证 SGX 功能，默认位于 opt/intel/sgxsdk/SampleCode 目录下。

本节演示其中的启动 Enclave 示例（SampleEnclave），效果为启动一个 Enclave，以验证是否可以正常使用安装的 SGX SDK。

- 安装编译工具

```sh
yum install -y gcc-c++
```

- 设置 SGX SDK 相关的环境变量。

```sh
source /opt/intel/sgxsdk/environment
```

- 编译示例代码 SampleEnclave

```sh
cd /opt/intel/sgxsdk/SampleCode/SampleEnclave && \
	make
```

- 运行编译出的可执行文件。

```sh
./app
```

预期输出：

```sh
Checksum(0x0x7ffc732e8d20, 100) = 0xfffd4143
Info: executing thread synchronization, please wait...
Info: SampleEnclave successfully returned.
Enter a character before exit ...
```

## 步骤六：停止虚拟机

### QEMU 停止虚拟机

在 SGX Guest 里输入`exit` 退出 Guest 虚拟机，然后 `CTRL+C` 终止 QEMU 进程。

### vrish 停止虚拟机

#### 停止虚拟机

请在 Host 机器上，输入以下命令停止虚拟机。

```sh
# virsh shutdown vsgx
Domain 'vsgx' is being shutdown
```

#### 删除虚拟机

请在 Host 机器上，输入以下命令删除虚拟机。

```sh
# virsh undefine vsgx
Domain 'vsgx' has been undefined
```

#### 强制停止虚拟机

> 说明：仅限 shutdown 不工作时才使用 destroy。

```sh
virsh destroy vsgx
```
