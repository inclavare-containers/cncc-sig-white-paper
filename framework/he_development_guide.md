# Intel_HE_Toolkit开发指南

本文首先介绍了如何在Anolis 8.6上分别以源码和二进制包形式编译与安装Intel HE Toolkit，以及如何运行HE相关例程。

## 源码安装
### 源码下载

```sh
sudo yum install -y git
git clone -b v2.0.1 https://github.com/intel/he-toolkit.git
```

### 安装依赖
* 安装系统依赖

```sh
sudo yum install -y m4 \
                    patchelf \
                    cmake \
                    gcc-toolset-10 \
                    glibc-devel \
                    virtualenv \
                    autoconf \
                    wget \
                    bzip2 \
                    python38
```
* 安装python依赖
```sh
cd he-toolkit
sudo pip3.8 install -r requirements.txt
sudo pip3.8 install -r dev_reqs.txt
```
* 安装gmp-6.2.1
```sh
wget https://gmplib.org/download/gmp/gmp-6.2.1.tar.xz
tar -xf gmp-6.2.1.tar.xz
cd gmp-6.2.1
./configure
make
sudo make install
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```
### 编译安装
* 初始化编译环境
```
sudo update-alternatives --config python3 # 选择python3.8选项
scl enable gcc-toolset-10 bash
cd he-toolkit
./hekit init --default-config
#根据命令提示 source bash_profile
```
* 编译和安装第三方依赖
```sh
./hekit install recipes/default.toml
```
>如果遇到找不到某个依赖包的问题，可以通过修改`default.toml`中相应的安装路径来解决。
比如：如果提示找不到zstd，需要修改default.toml里面的
`export_cmake = "install/lib/cmake/zstd"`
改成
`export_cmake = "install/lib64/cmake/zstd"`

* 查看编译安装的包

```sh
./hekit list
```

### 运行示例应用

#### 1. 逻辑回归  

这是一个用SEAL CKKS实现的逻辑回归推理，可以根据实际需要选择数据和模型都是密文或者其中一方是密文而另一方是明文的配置。通过传递命令行参数`--linear_regression`，也可以实现线性回归推理。
* 编译
```
cd he-toolkit/he-samples/examples/logistic-regression
cmake -S . -B build -DSEAL_DIR=$HOME/.hekit/components/seal/v3.7.2/install/lib64/cmake/SEAL-3.7 \
               -DMicrosoft.GSL_DIR=$HOME/.hekit/components/gsl/v3.1.0/install/share/cmake/Microsoft.GSL \
               -Dzstd_DIR=$HOME/.hekit/components/zstd/v1.4.5/install/lib64/cmake/zstd \
               -DHEXL_DIR=$HOME/.hekit/components/hexl/1.2.3/install/lib/cmake/hexl-1.2.3
cmake --build build
```
* 运行
```
cd build
./lr_test
```
* 预期输出结果：
>INFO: Wed Nov  2 03:21:03 2022: Loading EVAL dataset  
INFO: Wed Nov  2 03:21:03 2022:   dataLoader: datasets/lrtest_mid_eval.csv  
INFO: Wed Nov  2 03:21:04 2022: Loading EVAL dataset complete Elapsed(s): 0.102  
INFO: Wed Nov  2 03:21:04 2022: Input data size: (samples) 2000  (features) 80  
INFO: Wed Nov  2 03:21:04 2022: Loading Model  
INFO: Wed Nov  2 03:21:04 2022:   weightsLoader: datasets/lrtest_mid_lrmodel.csv  
INFO: Wed Nov  2 03:21:04 2022: Loading Model complete Elapsed(s): 0  
INFO: Wed Nov  2 03:21:04 2022: Encode/encrypt weights and bias  
INFO: Wed Nov  2 03:21:04 2022: Encode/encrypt weights and bias complete Elapsed(s): 0.041  
INFO: Wed Nov  2 03:21:04 2022: HE LR  
INFO: Wed Nov  2 03:21:04 2022:   # of batches: 1  Batch size: 4096  
INFO: Wed Nov  2 03:21:04 2022:   Transpose data  
INFO: Wed Nov  2 03:21:04 2022:   - Transpose data complete Elapsed(s): 0.005  
INFO: Wed Nov  2 03:21:04 2022:   Encode/encrypt data  
INFO: Wed Nov  2 03:21:04 2022:   - Encode/encrypt data complete! Elapsed(s): 0.026  
INFO: Wed Nov  2 03:21:04 2022:   Logistic Regression HE: 1 batch(es)  
INFO: Wed Nov  2 03:21:04 2022:   - LR HE complete! Elapsed(s): 0.085  
INFO: Wed Nov  2 03:21:04 2022:   Decrypt/decoding LRHE result  
INFO: Wed Nov  2 03:21:04 2022:   - Decrypt/decode complete! Elapsed(s): 0.003  
INFO: Wed Nov  2 03:21:04 2022: HE inference result - accuracy:  0.855  

#### 2. 安全查询  

&emsp;&emsp;这个例子演示了如何用同态加密技术（SEAL BFV scheme)实现安全数据库查询。在这个例子中，数据查询条件和数据库本身都是加密的状态。查询客户端负责初始化加密上下文、产生同态加密密钥对、加密查询条件和解密查询结果。查询服务端负责存储加密数据库，并实现加密数据库查询算法。
* 编译
```
cd he-toolkit/he-samples/examples/secure-query
cmake -S . -B build -DSEAL_DIR=$HOME/.hekit/components/seal/v3.7.2/install/lib64/cmake/SEAL-3.7 \
               -DMicrosoft.GSL_DIR=$HOME/.hekit/components/gsl/v3.1.0/install/share/cmake/Microsoft.GSL \
               -Dzstd_DIR=$HOME/.hekit/components/zstd/v1.4.5/install/lib64/cmake/zstd \
               -DHEXL_DIR=$HOME/.hekit/components/hexl/1.2.3/install/lib/cmake/hexl-1.2.3
cmake --build build
```
* 运行
```
cd build
./secure-query
```
* 预期输出结果
> Initialize SEAL BFV scheme with default parameters[Y(default)|N]:  
SEAL BFV context initialized with following parameters  
Polymodulus degree: 8192  
Plain modulus: 17  
Key length: 8  
Input file to use for database or press enter to use default[us_state_capitals.csv]:  
Number of database entries: 50  
Encrypting database entries into Ciphertexts  
Input key value to use for database query:Oregon  
Querying database for key: Oregon  
Decoded database entry: Salem  
>
>Total query elapsed time(seconds): (Time in seconds for database query)  
Records searched per second: (number of records searched per second)  

## 二进制包安装
除了直接通过源代码编译安装，Intel HE Toolkit即将在下一个Anolis版本中支持通过二进制包的形式安装。

## 参考
Intel HE Toolkit: https://github.com/intel/he-toolkit
