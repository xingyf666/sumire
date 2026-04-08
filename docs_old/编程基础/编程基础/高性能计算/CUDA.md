# CUDA

## 基本介绍

### 安装流程

首先 cmd 命令行检查 CUDA 所需的版本

![](image-20240209171050158.png)

在 CUDA 官网下载安装**不高于驱动版本**的安装包

![](4d856ec7737448e8b5552135e8f2a700.png)

直接执行安装程序即可完成安装。最后**通过 win + r 打开管理员模式的 cmd 窗口**，检查 CUDA 版本，确认安装成功

![](image-20240209171222336.png)



### 环境配置

下载 cuda-samples 库，存放到指定的路径下，然后添加两个环境变量

![](image-20240307135755710.png)



### nvidia-smi

nvidia-smi 是伴随 CUDA 一起下载的 GPU 管理工具。命令行输入 nvidia-smi 即可查看 GPU 参数

![](Screenshot_20240209_230717.jpg)

其中前两行框中的参数名和参数值一一对应。例如 GPU 对应下方的 0，表示第一块显卡。N/A 表示没有这一功能。

> 小知识：通常 GPU 使用率高时，显存使用率也会很高；但显存使用率高时，GPU 使用率可以很低。因为显存使用率只表示 CPU 与 GPU 之间交换数据很频繁，但此时 GPU 可能没有进行计算操作，因此其使用率可以很低。



通过命令行查看详细信息

```sh
$ nvidia-smi -q						# 查询所有 GPU 详细信息
$ nvidia-smi -q -i 0				# 查询特定 GPU 详细信息，其中 0 是 GPU 对应的 id
$ nvidia-smi -q -i 0 -d MEMORY		# 查询特定 GPU 的特定信息，例如 MEMORY
$ nvidia-smi -h						# 帮助命令
```



### GPU 硬件资源

GPU 包括更多的运算核心，适合数据并行的计算密集型任务；CPU 的运算核心较少，但是可以实现复杂的逻辑运算，适合控制密集型任务。CPU 上的线程是重量级的，上下文切换开销大；GPU 由于存在很多核心，其线程是轻量级的。

![](image-20240215012343555.png|800)



GPU 并行依靠流多处理器 SM (streaming multiprocessor) 来完成。

![](image-20240212174956583.png)

一个 GPU 由多个 SM 构成，每个 SM 都可以支持数百个线程**并发**执行。以线程块 block 为单位，向 SM 分配线程块，多个线程块可被同时分配到一个可用的 SM 上。当一个线程块被分配好 SM 后，就不能再分配到其它 SM 上。

![](Screenshot_20240212_175536.jpg|500)



下图分别是线程模型与物理结构。线程模型可以定义成千上万个线程，网格中的所有线程块需要分配到 SM 上执行，线程块内的所有线程分配到同一个 SM 中执行，但是每个 SM 上可以分配多个线程块。

![](Screenshot_20240212_175735.jpg|500)

线程块分配到 SM 上后，会以 32 个线程为一组进行分割，每个组成为一个 warp 。**同一线程块中相邻的 32 个线程构成一个线程束，在 GPU 硬件上真正做到了并行**。

![](Screenshot_20240212_180104.jpg|500)



### 自定义函数

#### 设备函数

定义只能执行在 GPU 设备上的函数称为设备函数，它只能被核函数或者其它设备函数调用。

```cpp
__device__ ret_type func(argument arg);
```



#### 核函数

用 `__global__` 修饰的函数称为核函数，一般由主机调用，在设备中执行。

```cpp
__global__ void func(argument arg);
```

核函数不能通过 `__host__` 或 `__device__` 修饰。



可以将 GPU 看作是 CPU 的外设，CUDA 程序不仅需要编写主机 CPU 代码，还需要编写外设 GPU 代码。主机对设备的调用通过核函数进行，**核函数在 GPU 上并行执行，不需要专门编写并行代码**。核函数的形式为

```cpp
__global__ void kernel_func(argument arg)
{
    printf("Hello World from the GPU!\n");
}

void __global__ kernel_func(argument arg)
{
    printf("Hello World from the GPU!\n");
}
```

返回值必须是 void，且需要 `__global__` 修饰。并且核函数

* 只能访问 GPU 内存（CPU 与 GPU 不能直接互相访问内存）
* 不能使用变长参数
* 不能使用静态变量
* 不能使用函数指针
* 具有异步性：GPU 上的核函数执行时，CPU 无法进行控制，因此 CPU 不会等待核函数执行完毕
* 不支持 iostream 方法



#### 主机函数

主机端的普通 C++ 函数可由 `__host__` 修饰，但通常忽略这一修饰。

```cpp
__host__ ret_type func(argument arg);
```

可以同时用 `__host__` 和 `__device__` 修饰一个函数来减少代码冗余，不需要重复编写相同的函数。



### 简单示例

首先给出一段示范代码

```cpp
#include <stdio.h>

__global__ void hello_from_gpu()
{
    printf("Hello World from the the GPU\n");
}

int main(void)
{
    // 4 个线程块，每个线程块有 4 个线程
    hello_from_gpu<<<4, 4>>>();
    
    // 同步主机与设备（等待 GPU 执行完成），促进缓冲区刷新
    cudaDeviceSynchronize();

    return 0;
}
```

可以直接通过命令行编译

```sh
$ nvcc hello.cu -o hello
```

注意需要将 `D:\Visual Studio\2022\Community\VC\Tools\MSVC\14.37.32822\bin\Hostx64\x64` 添加到环境变量。



给出 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.23)
project(hello LANGUAGES CXX CUDA)
add_executable(hello hello.cu)
```

命令行编译运行

```sh
$ cmake -B build
$ cmake --build build
$ ./build/Debug/hello.exe 
```

或者 build 生成 vs 项目，通过 vs 编译运行。



### 编译与兼容

#### 编译流程

nvcc 的编译流程为

1. 分离源代码为主机代码和设备代码
2. 主机 Host 代码是 C/C++ 语法，设备 Device 代码是 C/C++ 扩展语言
3. nvcc 先将设备代码编译为 PTX (Parallel Thread Execution) 伪汇编代码，再将 PTX 编译为二进制目标代码
4. 编译为 PTX 时，需要选项 `-arch=compute_XY` 指定一个虚拟架构的计算能力，用以确定代码中能够使用的 CUDA 功能
5. 编译 PTX 为二进制代码时，需要选项 `-code=sm_ZW` 指定一个真实架构的计算能力，用以确定可执行文件能够使用的 GPU

注意 GPU 的真实架构计算能力一定要大于虚拟架构的计算能力。

![](Screenshot_20240210_223637(1).jpg)

并非 GPU 的计算能力越高，性能就越高

![](Screenshot_20240210_223902.jpg)



#### 虚拟架构计算能力

编译为 PTX 时，需要选项 `-arch=compute_XY` 指定一个虚拟架构的计算能力，用以确定代码中能够使用的 CUDA 功能。这一转化步骤与 GPU 硬件无关。编译指令 `compute_XY` 中 X 表示计算能力的主版本号，Y 表示计算能力的次版本号。PTX 的指令只能在更高的计算能力的 GPU 下使用，即 XY 指定的版本不高于 GPU 的计算能力。例如

```sh
$ nvcc hello.cu -o hello -arch=compute_61
$ nvcc hello.cu -o hello -arch=compute_70
$ nvcc hello.cu -o hello -arch=compute_80
```

指定的版本都不高于 GPU 的计算能力，此时可以正常运行。但是

```sh
$ nvcc hello.cu -o hello -arch=compute_90
```

指定了过高的版本，因此核函数不会正常执行



#### 真实架构计算能力

PTX 指令转换为二进制代码与 GPU 架构有关。编译指令 `sm_XY` 与 `compute_XY` 格式相同。注意

* 二进制代码大版本之间不兼容
* 只要指定了真实架构计算能力，一定要指定虚拟架构计算能力
* 指定的真实架构计算能力必须大于虚拟架构计算能力

真实架构可以实现同一个大版本下，较低的次版本号与较高的次版本号之间的兼容。例如

```sh
$ nvcc hello.cu -o hello_sm_80 -arch=compute_80 -code=sm_80
```



#### 多个 GPU 版本编译

这样可以为多个 GPU 的版本进行编译。例如

```sh
$ nvcc hello.cu -o hello -gencode arch=compute_70,code=sm_70 -gencode arch=compute_80,code=sm_80
```

编译出的可执行文件包含 2 个二进制版本，生成的可执行文件称为胖二进制文件 fatbinary 。当然，指定多个计算能力，会增加编译时间和可执行文件的大小。



#### 默认计算能力

在没有指定计算能力时，会默认指定一个计算能力。要查看默认的计算能力，应该编译为 PTX 指令

```sh
$ nvcc hello.cu -ptx
```

这会生成一个 hello.ptx 文件，直接查看文件内容即可看到计算能力。



## 程序设计

### 计算精度

在大规模程序计算中，需要注意浮点数的计算精度问题。使用 float 将会对大规模计算造成极大的计算误差。



### 程序基本框架

CUDA 程序的基本流程为

```cpp
// 头文件

