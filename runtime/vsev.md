# AMD SEV机密虚拟机

## 背景信息

AMD SEV 技术基于AMD EPYC CPU，将物理机密计算能力传导至虚拟机实例，在公有云上打造一个立体化可信加密环境。 
SEV可保证单个虚拟机实例使用独立的硬件密钥对内存加密，同时提供高性能支持。密钥由 AMD 平台安全处理器 (PSP)在实例创建期间生成，而且仅位于处理器中，云厂商无法访问这些密钥。

## 测试环境

### 硬件配置

CPU: AMD EPYC 7763   *1

Memory: DDR4 3200 32G   *16

### 软件信息

OS: Anolis 8.5.0-10.0.1

Kernel: 5.10.134-12.1.an8.x86_64

## 第一步 开启 SME和 SEV 

### 开启 SME

将 mem_encrypt=on 添加到kernel的引导参数

### Enable SEV

将 kvm_amd.sev=1 kvm.添加到kernel的引导参数中

### 确保Kernel的引导参数生效

1. vim /etc/default/grub
2. 增加 "kvm_amd.sev=1 mem_encrypt=on" 到 "GRUB_CMDLINE_LINUX_DEFAULT=" 这一行
3. grub2-mkconfig -o /boot/efi/EFI/anolis/grub.cfg

## 第二步 重启服务器进入BIOS开启SEV相关选项

### BIOS 配置项如下

1. Advanced->AMD CBS->CPU Common Options->SMEE->Enable
2. Advanced->AMD CBS->NBIO Common Options->IOMMU->Enabled
3. Advanced->AMD CBS->NBIO Common Options->SEV-SNP Support->Enable

### Anolis OS 对SEV的配置进行确认

1. cat /proc/cmdline

确保输出有"mem_encrypt=on kvm_amd.sev=1"

类似输出如下
```
BOOT_IMAGE=(hd1,gpt2)/vmlinuz-5.10.134-12.1.an8.x86_64 root=/dev/mapper/ao-root ro crashkernel=auto resume=/dev/mapper/ao-swap rd.lvm.lv=ao/root rd.lvm.lv=ao/swap rhgb quiet mem_encrypt=on kvm_amd.sev=1 kvm_amd.sev_es=1
```

2. dmesg | grep -i SEV

确保输出有 "SEV supported"

类似输出如下
```
[    5.145496] ccp 0000:47:00.1: sev enabled
[    5.234221] ccp 0000:47:00.1: SEV API:1.49 build:6
[    5.445958] SEV supported: 253 ASIDs
```

3. cat /sys/module/kvm_amd/parameters/sev

确保输出值是 "1" 或者 "Y"

## 第三步 启动虚拟机相关的准备工作

1. yum update && yum upgrade
2. yum install libvirt-daemon virt-manager libvirt-client qemu-kvm epel-release cloud-utils virt-install
3. virsh domcapabilities

确保输出有 "sev supported='yes'"

类似输出如下
```
   <sev supported='yes'>
      <cbitpos>51</cbitpos>
      <reducedPhysBits>1</reducedPhysBits>
      <maxGuests>253</maxGuests>
      <maxESGuests>0</maxESGuests>
    </sev>
```

### VM 网络环境配置

1. 创建默认的网络环境配置 default.xml

```
<network>
  <name>default</name>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:0a:cd:21'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

2. virsh net-define --file default.xml
3. virsh net-start default
4. virsh net-autostart --network default

## 第四步  VM 镜像的配置

### ubuntu 作为VM

1. wget <https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img>
2. qemu-img convert focal-server-cloudimg-amd64.img /var/lib/libvirt/images/sev-guest.img
3. 创建VM镜像用户和密码配置文件 cloud-config

```
#cloud-config
ssh_pwauth: True
password: 123456
chpasswd: { expire: False }

chpasswd:
  list: |
     root:123456
     ubuntu:123456
  expire: False 
```

4. cloud-localds /var/lib/libvirt/images/init-passwd.iso cloud-config

## 第五步 启动VM 

### virsh 安装的方式
```
virt-install \
              --name sev-guest \
              --memory 4096 \
              --memtune hard_limit=4563402 \
              --boot uefi \
              --disk /var/lib/libvirt/images/sev-guest.img,device=disk,bus=scsi \
              --disk /var/lib/libvirt/images/init-passwd.iso,device=cdrom \
              --os-type linux \
              --os-variant centos8 \
              --import \
              --controller type=scsi,model=virtio-scsi,driver.iommu=on \
              --controller type=virtio-serial,driver.iommu=on \
              --network network=default,model=virtio,driver.iommu=on \
              --memballoon driver.iommu=on \
              --graphics none \
              --launchSecurity sev
```
### virsh 用xml文件 启动

- 创建sev.xml 文件,内容如下

```
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
        <cbitpos>51</cbitpos>
        <reducedPhysBits>1</reducedPhysBits>
    </launchSecurity>

    <qemu:commandline>
        <qemu:arg value="-drive"/>
        <qemu:arg value="if=pflash,format=raw,unit=0,file=/usr/share/edk2/ovmf/OVMF_CODE.cc.fd,readonly=on"/>
    </qemu:commandline>
</domain>
```

- 导入sev-guest虚拟机

virsh define sev.xml

- 开启虚拟机

virsh start sev-guest


## Step 6 检查SEV 在虚拟机中是否开启


1. virsh console sev-guest
2. dmesg | grep SEV
```
[    0.374549] AMD Memory Encryption Features active: SEV
```
