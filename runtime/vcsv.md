# 海光CSV机密虚拟机

本文主要为您介绍如何在 CSV裸金属/物理机环境中启动一个 CSV 虚拟机。

## 背景信息

海光CPU支持安全虚拟化技术CSV(China Secure Virtualization)，CSV的设计目标是通过CSV虚拟机提供可信执行环境，适用的场景包括云计算、机密计算等。海光2号支持CSV1技术，提供虚拟机内存加密能力，采用国密SM4算法，不同的CSV虚拟机使用不同的加密密钥，密钥由海光安全处理器管理，主机无法解密虚拟机的加密内存。

## 步骤一：安装edk2-ovmf

请执行如下命令安装edk2-ovmf。OVMF是一个基于EDK II的项目，用于虚拟机的UEFI支持，更多详细信息请参考[OVMF](https://github.com/tianocore/tianocore.github.io/wiki/OVMF)

```sh
yum install -y edk2-ovmf
```

## 步骤二：安装qemu-kvm

请执行以下命令安装 qemu-kvm。Qemu是一款开源的模拟器和虚拟机监视器，可用于虚拟机管理、加速，可以用来虚拟X86、Power、Arm、MIPS等平台，具有高速、跨平台的特性。qemu广泛应用于虚拟机管理、仿真等领域。更多详细信息请参考[qemu](https://github.com/qemu/qemu)

```sh
yum install -y qemu-kvm
```

## 步骤三：安装libvirt

请执行以下命令安装libvirt。libvirt是一套用于管理虚拟化的开源API、守护进程与管理工具。此套组件可用于管理KVM、Xen、VMware ESXi、QEMU及其他虚拟化技术。libvirt内置的API广泛用于云解决方案开发中的虚拟机监视器编排层（Orchestration Layer）。更多详细信息请参考[libvirt](https://libvirt.org/index.html)

```sh
yum install -y libvirt
```

## 步骤四：下载Guest OS镜像

- 请参考以下命令从龙蜥社区下载 Anolis 8.6 的 Guest OS 镜像：

```sh
cd /tmp
wget https://anolis.oss-cn-hangzhou.aliyuncs.com/anolisos_8_6_x64_20G_anck_uefi_community_alibase_20220817.vhd
```

- 将vhd格式镜像转换为qcow2格式：

```sh
qemu-img convert -p -f vpc anolis_8_4_x64_20G_uefi_anck_community_alibase_20220418.vhd -O qcow2 test.qcow2
```

## 步骤五：启动CSV guest虚拟机

### 启动方式一：使用 QEMU 命令行启动

请参考如下qemu命令行启动CSV guest虚拟机：

```sh
sudo /usr/libexec/qemu-kvm -enable-kvm -cpu host -smp 4 -m 4096 -drive if=pflash,format=raw,unit=0,file=/usr/share/edk2/ovmf/OVMF_CODE.cc.fd,readonly=on  -hda test.qcow2 -object sev-guest,id=sev0,policy=0x1,cbitpos=47,reduced-phys-bits=5 -machine memory-encryption=sev0 -name test -monitor stdio
```

### 启动方式二：使用virsh启动

virsh 是用于管理 虚拟化环境中的客户机和 Hypervisor 的命令行工具，与 virt-manager 等工具类似，它也是通过 libvirt API 来实现虚拟化的管理。virsh 是完全在命令行文本模式下运行的用户态工具，它是系统管理员通过脚本程序实现虚拟化自动部署和管理的理想工具之一。

#### 配置libvirt

请将/etc/libvirt/qemu.conf中的user和group设置为root，以免出现权限问题和报错：

```sh
442 # Some examples of valid values are:
443 #
444 #       user = "qemu"   # A user named "qemu"
445 #       user = "+0"     # Super user (uid=0)
446 #       user = "100"    # A user named "100" or a user with uid=100
447 #
448 user = "root"
449
450 # The group for QEMU processes run by the system instance. It can be
451 # specified in a similar way to user.
452 group = "root"
453
```

#### 重启libvirtd服务

```sh
systemctl daemon-reload
service libvirtd restart
```

#### 创建CSV guest配置文件

以下是CSV虚拟机的参考配置文件csv_launch.xml，在使用过程中，请根据实际需求，修改对应的配置字段。更多配置请参考[Launch security with AMD SEV](https://libvirt.org/kbase/launch_security_sev.html)

```sh
<domain type = 'kvm'  xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
    <name>csv_launch</name>
    <memory unit='GiB'>4</memory>
    <vcpu>4</vcpu>
    <os>
        <type arch = 'x86_64' machine = 'pc'>hvm</type>
        <boot dev = 'hd'/>
    </os>
    <features>
        <acpi/>
        <apic/>
        <pae/>
    </features>
    <clock offset = 'utc'/>
    <on_poweroff>destroy</on_poweroff>
    <on_reboot>restart</on_reboot>
    <on_crash>destroy</on_crash>
    <devices>
        <emulator>/usr/libexec/qemu-kvm</emulator>
        <disk type = 'file' device = 'disk'>
            <driver name = 'qemu' type = 'qcow2' cache = 'none'/>
            <source file = '/tmp/test.qcow2'/>
            <target dev = 'hda' bus = 'ide'/>
        </disk>
        <memballoon model='none'/>
        <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0' keymap='en-us'>
            <listen type='address' address='0.0.0.0'/>
        </graphics>
    </devices>

    <launchSecurity type='sev'>
        <policy>0x0001</policy>
        <cbitpos>47</cbitpos>
        <reducedPhysBits>5</reducedPhysBits>
    </launchSecurity>

    <qemu:commandline>
        <qemu:arg value="-drive"/>
        <qemu:arg value="if=pflash,format=raw,unit=0,file=/usr/share/edk2/ovmf/OVMF_CODE.cc.fd,readonly=on"/>
    </qemu:commandline>
</domain>
```

#### 启动CSV虚拟机

```sh
sudo virsh create csv_launch.xml
```

## 步骤六：检查guest的CSV使能状态

- 请使用vnc或其他远程工具连接guest
- anolis镜像默认用户名anuser，密码anolisos

```sh
localhost login: anuser
Password: anolisos
```

登录虚拟机后，执行：

```sh
dmesg | grep -i sev
```

显示内容应类似如下，则证明CSV虚拟机启动成功：

```sh
[    0.129692] AMD Secure Encrypted Virtualization (SEV) active
[    1.886794] software IO TLB: SEV is active and system is using DMA bounce buffers
```