__global__ void func(argument arg)
{
    // 核函数
}

int main(void)
{
    // 设置 GPU 设备
    // 分配主机和设备内存
    // 初始化主机中的数据
    // 数据从主机复制到设备
    // 调用核函数在设备中进行计算
    // 将计算结果从设备返回主机
    // 释放主机与设备内存
}
```



### GPU 设备

在 GPU 运行时，可以通过运行时 API 函数获取当前设备状态。例如

```cpp
// 检测计算机 GPU 数量
int iDeviceCount = 0;
cudaError_t error = cudaGetDeviceCount(&iDeviceCount);
// 返回 cudaSuccess 即成功

// 设置执行
int iDev = 0;
error = cudaSetDevice(iDev);
```

运行时 API 声明

```cpp
__host__ __device__ cudaError_t cudaGetDeviceCount ( int* count );
__host__ cudaError_t cudaSetDevice ( int  device );
```

这里 `__host__` 修饰表示此 API 可以在主机调用，`__device__` 修饰则表示可以在设备调用。



我们可以给出一个示例程序

```cpp
#include <stdio.h>

int main(void)
{
    // 检测计算机 GPU 数量
    int iDeviceCount = 0;
    cudaError_t error = cudaGetDeviceCount(&iDeviceCount);
	
    // 如果获取失败或者没有 GPU 则报错
    if (error != cudaSuccess || iDeviceCount == 0)
    {
        printf("No CUDA campatable GPU found!\n");
        exit(-1);
    }
    else
    {
        printf("The count of GPUs is %d.\n", iDeviceCount);
    }
    
    // 设置执行，只有 1 个显卡
    int iDev = 0;
    error = cudaSetDevice(iDev);
    if (error != cudaSuccess)
    {
        printf("fail to set GPU 0 for computing.\n");
        exit(-1);
    }
    else
    {
        printf("set GPU 0 for computing.\n");
    }

    return 0;
}
```



### 线程模型

线程模型分为 grid 网格和 block 线程块两个部分。一个核函数启动的所有线程称为一个网格。如图 Host 为 CPU 主机，Device 为 GPU 设备。每个 Kernel 核函数启动一系列的线程，构成 Grid 网格。网格内部根据线程块 Block 划分，每个线程块内部有多个线程。

![](Screenshot_20240210_164106.jpg)

线程分块是逻辑上的划分，不是物理上的分块。通过 `<<<grid_size, block_size>>>` 配置线程

* 最大允许线程块大小：1024
* 最大允许网格大小：$2^{21}-1$（针对一维网格）

在实际使用中，只有当**总的线程数大于计算核心数时才能更充分地利用 GPU 中的计算资源，因为这样能够减少计算核心空闲的时间**。



#### 一维线程模型

每个线程在核函数中具有唯一的身份标识。这个标识由 `<<<grid_size, block_size>>>` 确定，这两个变量保存在内建变量 build-in variable 中，即它是预定义好的变量。**目前考虑一维情形**：

* `gridDim.x` 保存执行配置中变量 grid_size 的值；
* `blockDim.x` 保存执行配置中变量 block_size 的值；

线程索引也会保存成内建变量：

* `blockIdx.x` 指定线程在网格中的**线程块索引值**，范围为 0~`gridDim.x-1`；
* `threadIdx.x` 指定线程在线程块中的**线程索引值**，范围为 0~`blockDim.x-1`；

![](Screenshot_20240210_165759.jpg)

通过线程块索引和线程索引，可以得到线程的唯一标识

```cpp
#include <stdio.h>

__global__ void hello_from_gpu()
{
    const int bid = blockIdx.x;
    const int tid = threadIdx.x;

    const int id = threadIdx.x + blockIdx.x * blockDim.x; 
    printf("Hello World from block %d and thread %d, global id %d\n", bid, tid, id);
}

int main(void)
{
    hello_from_gpu<<<2, 4>>>();
    cudaDeviceSynchronize();

    return 0;
}
```



#### 多维线程

CUDA 可以组织三维的网格和线程块。

> 多维网格和线程块本质上还是一维的，GPU 在物理上不分块。

内建 uint3 类型的变量，这里 uint3 可以理解为无符号整型的结构。

```cpp
struct blockIdx
{
    uint x, y, z;
};

struct threadIdx
{
    uint x, y, z;
};
```

以及内建 dim3 类型的变量

```cpp
struct gridDim
{
    uint x, y, z;
};

struct blockDim
{
    uint x, y, z;
};
```

内建变量只在核函数中有效，无需定义。



使用 `<<<grid_size, block_size>>>` 确定网格和线程块维数时，**没有给出的维度默认为 1**，例如 `<<<2, 4>>>` 对应

```cpp
gridDim.x = 2;
gridDim.y = 1;	// 默认
gridDim.z = 1;	// 默认

blockDim.x = 4;
blockDim.y = 1;	// 默认
blockDim.z = 1;	// 默认
```



要定义多维网格和线程块，使用构造函数语法

```cpp
dim3 grid_size(Gx, Gy, Gz);
dim3 block_size(Bx, By, Bz);

// 定义 2x2x1 的网格和 5x3x1 的线程块
dim3 grid_size(2, 2);
dim3 block_size(5, 3, 1);
```

需要注意多维情形下，线程的标识是按照 x,y,z 的顺序依次变化，因此与矩阵索引恰好相反。



多维情形下的规模限制：

* `gridDim.x` 最大值为 $2^{31}-1$​；
* `gridDim.y` 最大值为 $2^{16}-1$；
* `gridDim.z` 最大值为 $2^{16}-1$​；
* `blockDim.x` 最大值为 1024；
* `blockDim.y` 最大值为 1024；
* `blockDim.z` 最大值为 64；

需要注意**线程块的总大小，即 x,y,z 相乘的值最大为 1024 ！**。



#### 全局索引计算

在调用核函数时，经常需要明确当前是哪一个线程在调用，这就需要计算线程全局索引。通常的计算流程为：

1. 计算当前线程所在的线程块的索引 a；
2. 计算当前线程在 a 线程块中的索引 b；
3. 计算当前线程标识 $a\times grid\_size + b$；

定义多维线程时，首先定义线程规模，然后调用核函数

```cpp
dim3 grid_size(1, 2, 3);
dim3 block_size(3, 4, 5);

