# OpenMP

## 基本介绍

基本并行概念

- 指令级并行 - CPU 流水线
- 分布式并行 - MPI
- 共享储存式并行 - OpenMP



### OpenMP

Open Multi-Processing 可启用多个线程，利用计算机上的多个核心并行。核心数量指 CPU 的物理核心数，超线程技术对计算密集型代码没有帮助。OMP 库包含在编译器中，可以直接使用。



### MPI

OpenMP 只能在单机上使用，而 Message Passing Interface 可实现跨节点并行。MPI 在每个节点启用多线程并行执行。



## 基础知识

### OMP 标记

引入特殊的标记表示启用并行

```cpp
#pragma omp
```



### 并行域

OMP 将 `{}` 组成的作用域作为并行域

```cpp
#pragma omp parallel clause ...
{
    // 并行域
}
```

并行域中的代码不一定都会并行执行。



