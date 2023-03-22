# Apache_Teaclave_Java_TEE_SDK最佳实践

## 1. 背景信息

Intel SGX技术提供了极高安全等级的可信执行环境，但使用该技术需要对已有的应用代码进行改造，SGX SDK只提供了对C语言生态的支持，此外用户需要用.edl文件定义服务接口，开发过程繁琐，对开发者的编程习惯冲击较大，造成开发门槛很高，阻碍了该技术的发展与应用。

Apache Teaclave Java TEE SDK提供基于Intel SGX技术的Java生态机密计算开发框架。采用Java静态编译技术将Enclave Module编译成Native代码并运行在SGX TEE中，实现对高级语言的支持并最大限度保持较低的系统TCB。屏蔽底层交互细节，用户无须定义edl接口文件。给用户提供一个Pure Java的机密计算开发框架和编译构建工具链，极大降低Intel SGX的开发门槛。

## 2. 环境准备

构建SGX机密计算环境前，您可以通过cpuid检查SGX使能状态。

1、安装cpuid。

```sh
sudo yum install -y cpuid
```

2、检查SGX使能状态。

```sh
cpuid -1 -l 0x7 | grep SGX
```

**说明**  SGX被正确使能后，运行SGX程序还需要SGX驱动。

3、检查SGX驱动安装情况。

```sh
ls -l /dev/{sgx_enclave,sgx_provision}
```


## 3. 运行项目test/benchmark/sample

### 3.1 进入容器环境

目前Teaclave-Java-Tee-SDK支持两种容器环境ubuntu18.04和anolis8.6.  


`docker run -it --privileged --network host -v /dev/sgx_enclave:/dev/sgx/enclave -v /dev/sgx_provision:/dev/sgx/provision teaclave/teaclave-java-tee-sdk:v0.1.0-ubuntu18.04 /bin/bash`

或

`docker run -it --privileged --network host -v /dev/sgx_enclave:/dev/sgx/enclave -v /dev/sgx_provision:/dev/sgx/provision teaclave/teaclave-java-tee-sdk:v0.1.0-anolis8.6 /bin/bash`

### 3.2 运行samples

`cd /opt/javaenclave/samples`  


运行 HelloWorld Sample: `cd helloworld && ./run.sh`  

结果如下即代表运行成功：
```
   Downloading ...........
   
[INFO]
[INFO] helloworld ......................................... SUCCESS [  0.758 s]
[INFO] common ............................................. SUCCESS [  5.766 s]
[INFO] enclave ............................................ SUCCESS [ 55.070 s]
[INFO] host ............................................... SUCCESS [ 25.544 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:27 min
[INFO] Finished at: 2023-02-16T12:27:18Z
[INFO] ------------------------------------------------------------------------
Hello World
Hello World
Hello World
Hello World
```

运行 Springboot Sample: `cd springboot && ./run.sh`  

结果如下即代表运行成功：
```
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for springboot 1.0-SNAPSHOT:
[INFO]
[INFO] springboot ......................................... SUCCESS [ 22.343 s]
[INFO] common ............................................. SUCCESS [06:46 min]
[INFO] enclave ............................................ SUCCESS [02:52 min]
[INFO] host ............................................... SUCCESS [02:58 min]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  14:38 min
[INFO] Finished at: 2023-02-16T11:28:49Z
[INFO] ------------------------------------------------------------------------

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.0)


```




==若因网络原因无法下载依赖，可通过配置Maven镜像源解决==

### 3.3 运行tests

`cd /opt/javaenclave/test && ./run.sh`



```
...........
.enter test case: org.apache.teaclave.javasdk.test.host.TestSMEnclave
exit test case: org.apache.teaclave.javasdk.test.host.TestSMEnclave
.enter test case: org.apache.teaclave.javasdk.test.host.TestSMEnclave
exit test case: org.apache.teaclave.javasdk.test.host.TestSMEnclave
.enter test case: org.apache.teaclave.javasdk.test.host.TestSMEnclave
exit test case: org.apache.teaclave.javasdk.test.host.TestSMEnclave

Time: 109.616

OK (16 tests)

Teaclave java sdk ut result: true

```