kernel_func<<<grid_size, block_size>>>();
```



### 线程同步

为了确保数据读写的正确性，需要在适当的位置同步等待线程执行完毕。使用

```cpp
__host__ __device__ cudaError_t cudaDeviceSynchronize ( void );
```

阻塞主机，等待设备全部执行完毕。



在设备执行过程中，可以使用

```cpp
__syncthreads();
```

同步每个线程块内的所有线程。

> 需要注意多个线程块之间无法同步，因为这会导致资源浪费。



### 内存管理

CUDA 通过内存分配、数据传递、内存初始化、内存释放进行内存管理。

| STANDARD C FUNCTIONS | CUDA C FUNCTIONS |
| -------------------- | ---------------- |
| malloc               | cudaMalloc       |
| memcpy               | cudaMemcpy       |
| memset               | cudaMemset       |
| free                 | cudaFree         |



#### Host 内存

传输数据时，使用 malloc 申请主机内存。但是更建议使用

```cpp
__host__ cudaError_t cudaMallocHost ( void** ptr, size_t size );
__host__ cudaError_t cudaFreeHost ( void* ptr );
```

申请和释放速度要比普通 CPU 内存的申请和释放快一倍。



#### 统一 Unified 内存

Unified 内存可以同时被 CPU 与 GPU 访问

```cpp
__host__ cudaError_t cudaMallocManaged ( void** devPtr, size_t size, unsigned int  flags = cudaMemAttachGlobal );
```

其中 cudaMemAttachGlobal 表示同时被 CPU 与 GPU 访问。使用 `cudaFree` 释放内存。



#### 设备内存

设备内存分配函数声明

```cpp
__host__ __device__ cudaError_t cudaMalloc ( void** devPtr, size_t size );
```

此内存分配函数可以在主机调用。注意由于返回值是错误码，因此需要先指定一个指针变量，然后传入函数分配内存。



#### 数据拷贝

设备数据同步拷贝函数声明

```cpp
__host__ cudaError_t cudaMemcpy ( void* dst, const void* src, size_t count, cudaMemcpyKind kind );
```

注意这个函数在主机调用，最后一个参数指定拷贝方向

![](image-20240211181101815.png)

默认方式只能在支持统一虚拟寻址的系统上使用。



有同步拷贝就有异步拷贝

```cpp
__host__ __device__ cudaError_t cudaMemcpyAsync ( void* dst, const void* src, size_t count, cudaMemcpyKind kind, cudaStream_t stream = 0 );
```

这里的拷贝操作不会等待数据拷贝完成；如果 stream 不为零，可能会与其它 stream 操作重叠。



#### 内存初始化

设备内存初始化函数声明

```cpp
__host__ cudaError_t cudaMemset ( void* devPtr, int  value, size_t count );
```



#### 内存释放

设备内存释放函数声明

```cpp
__host__ __device__ cudaError_t cudaFree ( void* devPtr )
```



### 错误代码

CUDA 运行时 API 大多支持返回错误代码，返回值为枚举类型 cudaError_t 。如果成功执行，则返回值为 cudaSuccess 。



#### 检查设备函数

设备错误检查函数声明

```cpp
__host__ __device__ const char* cudaGetErrorName ( cudaError_t error );
__host__ __device__ const char* cudaGetErrorString ( cudaError_t error );
```

对错误检查进行封装

```cpp
cudaError_t ErrorCheck(cudaError_t error_code, const char *filename,
                       int lineNumber) {
  if (error_code != cudaSuccess) {
    printf("CUDA error:\r\ncode=%d, name=%s, description=%s\r\nfile=%s, "
           "line%d\r\n",
           error_code, cudaGetErrorName(error_code),
           cudaGetErrorString(error_code), filename, lineNumber);
    return error_code;
  }
  return error_code;
}
```

参数 filename 一般使用 `__FILE__`，参数 lineNumber 一般使用 `__LINE__` 。使用例如

```cpp
float *fpDevice_A;
cudaError_t error = ErrorCheck(cudaMalloc((float**)&fpDevice_A, 4), __FILE__, __LINE__);
```



#### 检查核函数

错误检测函数不能捕捉核函数的相关错误，因为核函数不返回任何错误代码。需要使用

```cpp
__host__ __device__ cudaError_t cudaGetLastError ( void );
__host__ __device__ cudaError_t cudaDeviceSynchronize ( void );
```

具体使用方法为

```cpp
ErrorCheck(cudaGetLastError(), __FILE__, __LINE__);
ErrorCheck(cudaDeviceSynchronize(), __FILE__, __LINE__);
```

这里 cudaGetLastError 将会捕捉同步函数之前的最后一个错误。因为 CPU 不会等待核函数执行完毕，因此需要同步，防止后面代码中的错误覆盖了核函数中的错误。



### 矩阵加法示例

我们可以封装两个通用的函数作为头文件 common.cuh

```cpp
#pragma once

#include <stdio.h>
#include <stdlib.h>

// 错误检查函数
cudaError_t ErrorCheck(cudaError_t error_code, const char *filename, int lineNumber)
{
    if (error_code != cudaSuccess)
    {
        printf("CUDA error:\r\ncode=%d, name=%s, description=%s\r\nfile=%s, "
               "line%d\r\n",
               error_code, cudaGetErrorName(error_code), cudaGetErrorString(error_code), filename, lineNumber);
        return error_code;
    }
    return error_code;
}

// 初始化 GPU 设置
void setGPU()
{
    // 检测计算机 GPU 数量
    int iDeviceCount = 0;
    cudaError_t error = ErrorCheck(cudaGetDeviceCount(&iDeviceCount), __FILE__, __LINE__);

    if (error != cudaSuccess || iDeviceCount == 0)
    {
        printf("No CUDA campatable GPU found!\n");
        exit(-1);
    }
    else
    {
        printf("The count of GPUs is %d.\n", iDeviceCount);
    }
    
    // 设置执行
    int iDev = 0;
    error = ErrorCheck(cudaSetDevice(iDev), __FILE__, __LINE__);
    if (error != cudaSuccess)
    {
        printf("fail to set GPU 0 for computing.\n");
        exit(-1);
    }
    else
    {
        printf("set GPU 0 for computing.\n");
    }
}
```



通常使用 CPU 的矩阵加法程序示例如下

```cpp
#include <stdio.h>

// 在 CPU 中循环计算加法
void addFromCPU(float *A, float *B, float *C, const int N)
{
    for (int i = 0; i < N; i++)
    {
        C[i] = A[i] + B[i];
    }
}

// 初始化矩阵元素
void initialData(float *addr, int elemCount)
{
    for (int i = 0; i < elemCount; i++)
    {
        addr[i] = (float)(rand() & 0xFF) / 10.f;
    }
    return;
}

int main(void)
{
    // 1、分配主机内存，并初始化
    int iElemCount = 512;                     			// 设置元素数量
    size_t stBytesCount = iElemCount * sizeof(float); 	// 字节数
    
    // 动态分配内存
    float *fpHost_A, *fpHost_B, *fpHost_C;
    fpHost_A = (float *)malloc(stBytesCount);
    fpHost_B = (float *)malloc(stBytesCount);
    fpHost_C = (float *)malloc(stBytesCount);
    if (fpHost_A != NULL && fpHost_B != NULL && fpHost_C != NULL)
    {
        memset(fpHost_A, 0, stBytesCount);  // 主机内存初始化为 0
        memset(fpHost_B, 0, stBytesCount);
        memset(fpHost_C, 0, stBytesCount);
    }
    else
    {
        printf("Fail to allocate host memory!\n");
        exit(-1);
    }
    

    // 2、初始化主机中数据
    srand(666); // 设置随机种子
    initialData(fpHost_A, iElemCount);
    initialData(fpHost_B, iElemCount);
    
    addFromCPU(fpHost_A, fpHost_B, fpHost_C, iElemCount);

    for (int i = 0; i < 10; i++)    // 打印
    {
        printf("idx=%2d\tmatrix_A:%.2f\tmatrix_B:%.2f\tresult=%.2f\n", i+1, fpHost_A[i], fpHost_B[i], fpHost_C[i]);
    }

    // 3、释放主机与设备内存
    free(fpHost_A);
    free(fpHost_B);
    free(fpHost_C);
    return 0;
}
```



使用 GPU 并行计算的程序示例如下

> 在调用核函数后，原本需要调用同步函数确保所有元素都计算完成再复制内存，但是 cudaMemcpy 自带有隐式的同步函数，所以不需要再进行同步操作。

```cpp
#include <stdio.h>
#include "common.cuh"

// 自定义设备函数 add，在核函数中调用
__device__ float add(const float x, const float y)
{
    return x + y;
}

// 每个线程对应矩阵的一个元素，只计算这个元素位置的加法
// 当然我们也可以指定超过矩阵元素个数的线程总数，这时就需要排除掉多余的线程
__global__ void addFromGPU(float *A, float *B, float *C, const int N)
{
    const int bid = blockIdx.x;
    const int tid = threadIdx.x;
    const int id = tid + bid * blockDim.x; 
	
    // 排除多出的线程
    if (id >= N) return;
    C[id] = add(A[id], B[id]);
}

// 初始化矩阵元素
void initialData(float *addr, int elemCount)
{
    for (int i = 0; i < elemCount; i++)
    {
        addr[i] = (float)(rand() & 0xFF) / 10.f;
    }
    return;
}

int main(void)
{
    // 1、设置 GPU 设备
    setGPU();

    // 2、分配主机内存和设备内存，并初始化
    int iElemCount = 512;                               // 设置元素数量
    size_t stBytesCount = iElemCount * sizeof(float);   // 字节数
    
    // （1）分配主机内存，并初始化
    float *fpHost_A, *fpHost_B, *fpHost_C;
    fpHost_A = (float *)malloc(stBytesCount);
    fpHost_B = (float *)malloc(stBytesCount);
    fpHost_C = (float *)malloc(stBytesCount);
    if (fpHost_A != NULL && fpHost_B != NULL && fpHost_C != NULL)
    {
        memset(fpHost_A, 0, stBytesCount);  // 主机内存初始化为0
        memset(fpHost_B, 0, stBytesCount);
        memset(fpHost_C, 0, stBytesCount);
    }
    else
    {
        printf("Fail to allocate host memory!\n");
        exit(-1);
    }

    // （2）分配设备内存，并初始化
    float *fpDevice_A, *fpDevice_B, *fpDevice_C;
    cudaMalloc((float**)&fpDevice_A, stBytesCount);
    cudaMalloc((float**)&fpDevice_B, stBytesCount);
    cudaMalloc((float**)&fpDevice_C, stBytesCount);
    if (fpDevice_A != NULL && fpDevice_B != NULL && fpDevice_C != NULL)
    {
        cudaMemset(fpDevice_A, 0, stBytesCount);  // 设备内存初始化为0
        cudaMemset(fpDevice_B, 0, stBytesCount);
        cudaMemset(fpDevice_C, 0, stBytesCount);
    }
    else
    {
        printf("fail to allocate memory\n");
        free(fpHost_A);
        free(fpHost_B);
        free(fpHost_C);
        exit(-1);
    }

    // 3、初始化主机中数据
    srand(666); // 设置随机种子
    initialData(fpHost_A, iElemCount);
    initialData(fpHost_B, iElemCount);
    
    // 4、数据从主机复制到设备
    cudaMemcpy(fpDevice_A, fpHost_A, stBytesCount, cudaMemcpyHostToDevice); 
    cudaMemcpy(fpDevice_B, fpHost_B, stBytesCount, cudaMemcpyHostToDevice); 
    cudaMemcpy(fpDevice_C, fpHost_C, stBytesCount, cudaMemcpyHostToDevice);

    // 5、调用核函数在设备中进行计算，分 16 块，每块 32 个线程
    dim3 block(32);
    dim3 grid(iElemCount / 32);

    addFromGPU<<<grid, block>>>(fpDevice_A, fpDevice_B, fpDevice_C, iElemCount);    // 调用核函数
    // cudaDeviceSynchronize();

    // 6、将计算得到的数据从设备传给主机
    cudaMemcpy(fpHost_C, fpDevice_C, stBytesCount, cudaMemcpyDeviceToHost);

    for (int i = 0; i < 10; i++)    // 打印 10 个元素
    {
        printf("idx=%2d\tmatrix_A:%.2f\tmatrix_B:%.2f\tresult=%.2f\n", i+1, fpHost_A[i], fpHost_B[i], fpHost_C[i]);
    }

    // 7、释放主机与设备内存
    free(fpHost_A);
    free(fpHost_B);
    free(fpHost_C);
    cudaFree(fpDevice_A);
    cudaFree(fpDevice_B);
    cudaFree(fpDevice_C);
	
    // 重置设备
    cudaDeviceReset();
    return 0;
}
```



### CUDA 计时

使用 CUDA 事件计时方式，可同时为主机代码和设备代码计时。事件队列函数 cudaEventQuery() 不能使用错误检测，因为它大概率会返回一个错误。

```cpp
// 保存总时间
float t_sum = 0;
for (int repeat = 0; repeat <= NUM_REPEATS; ++repeat)
{
    // 初始化事件 start 和 stop
    cudaEvent_t start, stop;
    ErrorCheck(cudaEventCreate(&start), __FILE__, __LINE__);
    ErrorCheck(cudaEventCreate(&stop), __FILE__, __LINE__);
    
    // 记录开始时间
    ErrorCheck(cudaEventRecord(start), __FILE__, __LINE__);
    cudaEventQuery(start);	// 此处不可用错误检测函数
    
    // 调用核函数，需要计时的代码
    
    // 记录结束时间，同步停止时间，计算时间差 elapsed_time
    ErrorCheck(cudaEventRecord(stop), __FILE__, __LINE__);
    ErrorCheck(cudaEventSynchronize(stop), __FILE__, __LINE__);
    float elapsed_time;
    ErrorCheck(cudaEventElapsedTime(&elapsed_time, start, stop), __FILE__, __LINE__);
	
    // 第一次调用核函数会花费更多时间，因此排除掉第一次执行的时间
    if (repeat > 0)
        t_sum += elapsed_time;
    
    // 销毁事件变量
    ErrorCheck(cudaEventDestroy(start), __FILE__, __LINE__);
    ErrorCheck(cudaEventDestroy(stop), __FILE__, __LINE__);
}

// 计算平均时间
const float t_ave = t_sum / NUM_REPEATS;
printf("Time = %g ms.\n", t_ave);
```



### 运行时 GPU 信息

使用此 API 函数获取 GPU 设备的详细信息，其中 device 就是 GPU 索引。

```cpp
cudaDeviceProp prop;
__host__ cudaError_t cudaGetDeviceProperties ( cudaDeviceProp* prop, int  device );
```

例如下面的程序输出 GPU 的各种信息

```cpp
#include "common.cuh"
#include <stdio.h>

int main(void)
{
    int device_id = 0;
    ErrorCheck(cudaSetDevice(device_id), __FILE__, __LINE__);

    cudaDeviceProp prop;
    ErrorCheck(cudaGetDeviceProperties(&prop, device_id), __FILE__, __LINE__);

    printf("Device id:                                 %d\n",
        device_id);
    printf("Device name:                               %s\n",
        prop.name);
    printf("Compute capability:                        %d.%d\n",
        prop.major, prop.minor);
    printf("Amount of global memory:                   %g GB\n",
        prop.totalGlobalMem / (1024.0 * 1024 * 1024));
    printf("Amount of constant memory:                 %g KB\n",
        prop.totalConstMem  / 1024.0);
    printf("Maximum grid size:                         %d %d %d\n",
        prop.maxGridSize[0], 
        prop.maxGridSize[1], prop.maxGridSize[2]);
    printf("Maximum block size:                        %d %d %d\n",
        prop.maxThreadsDim[0], prop.maxThreadsDim[1], 
        prop.maxThreadsDim[2]);
    printf("Number of SMs:                             %d\n",
        prop.multiProcessorCount);
    printf("Maximum amount of shared memory per block: %g KB\n",
        prop.sharedMemPerBlock / 1024.0);
    printf("Maximum amount of shared memory per SM:    %g KB\n",
        prop.sharedMemPerMultiprocessor / 1024.0);
    printf("Maximum number of registers per block:     %d K\n",
        prop.regsPerBlock / 1024);
    printf("Maximum number of registers per SM:        %d K\n",
        prop.regsPerMultiprocessor / 1024);
    printf("Maximum number of threads per block:       %d\n",
        prop.maxThreadsPerBlock);
    printf("Maximum number of threads per SM:          %d\n",
        prop.maxThreadsPerMultiProcessor);

    return 0;
}
```



### 核心数量

运行时一般不能查询 GPU 的核心数量，但是可以通过计算能力来计算出核心数量

```cpp
#include <stdio.h>
#include "common.cuh"

int getSPcores(cudaDeviceProp devProp)
{  
    int cores = 0;
    int mp = devProp.multiProcessorCount;
    switch (devProp.major){
     case 2: // Fermi
      if (devProp.minor == 1) cores = mp * 48;
      else cores = mp * 32;
      break;
     case 3: // Kepler
      cores = mp * 192;
      break;
     case 5: // Maxwell
      cores = mp * 128;
      break;
     case 6: // Pascal
      if ((devProp.minor == 1) || (devProp.minor == 2)) cores = mp * 128;
      else if (devProp.minor == 0) cores = mp * 64;
      else printf("Unknown device type\n");
      break;
     case 7: // Volta and Turing
      if ((devProp.minor == 0) || (devProp.minor == 5)) cores = mp * 64;
      else printf("Unknown device type\n");
      break;
     case 8: // Ampere
      if (devProp.minor == 0) cores = mp * 64;
      else if (devProp.minor == 6) cores = mp * 128;
      else if (devProp.minor == 9) cores = mp * 128; // ada lovelace
      else printf("Unknown device type\n");
      break;
     case 9: // Hopper
      if (devProp.minor == 0) cores = mp * 128;
      else printf("Unknown device type\n");
      break;
     default:
      printf("Unknown device type\n"); 
      break;
      }
    return cores;
}