### 3.4 运行benchmark

`cd /opt/javaenclave/benchmark`

运行 Guomi Benchmark: `cd guomi && ./run.sh`

```

REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.

Benchmark                   (enclaveServiceInstance)  (smAlgo)  Mode  Cnt   Score    Error  Units
GuoMiBenchMark.smBenchMark               MOCK_IN_JVM       SM2  avgt    4  39.032 ? 39.471  ms/op
GuoMiBenchMark.smBenchMark               MOCK_IN_JVM       SM3  avgt    4   8.656 ?  0.302  ms/op
GuoMiBenchMark.smBenchMark               MOCK_IN_JVM       SM4  avgt    4   3.410 ?  0.153  ms/op
GuoMiBenchMark.smBenchMark               MOCK_IN_SVM       SM2  avgt    4  31.899 ?  1.003  ms/op
GuoMiBenchMark.smBenchMark               MOCK_IN_SVM       SM3  avgt    4  10.755 ?  2.616  ms/op
GuoMiBenchMark.smBenchMark               MOCK_IN_SVM       SM4  avgt    4   4.515 ?  0.303  ms/op
GuoMiBenchMark.smBenchMark                   TEE_SDK       SM2  avgt    4  36.113 ?  2.337  ms/op
GuoMiBenchMark.smBenchMark                   TEE_SDK       SM3  avgt    4  11.331 ?  1.379  ms/op
GuoMiBenchMark.smBenchMark                   TEE_SDK       SM4  avgt    4  10.292 ?  0.896  ms/op
GuoMiBenchMark.smBenchMark           EMBEDDED_LIB_OS       SM2  avgt    4  32.058 ? 14.033  ms/op
GuoMiBenchMark.smBenchMark           EMBEDDED_LIB_OS       SM3  avgt    4  10.380 ?  0.741  ms/op
GuoMiBenchMark.smBenchMark           EMBEDDED_LIB_OS       SM4  avgt    4   9.190 ?  0.488  ms/op

Benchmark result is saved to guomi_benchmark.json

```



运行 String Benchmark: `cd string && ./run.sh`

```
REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.

Benchmark                        (enclaveServiceInstance)  (stringOpt)  Mode  Cnt  Score   Error  Units
StringBenchMark.stringBenchMark               MOCK_IN_JVM        regex  avgt    4  2.788 ? 0.090  ms/op
StringBenchMark.stringBenchMark               MOCK_IN_JVM       concat  avgt    4  2.692 ? 0.139  ms/op
StringBenchMark.stringBenchMark               MOCK_IN_JVM        split  avgt    4  2.698 ? 0.129  ms/op
StringBenchMark.stringBenchMark               MOCK_IN_SVM        regex  avgt    4  4.558 ? 0.084  ms/op
StringBenchMark.stringBenchMark               MOCK_IN_SVM       concat  avgt    4  4.879 ? 0.200  ms/op
StringBenchMark.stringBenchMark               MOCK_IN_SVM        split  avgt    4  5.595 ? 0.186  ms/op
StringBenchMark.stringBenchMark                   TEE_SDK        regex  avgt    4  5.377 ? 0.176  ms/op
StringBenchMark.stringBenchMark                   TEE_SDK       concat  avgt    4  5.269 ? 0.119  ms/op
StringBenchMark.stringBenchMark                   TEE_SDK        split  avgt    4  6.171 ? 0.403  ms/op
StringBenchMark.stringBenchMark           EMBEDDED_LIB_OS        regex  avgt    4  4.104 ? 0.992  ms/op
StringBenchMark.stringBenchMark           EMBEDDED_LIB_OS       concat  avgt    4  6.140 ? 0.278  ms/op
StringBenchMark.stringBenchMark           EMBEDDED_LIB_OS        split  avgt    4  4.305 ? 0.585  ms/op

Benchmark result is saved to string_benchmark.json
```