int main()
{
    int device_id = 0;
    ErrorCheck(cudaSetDevice(device_id), __FILE__, __LINE__);

    cudaDeviceProp prop;
    ErrorCheck(cudaGetDeviceProperties(&prop, device_id), __FILE__, __LINE__);

    printf("Compute cores is %d.\n", getSPcores(prop));

    return 0;
}
```



## 内存资源

### 内存模型

内存结构层次特点是运行速度越来越慢，储存容量越来越大。CPU 和 GPU 主存采用 DRAM (动态随机存取存储器)，低延迟内存采用 SRAM (静态随机存取存储器) 。

![](Screenshot_20240212_180536.jpg)

CUDA 内存模型可以概览如下

![](Screenshot_20240212_181009.jpg)

CUDA 内存和它们的主要特征

![](Screenshot_20240212_181240.jpg)



#### 寄存器

寄存器内存在片上 (on-chip)，具有 GPU 上最快的访问速度，但是数量有限。寄存器仅在线程内可见，生命周期与所在线程相同。

* 核函数中定义的不加任何限定符的变量一般存在寄存器中；
* 内建变量存放在寄存器中；
* 核函数中定义的不加任何限定符的数组可能存在寄存器中，也可能存在本地内存中；

寄存器都是 32 位，因此 double 类型需要两个寄存器保存。寄存器保存在 SM 的寄存器文件

![](Screenshot_20240212_181945.jpg)



#### 本地内存

寄存器放不下的内存会放在本地内存：

1. 索引值不能在编译时确定的数组（即动态分配内存的数组）存放在本地内存；
2. 可能占用大量寄存器空间的较大本地结构体和数组；
3. 任何不满足核函数寄存器限定条件的变量；

对于计算能力 2.0 以上的设备，本地内存的数据存储在每个 SM 的一级缓存和设备的二级缓存中。

![](Screenshot_20240212_182300.jpg)



#### 寄存器溢出

如果核函数所需的寄存器数量超出硬件设备支持，则数据会保存在本地内存。有两种情况：

1. 一个 SM 并行运行多个线程块/线程束，所需的寄存器容量大于 64 KB
2. 单个线程运行所需寄存器数量多于 255 个

寄存器溢出会降低程序运行性能，因为本地内存从硬件角度看只是全局内存的一部分，延迟较高；并且寄存器溢出的部分也可以进入 GPU 缓存中。



#### 全局内存

全局内存位于片外，具有容量最大、延迟最大、使用最多的特点。全局内存中的数据所有线程可见、主机端可见，且具有与程序相同的生命周期。全局内存访问开销远大于共享内存，因此在使用全局内存时，尽可能**让一个线程一次访问多个全局内存元素来减少总访问次数**。



全局内存可以动态初始化或静态初始化：

* 使用 cudaMalloc 动态声明内存空间，由 cudaFree 释放全局内存
* 使用 `__device__` 关键字静态声明全局内存，它静态全局内存的数量在编译之前已经确定，只能在**所有主机和设备函数之外定义**，因此要将其放在外部

由于**主机函数不能访问静态全局变量**，因此需要通过下面两个 API 进行数据传输

```cpp
// 从 symbol 传递数据给 dst
__host__ cudaError_t cudaMemcpyFromSymbol ( 
    void* dst, 
    const void* symbol, 
    size_t count, 
    size_t offset = 0, 
    cudaMemcpyKind kind = cudaMemcpyDeviceToHost 
);

// 从 src 传递数据给 symbol
__host__ cudaError_t cudaMemcpyToSymbol ( 
    const void* symbol, 
    const void* src, 
    size_t count, 
    size_t offset = 0, 
    cudaMemcpyKind kind = cudaMemcpyHostToDevice 
);
```

与 cudaMemcpy 不同，它们专门用于常量全局内存的拷贝，后者则是通用的数据传输方法。



例如我们在外部定义静态全局内存，可以在核函数中直接访问，但是主机不能访问，因此需要通过传输函数将主机数据传递过去

```cpp
#include <cuda_runtime.h>
#include <iostream>

#include "common.cuh"

// 静态全局内存
__device__ int d_x = 1;
__device__ int d_y[2];

__global__ void kernel(void)
{
    d_y[0] += d_x;
    d_y[1] += d_x;

    printf("d_x=%d,d_y[0]=%d,d_y[1]=%d.\n", d_x, d_y[0], d_y[1]);
}

int main(void)
{
    // 获得设备信息
    int devID = 0;
    cudaDeviceProp deviceProps;
    ErrorCheck(cudaGetDeviceProperties(&deviceProps, devID), __FILE__, __LINE__);
    std::cout << "运行 GPU 设备：" << deviceProps.name << std::endl;

    // 将 Host h_y 中的数据拷贝到 Device d_y 中
    int h_y[2] = {10, 20};
    ErrorCheck(cudaMemcpyToSymbol(d_y, h_y, sizeof(int) * 2), __FILE__, __LINE__);

    // 调用一次核函数，将 d_y 中的数据 + 1，然后重新拷贝到 h_y 中
    dim3 block(1);
    dim3 grid(1);
    kernel<<<grid, block>>>();
    ErrorCheck(cudaDeviceSynchronize(), __FILE__, __LINE__);
    ErrorCheck(cudaMemcpyFromSymbol(h_y, d_y, sizeof(int) * 2), __FILE__, __LINE__);
    printf("h_y[0]=%d,h_y[1]=%d.\n", h_y[0], h_y[1]);

    ErrorCheck(cudaDeviceReset(), __FILE__, __LINE__);

    return 0;
}
```



#### 共享内存

共享内存在片上 (on-ship)，与本地内存和全局内存相比具有更高的带宽和更低的延迟。共享内存中的数据在线程块中所有线程可见，因此可用于线程间通讯，其生命周期与所属线程块一致。每个 SM 的共享内存数量固定，因此**如果单个线程块中分配过多的共享内存，则会限制活跃线程束的数量**。这种限制是因为共享内存总量非常稀少

![](Screenshot_20240212_222505.jpg)

共享内存用于保存需要经常访问的数据，将它们从全局内存移动到共享内存可以提高访问效率；同时可以改变全局内存访问内存的方式，提高数据访问的带宽。



静态共享内存声明为

```cpp
__shared__ float tile[size];
```

静态共享内存作用域

1. 核函数中声明的内存作用域就是这个核函数
2. 核函数外声明的内存作用域是所有核函数

静态共享内存在编译时就要确定内存大小。在使用共享内存时，需要注意共享内存是对每个线程块定义的，即每个共享内存只有对应的线程块中的线程可以访问

```cpp
__global__ void kernel_1(void)
{
    // 通常定义长度为线程块大小
    // 每个线程块中都会有一个共享内存
    __shared__ float s_array[32];

    for (int i = 0; i < 32; i++)
        s_array[i] = i;

    // 同步同一个线程块中的线程 
    __syncthreads();
}
```

同步函数的目的是确保处于**同一线程块中的所有线程**执行到一个位置。



动态共享内存声明为

```cpp
extern __shared__ float tile[];
```

注意不能使用指针声明，方括号内也不能填值。动态共享内存的大小在核函数调用时指定，例如

```cpp
extern __shared__ float s_array[];

__global__ void kernel_1(void)
{
    for (int i = 0; i < 32; i++)
        s_array[i] = i;

    // 同步同一个线程块中的线程 
    __syncthreads();
}

// 最后一个参数指定动态共享内存的大小
kernel_1<<<grid, block, 32>>>();
```



#### 常量内存

常量内存是有常量缓存的全局内存，数量有限，仅为 64 KB，由于存在缓存，线程束在读取相同的常量内存数据时，访问速度比全局内存快。常量内存中的数据对同一编译单元内的所有线程可见。

> 当线程束中的线程从相同内存地址读取数据时，可以使用常量内存。只需要一个线程读取，然后广播给其它所有线程。

常量内存不能定义在核函数和主机函数中。



静态常量内存可以直接声明时赋值，或者在主机端使用 cudaMemcpyToSymbol 初始化

```cpp
__constant__ float c_data;
__constant__ float c_data2 = 1.0;

int main(void)
{
    // 通过主机端初始化
    float h_data = 1.0;
    cudaMemcpyToSymbol(c_data, &h_data, sizeof(float));
    
    return 0;
}
```

给核函数传递数值参数时，对应的参数变量就存放在常量内存中。例如

```cpp
__global__ void kernel(int N)
{
    // 这里 N 存放在常量内存，每个线程都同时获得 N 
}
```



#### GPU 缓存

GPU 缓存是不可编程的内存。每个 SM 都有一个一级缓存，所有 SM 共用一个二级缓存。每个 SM 有一个只读常量缓存和只读纹理缓存，用于在设备内存中提高读取性能。L1 缓存和 L2 缓存用来存储本地内存和全局内存的数据，也包括寄存器溢出的部分。

> 在 GPU 上只有当访问内存时会经过缓存，而储存到内存时不经过缓存。

![](Screenshot_20240213_001458.jpg)

默认情况下，数据不会缓存在统一的 L1 纹理缓存中，但可以通过编译指令启用缓存

* `-Xptxas -dlcm=ca` 参数指定除了带有禁用缓存修饰符的内联汇编修饰的数据外，所有读取将被缓存
* `-Xptxas -fscm=ca` 参数指定所有数据读取都将被缓存

不是所有 GPU  都支持 L1 缓存查询指令，需要进行判断

```cpp
#include <cuda_runtime.h>
#include <iostream>

#include "../tools/common.cuh"

__global__ void kernel(void)
{
}

int main(void)
{
    // 获得设备信息
    int devID = 0;
    cudaDeviceProp deviceProps;
    ErrorCheck(cudaGetDeviceProperties(&deviceProps, devID), __FILE__, __LINE__);
    std::cout << "运行 GPU 设备：" << deviceProps.name << std::endl;

    // 查询是否支持全局内存 L1 缓存
    if (deviceProps.globalL1CacheSupported)
        std::cout << "支持全局内存 L1 缓存" << std::endl;
    else
        std::cout << "不支持全局内存 L1 缓存" << std::endl;
    std::cout << "L2 缓存大小：" << deviceProps.l2CacheSize / (1024 * 1024) << "M" << std::endl;

    // 调用一次核函数
    dim3 block(1);
    dim3 grid(1);
    kernel<<<grid, block>>>();
    ErrorCheck(cudaDeviceSynchronize(), __FILE__, __LINE__);

    ErrorCheck(cudaDeviceReset(), __FILE__, __LINE__);

    return 0;
}
```



#### 检查内存使用

对之前矩阵加法的示例程序通过 `--resource-usage` 参数编译，可以看到内存的使用情况

```sh
$ nvcc --resource-usage registerNum.cu -o registerNum -arch=sm_61
registerNum.cu
ptxas info    : 0 bytes gmem
ptxas info    : Compiling entry function '_Z9addMatrixPiS_S_ii' for 'sm_61'
ptxas info    : Function properties for _Z9addMatrixPiS_S_ii
    0 bytes stack frame, 0 bytes spill stores, 0 bytes spill loads
ptxas info    : Used 8 registers, 352 bytes cmem[0]
tmpxft_000066f0_00000000-10_registerNum.cudafe1.cpp
  正在创建库 registerNum.lib 和对象 registerNum.exp
```

其中

* gmem 全局内存
* register 寄存器
* cmem 常量内存

注意到在示例中 cudaMalloc 实际上使用了全局内存，但是这里却显示没有使用，是因为这里只记录静态全局内存，不考虑动态内存。



### 计算资源分配

线程束本地执行上下文的资源主要有

* 程序计数器
* 寄存器
* 共享内存

SM 处理的每个线程束计算所需的计算资源属于片上 (on-chip) 资源，因此从一个执行上下文切换到另一个执行上下文没有时间损耗。



对于一个给定的内核，同时存在同一个 SM 中的线程块和线程束的数量，取决于在 SM 中可用的内核所需寄存器和共享内存的数量。

* 如果每个线程消耗的寄存器越多，则可以放在一个 SM 中的线程束就越少；
* 如果减少内核消耗寄存器的数量，SM 便可以同时处理更多的线程束；

![](Screenshot_20240213_003232.jpg)

* 一个线程块消耗的共享内存越多，则在一个 SM 中可以同时处理的线程块就会变少；
* 如果每个线程块使用的共享内存数量变少，则可以同时处理更多的线程块；

![](Screenshot_20240213_003331.jpg)



当计算资源（如寄存器和共享内存）已分配给线程块时，线程块被称为活跃块，线程块所包含的线程束被称为活跃线程束。活跃线程束分为三种：

1. 选定的线程束，即正在执行的线程束
2. 阻塞的线程束，未做好执行准备的线程束
3. 符合条件的线程束，已经准备执行但是尚未执行的线程束

占用率是每个 SM 中活跃线程束占最大线程束数量的比率。



使用下面样例输出 GPU 设备的信息

```cpp
#include <cuda_runtime.h>
#include <iostream>

#include "common.cuh"

int main(int argc, char **argv)
{
    int devID = 0;
    cudaDeviceProp deviceProp;
    ErrorCheck(cudaGetDeviceProperties(&deviceProp, devID), __FILE__, __LINE__);
    std::cout << "运行 GPU 设备：" << deviceProp.name << std::endl;
    std::cout << "SM 数量：" << deviceProp.multiProcessorCount << std::endl;
    std::cout << "L2 缓存大小：" << deviceProp.l2CacheSize / (1024 * 1024) << "M" << std::endl;
    std::cout << "SM 最大驻留线程数量：" << deviceProp.maxThreadsPerMultiProcessor << std::endl;
    std::cout << "设备是否支持流优先级：" << deviceProp.streamPrioritiesSupported << std::endl;
    std::cout << "设备是否支持在 L1 缓存中缓存全局内存：" << deviceProp.globalL1CacheSupported << std::endl;
    std::cout << "设备是否支持在 L1 缓存中缓存本地内存：" << deviceProp.localL1CacheSupported << std::endl;
    std::cout << "一个 SM 可用的最大共享内存量：" << deviceProp.sharedMemPerMultiprocessor / 1024 << "KB" << std::endl;
    std::cout << "一个 SM 可用的 32 位最大寄存器数量：" << deviceProp.regsPerMultiprocessor / 1024 << "K" << std::endl;
    std::cout << "一个 SM 最大驻留线程块数量：" << deviceProp.maxBlocksPerMultiProcessor << std::endl;
    std::cout << "GPU 内存带宽：" << deviceProp.memoryBusWidth << std::endl;
    std::cout << "GPU 内存频率：" << deviceProp.memoryClockRate / (1024 * 1024.0) << "GHz" << std::endl;
    
    ErrorCheck(cudaDeviceReset(), __FILE__, __LINE__);

    return 0;
}
```

可以总结得到下面的参数表

| 计算能力                | 8.9             |
| ----------------------- | --------------- |
| GPU 型号                | RTX 4050 Laptop |
| SM 数量                 | 20              |
| SM 寄存器数量           | 64K             |
| SM 共享内存上限         | 100KB           |
| 单线程块共享内存上限    | 99KB            |
| SM 中最多驻留线程块数量 | 24              |
| SM 中最多驻留线程数量   | 1536            |



以计算能力 8.9 为例：

1. 一个 SM 最多拥有 24 个线程块
2. 一个 SM 最多拥有 1536 个线程

在并行规模足够大（即调用的总线程数足够多）的前提下：

1. 在寄存器和共享内存使用很少时，线程块大小不小于 $1536 / 24=64$ 时可以获得 100% 的占用率；
2. 寄存器数量会影响占有率：当在 SM 上驻留 1536 个线程，则核函数中每个线程最多使用 $64000/1536\approx 42$ 个寄存器；
3. 共享内存大小会影响占有率：若线程块大小为 64，每个 SM 激活 24 个线程块，使得 SM 上驻留 1536 个线程，占有率达到 100% 时每个线程块可分配 $100 / 24\approx 4.16$ KB 共享内存；反之，如果每个线程块占用 8.32 KB 的共享内存，则占有率不会超过 50%；

网格和线程块大小的准则：

1. 保持每个线程块中线程数量是线程束大小 32 的倍数，避免多余线程束中出现空闲的线程；
2. 线程块不要太小，尽量使总线程数达到 1536 个；
3. 根据内核资源调整线程块的大小；
4. 线程块的数量要远远大于 SM 的数量 20，确保实现延迟隐藏，提高利用率。



## 程序优化

### 延迟隐藏

GPU 中指令发出和完成之间的时钟周期被定义为指令延迟。当每个时钟周期中所有线程束调度器都有一个符合条件的线程束时，可以达到计算资源的完全利用。例如一个线程束中的指令延迟为 4 个时钟周期时，指令在 A 时发出，B 时完成。如果在此期间有另一个线程束在 A' 时发出，B' 时完成，则前一个线程束的延迟被这一个线程束的计算隐藏，这就是延迟隐藏。

![](Screenshot_20240214_212025.jpg)



#### 算术指令隐藏

算术运算指令延迟是从开始运算到得到计算结果的时钟周期，通常为 4 个时钟周期。满足延迟隐藏所需的线程束数量，利用利特尔法则可以提供一个估计值：所需线程束数量 = 延迟 x 吞吐量。

* 带宽：理论上的数据交换峰值
* 吞吐量：实际上达到的值

吞吐量是 SM 中每个时钟周期的操作数量确定的。

![](Screenshot_20240214_212527.jpg)

根据官方给出的参数

|        | 指令延迟（周期） | 吞吐量（操作/周期） | 指令操作数量（操作） |
| :----: | :--------------: | :-----------------: | :------------------: |
| 16-bit |        4         |         128         |         512          |
| 32-bit |        4         |         128         |         512          |
| 64-bit |        4         |          2          |          8           |

可以得到

* 16-bit 所需线程束数量为 $512/32=16$​；
* 32-bit 所需线程束数量为 $512/32=16$；
* 64-bit 所需线程束数量为 $8/32=1$（8 个操作也需要一个线程束）；



#### 内存指令隐藏

内存访问指令延迟是从命令发出到数据到达目的地的时钟周期，通常为 400-800 个时钟周期。对于内存操作来说，所需的并行可以表示为在每个时钟周期内隐藏内存延迟所需的字节数。例如

| 指令延迟（周期） | BandWidt(G/S) | BandWidth(B/cycle) | GPU 内存频率（GHz） | 内存操作字节数量 |
| :--------------: | :-----------: | :----------------: | :-----------------: | :--------------: |
|       800        |     504.2     |         50         |       10.0145       |        39        |

其中参数

* $504.2 / 10.0145\approx 50$
* $800\times 50/ 1024=39$

假设每个线程都把一个 4 字节浮点数从全局内存移动到 SM 中，则要用非常大量线程来填补每次移动操作的延迟。事实上，有

* $390000/4\approx 10000$ 个线程
* $10000/32\approx 313$ 个线程束

即需要 10000 个线程或 313 个线程束来隐藏所有内存延迟。



### 并行归约计算

在向量中满足交换律和结合律的运算，称为归约问题。并行执行的归约计算称为并行归约计算。

* 领域并行：每次计算相邻元素。
* 间域并行：每次计算等距元素。

![](Screenshot_20240214_222422.jpg)



#### 线程束分支

GPU 是相对简单的设备，没有复杂的分支预测机制。我们尽量让一个线程束中的所有线程在同一个周期中执行相同的指令，否则会产生线程束分支。例如我们让奇数号线程与偶数号线程执行不同的操作，这时每个线程束中有一半的线程执行不同操作。这会导致

* 当线程束中的线程执行 if 语句时，其余执行 else 语句的线程阻塞；
* 反之，当执行 else 语句时，其余执行 if 语句的线程阻塞；

因此产生了资源浪费。

![](Screenshot_20240214_221438.jpg)



线程束分支会降低 GPU 的并行能力，条件分支越多，并行性削弱越严重。

> 线程束分支只发生在同一个线程束中。

为了获取最佳性能，应当避免**同一个线程束**中有不同的执行路径。例如修改先前的代码

```cpp
if ((tid / 32) % 2 == 0)
    a = 10.0f;