## 4. HelloWorld Demo演示

### 4.1 进入容器环境

与《运行项目test/benchmark/sample》的进入容器环境步骤相同。


### 4.2 创建JavaEnclave工程框架

利用Teaclave-Java-Tee-SDK提供的脚手架创建JavaEnclave工程框架:  


`mvn archetype:generate -DgroupId=com.sample -DartifactId=helloworld -DarchetypeGroupId=org.apache.teaclave.javasdk -DarchetypeArtifactId=javaenclave-archetype -DarchetypeVersion=0.1.0 -DinteractiveMode=false`

该工程包括三个子工程, 分别是host、common和enclave.

### 4.3 定义服务接口(common)

在common子模块中定义服务接口:

`cd helloworld/common/src/main/java/com/sample/`

`mkdir -p helloworld/common`

创建Service.java, 定义服务接口:
```
package com.sample.helloworld.common;

import org.apache.teaclave.javasdk.common.annotations.EnclaveService;

@EnclaveService
public interface Service {
    String sayHelloWorld();
}
```

### 4.4 服务接口实现(enclave)

在enclave子模块实现所定义的服务接口:  


`cd helloworld/enclave/src/main/java/com/sample/`

`mkdir -p helloworld/enclave`

创建ServiceImpl.java, 实现服务接口:  

```
package com.sample.helloworld.enclave;

import com.sample.helloworld.common.Service;
import com.google.auto.service.AutoService;

@AutoService(Service.class)
public class ServiceImpl implements Service {
    @Override
    public String sayHelloWorld() {
        return "Hello World";
    }
}
```

### 4.5 管理服务实现(host)

在host子模块实现对enclave的管理与服务加载.  


`cd helloworld/host/src/main/java/com/sample/`

`mkdir -p helloworld/host`

创建Main.java, 实现机密计算服务管理:

```
package com.sample.helloworld.host;

import org.apache.teaclave.javasdk.host.Enclave;
import org.apache.teaclave.javasdk.host.EnclaveFactory;
import org.apache.teaclave.javasdk.host.EnclaveType;

import com.sample.helloworld.common.Service;

import java.util.Iterator;

public class Main {
    public static void main(String[] args) throws Exception {
        EnclaveType[] enclaveTypes = {
                EnclaveType.MOCK_IN_JVM,
                EnclaveType.MOCK_IN_SVM,
                EnclaveType.TEE_SDK};

        for (EnclaveType enclaveType : enclaveTypes) {
            Enclave enclave = EnclaveFactory.create(enclaveType);
            Iterator<Service> services = enclave.load(Service.class);
            System.out.println(services.next().sayHelloWorld());
            enclave.destroy();
        }
    }
}
```

### 4.6 编译和运行

回到工程根目录并编译工程: `mvn -Pnative clean package`

编译成功后, 运行该Demo: `$JAVA_HOME/bin/java -cp host/target/host-1.0-SNAPSHOT-jar-with-dependencies.jar:enclave/target/enclave-1.0-SNAPSHOT-jar-with-dependencies.jar com.sample.helloworld.host.Main`



```


[INFO] Building jar: /opt/temp/helloworld/host/target/host-1.0-SNAPSHOT-jar-with-dependencies.jar
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for helloworld 1.0-SNAPSHOT:
[INFO]
[INFO] helloworld ......................................... SUCCESS [  0.111 s]
[INFO] common ............................................. SUCCESS [ 14.584 s]
[INFO] enclave ............................................ SUCCESS [01:29 min]
[INFO] host ............................................... SUCCESS [ 51.027 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  02:35 min
[INFO] Finished at: 2023-02-16T12:01:09Z
[INFO] ------------------------------------------------------------------------
# $JAVA_HOME/bin/java -cp host/target/host-1.0-SNAPSHOT-jar-with-dependencies.jar:enclave/target/enclave-1.0-SNAPSHOT-jar-with-dependencies.jar com.sample.helloworld.host.Main
Hello World
Hello World
Hello World
```