else
    b = 20.0f;
```

这样代码逻辑与之前相同，但是我们确保了**同一个线程束**中的线程执行同一个条件代码。



#### 邻域并行

假设要计算 4096 个元素和，设计线程块大小为 512，每个线程负责一个数据元素，共需 8 个线程块。每计算一次，总元素数减少一半，最终 8 个线程块计算出 8 个结果。由于线程块之间无法同步，我们将 8 个结果复制回主机，在 CPU 上求和。

![](Screenshot_20240214_222737.jpg)



我们有两种计算方法。第一种是将 4096 个线程与 4096 个元素依次对应，然后分组计算求和。这种计算方法就会导致

* 第一轮计算时，16 个线程块都有 16 个空闲线程；
* 第二轮计算时，8 个线程块都有 16 个空闲线程……

这将是严重的线程束分化。第二种是每轮计算时都重新对应元素

* 第一轮计算时，8 个线程块计算，8 个线程块空闲；
*  第二轮计算时，4 个线程块计算，12 个线程块空闲……

线程块空闲不会影响计算性能。当然，最后几轮计算时，数据量少于 32 时，也会产生线程束分化。

![](Screenshot_20240214_223033.jpg)



带分支的归约算法执行代码为

```cpp
__global__ void ReduceNeighboredWithDivergence(float *d_idata, float *d_odata, int size)
{
    // 获取线程 ID
    unsigned int tid = threadIdx.x;
    unsigned int idx = blockIdx.x * blockDim.x + tid;

    // 获得当前线程对应的块
    float *idata = d_idata + blockIdx.x * blockDim.x;

    // 边界检测
    if (idx >= size)
        return;

    // 内部归约：每次循环一次，需要调用的线程数减半
    for (int i = 1; i < blockDim.x; i *= 2)
    {
        // 第 i 次循环，使用 0, 2i, 4i, ... 线程计算，归约到 0, 2i, 4i, ... 对应的元素
        if ((tid % (2 * i)) == 0)
            idata[tid] += idata[tid + i];

        // 线程同步
        __syncthreads();
    }

    // 写入结果（只需要一个线程块写入一次即可，这里只让每个线程块中的第一个线程写入）
    if (tid == 0)
        d_odata[blockIdx.x] = idata[0];
}
```



无分支归约算法执行代码为

```cpp
__global__ void ReduceNeighboredWithoutDivergence(float *d_idata, float *d_odata, int size)
{
    // 获取线程 ID
    unsigned int tid = threadIdx.x;
    unsigned int idx = blockIdx.x * blockDim.x + tid;

    // 获得当前线程对应的块
    float *idata = d_idata + blockIdx.x * blockDim.x;

    // 边界检测
    if (idx >= size)
        return;

    // 内部归约：每次循环一次，需要调用的线程数减半
    for (int i = 1; i < blockDim.x; i *= 2)
    {
        // 第 i 次循环，第 tid 个线程操作 2 * i * tid 对应的元素
        // 这样会有一部分线程对应元素超出数组范围，它们什么都不做
        int index = 2 * i * tid;

        if (index < blockDim.x)
            idata[index] += idata[index + i];

        // 线程同步
        __syncthreads();
    }

    // 写入结果（只需要一个线程块写入一次即可，这里只让每个线程块中的第一个线程写入）
    if (tid == 0)
        d_odata[blockIdx.x] = idata[0];
}
```



#### 间域并行

共享内存被组织成若干个 bank，每个 bank 可以在一个时钟周期内独立地服务一个内存请求。当两个或多个线程**同时访问同一个 bank 中不同地址的数据**时，会发生 bank 冲突。这些访问不能在同一个时钟周期内完成，必须串行化，从而降低了访问速度。

> 如果所有线程访问的是同一个地址（即广播访问），则不会产生冲突。但如果是访问不同的地址，就会产生 bank 冲突。

为了避免 bank 冲突，可以使用数据对齐方式访问。这就是间域并行的优势：每个线程访问连续的内存可以尽量避免对同一个 bank 的过多访问，从而提高并行效率。

![](Screenshot_20240222_113117.jpg)



优化归约代码为

```cpp
// 内部归约：每次循环一次，需要调用的线程数减半
for (int i = blockDim.x / 2; i > 0; i /= 2)
{
    // 第 1 次循环，调用 0,1,2,...,dim/2 线程，对应 0,1,2,...,dim/2 元素，间隔为 i
    // 第 2 次循环，调用 0,1,2,...,dim/4 线程，对应 0,1,2,...,dim/4 元素，间隔为 i
    if (tid < i)
        idata[tid] += idata[tid + i];

    // 线程同步
    __syncthreads();
}
```



#### 循环展开

上述计算到了最后几步，元素数少于 32 时，就只有一个线程块工作，这时候就不需要进行同步。并且可以对操作循环展开

```cpp
// 内部归约：每次循环一次，需要调用的线程数减半
// 当少于 32 个元素时退出
for (int i = blockDim.x / 2; i > 32; i /= 2)
{
    // 第 1 次循环，调用 0,1,2,...,dim/2 线程，对应 0,1,2,...,dim/2 元素，间隔为 i
    // 第 2 次循环，调用 0,1,2,...,dim/4 线程，对应 0,1,2,...,dim/4 元素，间隔为 i
    if (tid < i)
        idata[tid] += idata[tid + i];

    // 线程同步
    __syncthreads();
}

// 循环展开，注意到 < 32 的线程都在一个线程束中，因此不存在线程束分支问题
if (tid < 32)
{
    idata[tid] += idata[tid + 32];
    
    // 每一行都是原先的一步循环，省去了 __syncthreads();
    
    idata[tid] += idata[tid + 16];
    
    // 每一行都是原先的一步循环，省去了 __syncthreads();
    
    idata[tid] += idata[tid + 8];
    idata[tid] += idata[tid + 4];
    idata[tid] += idata[tid + 2];
    idata[tid] += idata[tid + 1];
}

// 写入结果（只需要一个线程块写入一次即可，这里只让每个线程块中的第一个线程写入）
if (tid == 0)
    d_odata[blockIdx.x] = idata[0];
```

循环展开似乎依然需要每一步都同时进行，由于每步计算量相同，因此不再需要手动同步。



#### 完全循环展开

在特定情形下，例如指定线程块大小为 512，可以针对这种大小的线程块将循环展开，从而进一步提高效率。为了增加通用性，使用模板函数

```cpp
template <unsigned int blockSize> __global__ void reduce(float *d_idata, float *d_odata, int size)
{
    // 获取线程 ID
    unsigned int tid = threadIdx.x;
    unsigned int idx = blockIdx.x * blockDim.x + tid;

    // 获得当前线程对应的块
    double *idata = d_idata + blockIdx.x * blockDim.x;

    // 边界检测
    if (idx >= size)
        return;

    if (blockSize >= 512)
    {
        if (tid < 256)
            idata[tid] += idata[tid + 256];

        __syncthreads();
    }

    if (blockSize >= 256)
    {
        if (tid < 128)
            idata[tid] += idata[tid + 128];

        __syncthreads();
    }

    if (blockSize >= 128)
    {
        if (tid < 64)
            idata[tid] += idata[tid + 64];

        __syncthreads();
    }
	
    // 当线程块规模不超过 64，就可以只用一个线程束进行计算，因此与 tid < 32 的情况合并
    
    if (tid < 32)
    {
        // 利用 >= 巧妙展开
        if (blockSize >= 64)
            idata[tid] += idata[tid + 32];
        if (blockSize >= 32)
            idata[tid] += idata[tid + 16];
        if (blockSize >= 16)
            idata[tid] += idata[tid + 8];
        if (blockSize >= 8)
            idata[tid] += idata[tid + 4];
        if (blockSize >= 4)
            idata[tid] += idata[tid + 2];
        if (blockSize >= 2)
            idata[tid] += idata[tid + 1];
    }

    // 写入结果（只需要一个线程块写入一次即可，这里只让每个线程块中的第一个线程写入）
    if (tid == 0)
        d_odata[blockIdx.x] = idata[0];
}
```

其中与模板参数 blockSize 相关的比较将会在编译阶段计算并优化，最终会得到非常高效的循环。



#### 防止编译优化

有些时候我们不希望编译器对某些内存操作进行优化，例如

```cpp
__shared__ volatile float data[100];
```

对于共享内存，程序运行过程中可能被其它进程修改，因此编译器可能会调整代码中对该内存的操作顺序，这时候就需要 `volatile` 修饰符禁止编译器优化对该内存的操作。



#### 测试样例

为了方便测试，我们首先定义一个计时器类，计算 CPU 和 GPU 的执行时间

```cpp
#pragma once

#include <memory>
#include <random>
#include <ratio>
#include <stdio.h>
#include <cuda_runtime.h>
#include <chrono>

cudaError_t ErrorCheck(cudaError_t error_code, const char *filename, int lineNumber)
{
    if (error_code != cudaSuccess)
    {
        printf("CUDA error:\r\ncode=%d, name=%s, description=%s\r\nfile=%s, "
               "line%d\r\n",
               error_code, 
               cudaGetErrorName(error_code), cudaGetErrorString(error_code), filename, lineNumber);
        return error_code;
    }
    return error_code;
}

class Timer
{
  public:
    using s = std::ratio<1, 1>;
    using ms = std::ratio<1, 1000>;
    using us = std::ratio<1, 1000000>;
    using ns = std::ratio<1, 1000000000>;

  private:
    std::chrono::time_point<std::chrono::high_resolution_clock> _cStart;
    std::chrono::time_point<std::chrono::high_resolution_clock> _cStop;
    cudaEvent_t _gStart;
    cudaEvent_t _gStop;
    float _timeElasped;

  public:
    Timer()
    {
        // 创建事件变量
        ErrorCheck(cudaEventCreate(&_gStart), __FILE__, __LINE__);
        ErrorCheck(cudaEventCreate(&_gStop), __FILE__, __LINE__);
    }

    ~Timer()
    {
        // 销毁事件变量
        ErrorCheck(cudaEventDestroy(_gStart), __FILE__, __LINE__);
        ErrorCheck(cudaEventDestroy(_gStop), __FILE__, __LINE__);
    }

    void start_cpu()
    {
        _cStart = std::chrono::high_resolution_clock::now();
    }

    void start_gpu()
    {
        // 记录开始时间
        ErrorCheck(cudaEventRecord(_gStart), __FILE__, __LINE__);
        cudaEventQuery(_gStart); // 此处不可用错误检测函数
    }

    void stop_cpu()
    {
        _cStop = std::chrono::high_resolution_clock::now();
    }

    void stop_gpu()
    {
        // 计算时长
        ErrorCheck(cudaEventRecord(_gStop), __FILE__, __LINE__);
        ErrorCheck(cudaEventSynchronize(_gStop), __FILE__, __LINE__);
        ErrorCheck(cudaEventElapsedTime(&_timeElasped, _gStart, _gStop), __FILE__, __LINE__);
    }

    // span 是时间单位
    template <typename span> void duration_cpu(std::string msg)
    {
        std::string str;

        if (std::is_same<span, s>::value)
            str = "s";
        else if (std::is_same<span, ms>::value)
            str = "ms";
        else if (std::is_same<span, us>::value)
            str = "us";
        else if (std::is_same<span, ns>::value)
            str = "ns";

        std::chrono::duration<double, span> time = _cStop - _cStart;
        printf("%-40s uses %.6lf %s\n", msg.c_str(), time.count(), str.c_str());
    }

    void duration_gpu(std::string msg)
    {
        printf("%-40s uses %.6lf ms\n", msg.c_str(), _timeElasped);
    }
};
```



### 指令优化

对于 float, int 的加乘运算，int 的偏移、取最值操作，每个线程束需要 4 个循环。int 乘法默认 32 位，需要多个循环，可以使用

```cpp
__[u]mul24()
```

在 int 值比较小的时候，使用 24 位乘法操作。



对于整数除法和取模操作，如果除数是 2 的幂，就存在一定优化空间。例如设 n 是 2 的幂，则

```cpp
foo % n == foo & (n-1)
```

右边的操作会更高效。



计算特殊函数时，也可以采用内置的特殊函数。例如 `sin(), exp(), pow()` 都存在针对 GPU 加速的版本

```cpp
__sin(), __exp(), __pow()
```

它们远远快于标准库函数，但是精度会更低。



### 程序分析

#### 性能剖析

使用 nvprof 可以分析程序的性能

```sh
$ nvprof test.exe
```

不过这一指令在计算能力 8.0 及以上的版本不再使用。作为替代，使用 C:\Program Files\NVIDIA Corporation\Nsight Systems 2023.3.3\target-windows-x64 目录下的 nsys 指令。



#### 错误分析

最开始容易遇到的错误是编译问题。尤其是在多个文件编译时，某些 API 函数可能会出现“未定义标识符”的错误。这是因为文件没有被识别为 CUDA 代码文件，最好修改后缀为 .cu 或 .cuh 而不是 .cpp 等。



#### 运行信息

使用 Nsight Systems 分析程序的运行信息。只需要给出可执行文件所在路径，以及需要执行的命令行参数，点击 "Start" 即可开始分析

![](image-20240216180156110.png)

执行后，在 `Processes-main.exe-CUDA HW` 下可以看到对应核函数的执行占比。右键显示在事件窗口，可以看到启动时间和持续时间。

![](image-20240216181249110.png)

可以在右侧窗口中选中一段，右键选择 `Filter and Zoom in` 放大到对应位置，可以看到对应的执行时长相对于其它操作的比例

![](image-20240216181343094.png)
