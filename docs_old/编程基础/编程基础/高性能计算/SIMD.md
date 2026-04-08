# SIMD

## 基础入门

### SSE 指令集

| Header          | Extentions                            |
| --------------- | ------------------------------------- |
| `<mmintrin.h>`  | MMX                                   |
| `<xmintrin.h>`  | SSE                                   |
| `<emintrin.h>`  | SSE2                                  |
| `<pmintrin.h>`  | SSE3                                  |
| `<tmintrin.h>`  | SSSE3                                 |
| `<smintrin.h>`  | `SSE4.1`                              |
| `<nmintrin.h>`  | `SSE4.2`                              |
| `<wmintrin.h>`  | AES                                   |
| `<imintrin.h>`  | `AVX, AVX2, FMA, BMI, POPCNT, AVX512` |
| `<x86intrin.h>` | Auto (GCC)                            |
| `<intrin.h>`    | Auto (MSVC)                           |

建议导入 `<imintrin.h>`，自动包含所有指令集扩展。



可以通过 CPU-Z 查看处理器支持的指令集类型

![](SIMD.assets/image-20240927214840572.png|500)



### 基本运算

封装的指令集中提供基本的写入、读取和加减函数

```cpp
float x = 1.0f;
float y = 3.3f;
float z = 2.1f;
float w = -4.2f;

// _mm_setr_ps
__m128 m = _mm_set_ps(w, z, y, x);
__m128 one = _mm_set1_ps(1.0f);
m = _mm_add_ps(m, one);

std::vector<float> v(4);
_mm_store_ps(v.data(), m);
for (auto i : v) {
	std::cout << i << " ";
}
std::cout << std::endl;
```

其中 `_mm_setr_ps` 赋值方向与 `_mm_set_ps` 相反。



### 储存类型

使用不同的后缀名来使用对应类型或函数

```cpp
#include <immintrin.h>
#include <iostream>
#include <vector>

template <class Ty = float, class T> void print(T t) {
  if constexpr (std::is_convertible_v<T, float>) {
    std::cout << t << std::endl;
  } else {
    constexpr int size = sizeof(T) / sizeof(Ty);
    std::vector<Ty> v(size);
    if constexpr (std::is_same_v<Ty, double>) {
      _mm_store_pd(v.data(), t);
    } else {
      _mm_store_ps((float *)v.data(), t);
    }
    for (auto i : v) {
      std::cout << i << " ";
    }
    std::cout << std::endl;
  }
}

int main() {
  {
    // 双精度浮点 double
    __m128d m = _mm_setr_pd(1, 2);
    __m128d two = _mm_set1_pd(2);
    m = _mm_mul_pd(m, two);
    print<double>(m);
  }
  {
    // 单精度浮点 float
    __m128 m = _mm_setr_ps(1, 2, 3, 4);
    __m128 two = _mm_set1_ps(2);
    m = _mm_mul_ps(m, two);
    print(m);
  }
  {
    // 32 位整型 int
    __m128i m = _mm_setr_epi32(1, 2, 3, 4);
    __m128i two = _mm_set1_epi32(2);
    print<int>(_mm_mul_epi32(m, two));
    print<int>(_mm_mullo_epi32(m, two));
  }
  return 0;
}
```

其中 `_mm_mul_epi32` 将每块 32 位元素对应相乘，并将结果存放在每个 64 位块中

```cpp
_mm_mul_epi32(m, two);
// | 4 | 3 | 2 | 1 |
// | 2 | 2 | 2 | 2 |
// | 0 | 6 | 0 | 2 |
```

而 `_mm_mullo_epi32` 将每块 32 位元素对应相乘，并将结果的低位存放在块中

```cpp
_mm_mullo_epi32(m, two);
// | 4 | 3 | 2 | 1 |
// | 2 | 2 | 2 | 2 |
// | 8 | 6 | 4 | 2 |
```



### 类型转换

可以在不同的 `__m128` 类型之间转换。转换函数例如

| 函数               | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| `_mm_castps_si128` | 按位将单精度浮点转换为 `__m128i` 类型                        |
| `_mm_cvtps_epi32`  | 将单精度浮点转换为 `__m128i` 类型，每块 32 位。向最靠近的整数取整，如果小数位为 `0.5`，则向最靠近的偶数取整，防止整体平均值偏移。 |
| `_mm_cvttps_epi32` | 将单精度浮点转换为 `__m128i` 类型，每块 32 位。截断小数位取整。 |

可以使用 `_MM_SET_ROUNDING_MODE` 设置取整模式，使用默认的 `_MM_ROUND_NEAREST` 即可。

```cpp
//_MM_SET_ROUNDING_MODE(_MM_ROUND_DOWN);
__m128 m = _mm_setr_ps(1.1f, 2.2f, 3.8f, 4.9f);
print(m);
__m128i mcast = _mm_castps_si128(m);
print<int>(mcast);
__m128i mcvt = _mm_cvtps_epi32(m);
print<int>(mcvt);
__m128i mcvtt = _mm_cvttps_epi32(m);
print<int>(mcvtt);
```



### 内存重排

使用 `shuffle` 指令重排内存中的块。首先是对 32 位整型的重排

```cpp
__m128i m = _mm_set_epi32(4, 3, 2, 1);
__m128i ms = _mm_shuffle_epi32(m, _MM_SHUFFLE(0, 2, 1, 3));
print<int>(ms);
// 等价的写法
__m128i ms2 = _mm_shuffle_epi32(m, 0b00100111);
print<int>(ms2);
```

以及对单精度浮点的重排

```cpp
__m128 m = _mm_set_ps(4, 3, 2, 1);
__m128 n = _mm_set_ps(8, 7, 6, 5);
__m128 mns = _mm_shuffle_ps(m, n, _MM_SHUFFLE(2, 3, 0, 1));
print(mns);
// 等价的写法
__m128 mns2 = _mm_shuffle_ps(m, n, 0b10110001);
print(mns2);
```

其中 `_MM_SHUFFLE` 用于指定掩模。重排会打乱每个 32 位块的顺序，例如

```cpp
_mm_shuffle_ps(a, b, 0b01 11 10 00);
// a0, b0 为最低位
// | a3 | a2 | a1 | a0 |
// | b3 | b2 | b1 | b0 |

// 混合后得到
// | 01 | 11 | 10 | 00 |
// |  1 |  3 |  2 |  0 |
// | b1 | b3 | a2 | a0 |
```



利用重排，实现将 `__m128` 变量中的 4 个浮点相加

```cpp
__m128 m = _mm_set_ps(4, 3, 2, 1);
__m128 n = _mm_shuffle_ps(m, m, _MM_SHUFFLE(2, 3, 0, 1));
print(n);
m = _mm_add_ps(m, n);
print(m);
n = _mm_shuffle_ps(m, m, _MM_SHUFFLE(0, 0, 2, 2));
print(n);
// 只加最后一个单精度数
m = _mm_add_ss(m, n);
print(m);
// 转换最后一个单精度数
float x = _mm_cvtss_f32(m);
print(x);
```



根据掩模的二进制值，可以直接指定索引重排来获得对应的块

```cpp
__m128 m = _mm_set_ps(4, 3, 2, 1);
float x = _mm_cvtss_f32(m);
print(x);
x = _mm_cvtss_f32(_mm_shuffle_ps(m, m, 1));	// 0b00000001
print(x);
x = _mm_cvtss_f32(_mm_shuffle_ps(m, m, 2));	// 0b00000010
print(x);
x = _mm_cvtss_f32(_mm_shuffle_ps(m, m, 3));	// 0b00000011
print(x);
```



### 内存读写

使用 `load, store` 指令读写内存

```cpp
_mm_load_ps();
_mm_loadu_ps();
_mm_store_ps();
_mm_storeu_ps();

_mm256_load_ps();
_mm256_loadu_ps();
_mm256_store_ps();
_mm256_storeu_ps();
```

其中 `u` 表示是否要求对齐，通常 `__m128` 变量读写要求 16 字节对齐，`__m256` 变量读写要求 32 字节对齐。而浮点数 `float a[8]` 只保证对齐到 16 字节，因此要使用 `__m256` 变量就需要显式要求对齐

```cpp 
alignas(32) float a[8];
```

最好能够使用 `u` 命令，在现代 CPU 中 `u` 命令对性能影响可以忽略。



## 实战案例

### 线性组合

我们使用如下几个矢量化函数j

| 函数            | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| `_mm_loadu_ps`  | 装载 4 个 32 位连续内存                                      |
| `_mm_add_ps`    | 对 `__m128` 变量分块对应相加，每块 32 位                     |
| `_mm_mul_ps`    | 对 `__m128` 变量分块对应相乘，每块 32 位                     |
| `_mm_set_ps`    | 设置 `__m128` 变量的 4 个块的值。从高位向低位填充，例如填充内存为 `abcd` 需要使用 `_mm_set_ps(a,b,c,d)` 填充 |
| `_mm_set1_ps`   | 将 1 个 32 位浮点填充到 4 个块中                             |
| `_mm_storeu_ps` | 储存 4 个 32 位连续内存                                      |



#### 基础样例

两个数组循环计算的样例如下

```cpp
constexpr int shift = 20;

void add(benchmark::State &state) {
  auto f = [](float a, const float *x, const float *y, float *z,
              std::size_t n) {
    for (std::size_t i = 0; i < n; i++) {
      z[i] = a * x[i] + y[i];
    }
  };

  auto n = std::size_t(1 << shift);
  auto a = 0.5f;
  std::vector<float> x(n);
  std::vector<float> y(n);
  std::vector<float> z(n);

  for (auto _ : state) {
    f(a, x.data(), y.data(), z.data(), n);
    benchmark::DoNotOptimize(z.data());
  }
}
BENCHMARK(add);
```



#### 矢量化

利用 `ps` 矢量化指令和 `__m128` 寄存器变量优化循环，由于 `ps` 指令会对储存值按 16 字节分块，然后每块对应相加，可以计算 4 个浮点相加，因此可以同时计算 4 次加法。

```cpp
void add_ps(benchmark::State &state) {
  auto f = [](float a, const float *x, const float *y, float *z,
              std::size_t n) {
    for (std::size_t i = 0; i < n; i += 4) {
      // loadu means memory doesn't need to be aligned
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 yi = _mm_loadu_ps(&y[i]);
      __m128 zi = _mm_add_ps(_mm_mul_ps(_mm_set1_ps(a), xi), yi);
      _mm_storeu_ps(&z[i], zi);
    }
  };

  auto n = std::size_t(1 << shift);
  auto a = 0.5f;
  std::vector<float> x(n);
  std::vector<float> y(n);
  std::vector<float> z(n);

  for (auto _ : state) {
    f(a, x.data(), y.data(), z.data(), n);
    benchmark::DoNotOptimize(z);
  }
}
BENCHMARK(add_ps);
```

其中 `_mm_loadu_ps` 用于装载可能非 16 字节对齐的内存，如果能够确保对齐，可以使用 `_mm_load_ps`，不过性能不会提高多少。



#### 排除边界

如果数组长度不是 4 的倍数，可以手动排除边界

```cpp
void add_ps_aligned(benchmark::State &state) {
  auto f = [](float a, const float *x, const float *y, float *z,
              std::size_t n) {
    std::size_t i;
    for (i = 0; i + 3 < n; i += 4) {
      // loadu means memory doesn't need to be aligned
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 yi = _mm_loadu_ps(&y[i]);
      __m128 zi = _mm_add_ps(_mm_mul_ps(_mm_set1_ps(a), xi), yi);
      _mm_storeu_ps(&z[i], zi);
    }
    for (; i < n; i++) {
      z[i] = a * x[i] + y[i];
    }
  };

  auto n = std::size_t(1 << shift);
  auto a = 0.5f;
  std::vector<float> x(n);
  std::vector<float> y(n);
  std::vector<float> z(n);

  for (auto _ : state) {
    f(a, x.data(), y.data(), z.data(), n);
    benchmark::DoNotOptimize(z);
  }
}
BENCHMARK(add_ps_aligned);
```



#### 循环展开

还可以进一步将两步计算合并到一次循环中

```cpp
void add_ps_aligned_8(benchmark::State &state) {
  auto f = [](float a, const float *x, const float *y, float *z,
              std::size_t n) {
    std::size_t i;
    __m128 av = _mm_set1_ps(a);
    for (i = 0; i + 7 < n; i += 8) {
      // loadu means memory doesn't need to be aligned
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 xi2 = _mm_loadu_ps(&x[i + 4]);
      __m128 yi = _mm_loadu_ps(&y[i]);
      __m128 yi2 = _mm_loadu_ps(&y[i + 4]);
      __m128 zi = _mm_add_ps(_mm_mul_ps(av, xi), yi);
      __m128 zi2 = _mm_add_ps(_mm_mul_ps(av, xi2), yi2);
      _mm_storeu_ps(&z[i], zi);
      _mm_storeu_ps(&z[i + 4], zi2);
    }
    for (; i < n; i++) {
      z[i] = a * x[i] + y[i];
    }
  };

  auto n = std::size_t(1 << shift);
  auto a = 0.5f;
  std::vector<float> x(n);
  std::vector<float> y(n);
  std::vector<float> z(n);

  for (auto _ : state) {
    f(a, x.data(), y.data(), z.data(), n);
    benchmark::DoNotOptimize(z);
  }
}
BENCHMARK(add_ps_aligned_8);
```



在开启 `/O2` 优化的情况下，最终计算耗时

```shell
|| add                 1182230 ns      1024933 ns          747
|| add_ps               935687 ns       927734 ns          640
|| add_ps_aligned       929607 ns       920348 ns          747
|| add_ps_aligned_8     954808 ns       962182 ns          747      
```



### 循环求和

我们使用如下几个矢量化函数

| 函数             | 作用                                     |
| ---------------- | ---------------------------------------- |
| `_mm_setzero_ps` | 初始化 `__m128` 寄存器中的每个浮点数为 0 |
| `_mm_shuffle_ps` | 打乱 `__m128` 中各个 32 位块的顺序       |
| `_mm_add_ss`     | 只对 `__m128` 中的低 32 位做加法         |
| `_mm_cvtss_f32`  | 将 `__m128` 寄存器的低 32 位转换为浮点数 |



#### 基础样例

首先给出最简单的求和循环

```cpp
constexpr int shift = 20;

void sum(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });

  auto f = [](const float *x, std::size_t n) {
    float ret = 0.0f;
    for (std::size_t i = 0; i < n; i++) {
      ret += x[i];
    }
    return ret;
  };

  for (auto _ : state) {
    float ret = f(x.data(), n);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(sum);
```



#### 矢量化

然后仿照前面的方式矢量化

```cpp
void sum_ps(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });

  auto f = [](const float *x, std::size_t n) {
    __m128 ret = _mm_setzero_ps();
    for (std::size_t i = 0; i < n; i += 4) {
      __m128 xi = _mm_loadu_ps(&x[i]);
      ret = _mm_add_ps(ret, xi);
    }
    float retarr[4];
    _mm_storeu_ps(retarr, ret);
    return retarr[0] + retarr[1] + retarr[2] + retarr[3];
  };

  for (auto _ : state) {
    float ret = f(x.data(), n);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(sum_ps);
```



#### 循环展开

注意到在 `x86` 下共有 8 个 XMM 寄存器，我们这里对循环做展开

```cpp
void sum_ps_16(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });

  auto f = [](const float *x, std::size_t n) {
    __m128 ret = _mm_setzero_ps();
    __m128 ret2 = _mm_setzero_ps();
    for (std::size_t i = 0; i < n; i += 16) {
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 xi2 = _mm_loadu_ps(&x[i + 4]);
      __m128 xi3 = _mm_loadu_ps(&x[i + 8]);
      __m128 xi4 = _mm_loadu_ps(&x[i + 12]);
      xi = _mm_add_ps(xi, xi3);
      xi2 = _mm_add_ps(xi2, xi4);
      ret = _mm_add_ps(ret, xi);
      ret2 = _mm_add_ps(ret2, xi2);
    }
    float retarr[4];
    ret = _mm_add_ps(ret, ret2);
    _mm_storeu_ps(retarr, ret);
    return retarr[0] + retarr[1] + retarr[2] + retarr[3];
  };

  for (auto _ : state) {
    float ret = f(x.data(), n);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(sum_ps_16);
```



#### 优化累加

可以使用 `shuffle` 进一步优化最后的累加方法

```cpp
void sum_ps_16_shuffle(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });

  auto f = [](const float *x, std::size_t n) {
    __m128 ret = _mm_setzero_ps();
    __m128 ret2 = _mm_setzero_ps();
    for (std::size_t i = 0; i < n; i += 16) {
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 xi2 = _mm_loadu_ps(&x[i + 4]);
      __m128 xi3 = _mm_loadu_ps(&x[i + 8]);
      __m128 xi4 = _mm_loadu_ps(&x[i + 12]);
      xi = _mm_add_ps(xi, xi3);
      xi2 = _mm_add_ps(xi2, xi4);
      ret = _mm_add_ps(ret, xi);
      ret2 = _mm_add_ps(ret2, xi2);
    }
    ret = _mm_add_ps(ret, ret2);
    ret = _mm_add_ss(ret, _mm_shuffle_ps(ret, ret, _MM_SHUFFLE(0, 0, 0, 0)));
    ret = _mm_add_ss(ret, _mm_shuffle_ps(ret, ret, _MM_SHUFFLE(0, 0, 0, 1)));
    ret = _mm_add_ss(ret, _mm_shuffle_ps(ret, ret, _MM_SHUFFLE(0, 0, 0, 2)));
    ret = _mm_add_ss(ret, _mm_shuffle_ps(ret, ret, _MM_SHUFFLE(0, 0, 0, 3)));
    return _mm_cvtss_f32(ret);
  };

  for (auto _ : state) {
    float ret = f(x.data(), n);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(sum_ps_16_shuffle);
```

这其中 `_mm_add_ss` 只对低 32 位求和；最后 `_mm_cvtss_f32` 获得低 32 位结果。



计算耗时

```shell
|| sum                  3265835 ns      2913136 ns          236
|| sum_ps                717943 ns       714983 ns          896
|| sum_ps_16             321957 ns       322316 ns         2133
|| sum_ps_16_shuffle     316316 ns       320871 ns         2240    
```



### 条件计数

我们使用如下几个矢量化函数

| 函数                | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| `_mm_setzero_si128` | 初始化 `__m128` 寄存器的每个正数为 0                         |
| `_mm_set1_epi32`    | 将 1 个 32 位整型填充到 4 个块中                             |
| `_mm_cmpgt_ps`      | 比较每个 32 位浮点数的大小，如果大于则对应块填 `0xFFFFFFFF`，否则填 0 |
| `_mm_add_epi32`     | 对 `__m128` 变量分块对应相加，每块 32 位                     |
| `_mm_castps_si128`  | 将 `__m128` 类型按位转换为 `__m128i` 类型                    |
| `_mm_blendv_ps`     | 混合两个 `__m128` 类型变量                                   |
| `_mm_castsi128_ps`  | 将 `__m128i` 类型按位转换为 `__m128` 类型                    |
| `_mm_hadd_epi32`    | 对 `__m128` 变量分块水平 horizon 相加，然后组合成 `__m128` 变量 |
| `_mm_cvtsi128_si32` | 将 `__m128` 寄存器的低 32 位转换为整型                       |
| `_mm_movemask_ps`   | 将 4 个 32 位浮点数的符号位保存在低 4 位，其余位设为 0 并返回 |
| `_mm_popcnt_u32`    | 返回 32 位无符号整型的二进制表示中 1 的个数                  |

需要开启 `SSE4.2` 指令集来使用 `hadd` 指令

```cmake
target_compile_options(main PUBLIC "-msse4.2")		# clang-cl gcc
target_compile_options(main PUBLIC "/arch:SSE4.2") 	# MSVC
```



#### 基础样例

给定数组和阈值，统计大于阈值的元素个数

```cpp
constexpr int shift = 20;

void countp(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });
  float y = x.front();

  auto f = [](const float *x, std::size_t n, float y) {
    std::size_t ret = 0;
    for (std::size_t i = 0; i < n; i++) {
      ret += x[i] > y ? 1 : 0;
    }
    return ret;
  };

  for (auto _ : state) {
    auto ret = f(x.data(), n, y);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(countp);
```



#### 矢量化

直接矢量化

```cpp
void countp_ps(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });
  float y = x.front();

  auto f = [](const float *x, std::size_t n, float y) {
    __m128i ret = _mm_setzero_si128();
    __m128 yv = _mm_set1_ps(y);
    __m128i one = _mm_set1_epi32(1);
    __m128i zero = _mm_setzero_si128();
    for (std::size_t i = 0; i < n; i += 4) {
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 mask = _mm_cmpgt_ps(xi, yv);
      ret = _mm_add_epi32(
          ret, _mm_castps_si128(_mm_blendv_ps(_mm_castsi128_ps(zero),
                                              _mm_castsi128_ps(one), mask)));
    }
    ret = _mm_hadd_epi32(ret, ret);
    ret = _mm_hadd_epi32(ret, ret);
    return _mm_cvtsi128_si32(ret);
  };

  for (auto _ : state) {
    auto ret = f(x.data(), n, y);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(countp_ps);
```

首先使用 `_mm_cmpgt_ps` 比较 4 个浮点数，并将结果保存。将 `one, zero` 两个整型 `__m128` 变量转换为浮点变量，然后混合

```cpp
// _mm_blendv_ps(zero, one, mask);
// 从右向左 -> 从低位向高位
// mask  | true 	  | true 	   | false 	    | true 		 |
// mask  | 0xFFFFFFFF | 0xFFFFFFFF | 0x00000000 | 0xFFFFFFFF |
// zero  | 0x00000000 | 0x00000000 | 0x00000000 | 0x00000000 |
// one   | 0x00000001 | 0x00000001 | 0x00000001 | 0x00000001 |
// blend | 0x00000001 | 0x00000001 | 0x00000000 | 0x00000001 |
// mask 为 0 的块选择第一个 __m128 对应的块
// 		为 -1 的块选择第二个 __m128 对应的块
```

使用 `_mm_hadd_epi32` 水平相加混合

```cpp
_mm_hadd_epi32(a, b);
// | a3 | a2 | a1 | a0 |
// | b3 | b2 | b1 | b0 |
// | b3 + b2 | b1 + b0 | a3 + a2 | a1 + a0 |
```

最后获得低 32 位就是计数结果。



#### 统计符号

由于比较结果的返回值 `0x00000000, 0xFFFFFFFF` 符号不同，我们还可以统计这些符号数

```cpp
void countp_ps_pop(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });
  float y = x.front();

  auto f = [](const float *x, std::size_t n, float y) {
    int ret = 0;
    __m128 yv = _mm_set1_ps(y);
    for (std::size_t i = 0; i < n; i += 4) {
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 mask = _mm_cmpgt_ps(xi, yv);
      int m = _mm_movemask_ps(mask);
      ret += _mm_popcnt_u32(m);
    }
    return ret;
  };

  for (auto _ : state) {
    auto ret = f(x.data(), n, y);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(countp_ps_pop);
```

使用 `_mm_movemask_ps` 获得符号位的二进制拼接，然后用 `_mm_popcnt_u32` 获得 1 的个数

```cpp
_mm_movemask_ps(mask);
// 从右向左 -> 从低位向高位
// mask  | true 	  | true 	   | false 	    | true 		 |
// mask  | 0xFFFFFFFF | 0xFFFFFFFF | 0x00000000 | 0xFFFFFFFF |
// m 	 | 0 		  | 0 		   | 0 			| 0x00001101 |
_mm_popcnt_u32(m);	// 3
```



#### 优化混合

注意到 `true` 对应的 `0xFFFFFFFF` 其实就是 `-1`，因此我们实际上可以直接将 `mask` 用于统计数，只不过要将符号反转，变成 `ret - mask`，并利用循环展开优化

```cpp
void countp_ps_8(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });
  float y = x.front();

  auto f = [](const float *x, std::size_t n, float y) {
    __m128i ret = _mm_setzero_si128();
    __m128 yv = _mm_set1_ps(y);
    for (std::size_t i = 0; i < n; i += 8) {
      __m128 xi = _mm_loadu_ps(&x[i]);
      __m128 xi2 = _mm_loadu_ps(&x[i + 4]);
      __m128i mask = _mm_castps_si128(_mm_cmpgt_ps(xi, yv));
      __m128i mask2 = _mm_castps_si128(_mm_cmpgt_ps(xi2, yv));
      mask = _mm_add_epi32(mask, mask2);
      ret = _mm_sub_epi32(ret, mask);
    }
    ret = _mm_hadd_epi32(ret, ret);
    ret = _mm_hadd_epi32(ret, ret);
    return _mm_cvtsi128_si32(ret);
  };

  for (auto _ : state) {
    auto ret = f(x.data(), n, y);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(countp_ps_8);
```



计算耗时

```shell
|| countp           1556147 ns      1339286 ns          560
|| countp_ps         688681 ns       697545 ns         1120
|| countp_ps_pop     694739 ns       697545 ns          896
|| countp_ps_8       602352 ns       599888 ns         1120
```



### 循环查找

我们使用如下几个矢量化函数

| 函数           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| `_mm_or_ps`    | 对 `__m128` 变量分块对应或，每块 32 位                       |
| `_mm_load_ss`  | 将 32 位浮点装载到 `__m128` 变量的低 32 位                   |
| `_mm_cmpgt_ss` | 比较两个 `__m128` 变量的低 32 位，如果大于则对应块填 `0xFFFFFFFF`，否则填 0 |
| `_tzcnt_u32`   | 返回 32 位无符号整型中从低位开始第一个 1 所在的位置          |



#### 基础样例

寻找给定数组中第一个大于阈值的元素的索引

```cpp
constexpr int shift = 20;

void scalar_findp(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });
  float y = 0.999f;

  auto f = [](const float *x, std::size_t n, float y) {
    for (std::size_t i = 0; i < n; i++) {
      if (x[i] > y)
        return i;
    }
    return (std::size_t)-1;
  };

  for (auto _ : state) {
    auto ret = f(x.data(), n, y);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(scalar_findp);
```



#### 矢量化

直接矢量化并循环展开

```cpp
void scalar_findp_ps(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);
  std::generate(x.begin(), x.end(),
                [uni = std::uniform_real_distribution<float>(),
                 rng = std::mt19937()]() mutable { return uni(rng); });
  float y = 0.999f;

  auto f = [](const float *x, std::size_t n, float y) {
    __m128 pred = _mm_set1_ps(y);
    std::size_t i;
    for (i = 0; i + 16 <= n; i += 16) {
      __m128 xi = _mm_loadu_ps(x + i);
      __m128 xi2 = _mm_loadu_ps(x + i + 4);
      __m128 xi3 = _mm_loadu_ps(x + i + 8);
      __m128 xi4 = _mm_loadu_ps(x + i + 12);
      xi = _mm_cmpgt_ps(xi, pred);
      xi2 = _mm_cmpgt_ps(xi2, pred);
      xi3 = _mm_cmpgt_ps(xi3, pred);
      xi4 = _mm_cmpgt_ps(xi4, pred);
        
      // 将所有比较结果合并到一个 __m128 变量中
      xi = _mm_or_ps(xi, xi2);
      xi3 = _mm_or_ps(xi3, xi4);
      xi = _mm_or_ps(xi, xi3);
      
      // 获得合并后符号位的二进制拼接，如果非零，说明找到了
      if (_mm_movemask_ps(xi)) [unlikely](unlikely) {
        // 重新查找一遍
        __m128 xi = _mm_loadu_ps(x + i);
        __m128 xi2 = _mm_loadu_ps(x + i + 4);
        __m128 xi3 = _mm_loadu_ps(x + i + 8);
        __m128 xi4 = _mm_loadu_ps(x + i + 12);
        xi = _mm_cmpgt_ps(xi, pred);
        xi2 = _mm_cmpgt_ps(xi2, pred);
        xi3 = _mm_cmpgt_ps(xi3, pred);
        xi4 = _mm_cmpgt_ps(xi4, pred);

        // 获得符号位的二进制拼接，m 只有低 4 位有效，其余 28 位为 0
        int m = _mm_movemask_ps(xi);
        int m2 = _mm_movemask_ps(xi2);
        int m3 = _mm_movemask_ps(xi3);
        int m4 = _mm_movemask_ps(xi4);

        // 拼接 m4 m3 m2 m 得到 16 位有效值，其余 16 位为 0
        m |= m2 << 4;
        m3 |= m4 << 4;
        m |= m3 << 8;
        // 获得 1 所在的位，然后加上 i 得到实际的位置
        return _tzcnt_u32(m) + i;
      }
    }
    for (; i < n; i++) {
      __m128 xi = _mm_load_ss(x + i);
      __m128 mask = _mm_cmpgt_ss(xi, pred);
      if (_mm_movemask_ps(mask) & 1)
        return i;
    }
    return (std::size_t)-1;
  };

  for (auto _ : state) {
    auto ret = f(x.data(), n, y);
    benchmark::DoNotOptimize(ret);
  }
}
BENCHMARK(scalar_findp_ps);
```

由于只有一次查找有效，我们希望大部分循环能够快速跳过，因此通过 `_mm_or_ps, _mm_movemask_ps` 快速判断是否存在有效值。当存在有效值时才计算位置。



计算耗时

```shell
|| scalar_findp          1204 ns         1004 ns       746667
|| scalar_findp_ps        268 ns          267 ns      2635294
```



### 三角函数

由于我们要使用 `AVX` 指令集，需要指定编译选项

```cmake
target_compile_options(main PUBLIC "--msse4.2 -mavx2 -mfma")	# clang-cl gcc
target_compile_options(main PUBLIC "/arch:AVX2") 				# MSVC
```



#### 基础样例

考虑计算正弦值的样例

```cpp
constexpr int shift = 20;

void fillsin(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);

  auto f = [](float *x, std::size_t n) {
    for (std::size_t i = 0; i < n; i++) {
      x[i] = sin(i);
    }
  };

  for (auto _ : state) {
    f(x.data(), n);
    benchmark::DoNotOptimize(x);
  }
}
BENCHMARK(fillsin);
```



#### 使用单精度

首先可以将 `sin` 替换为 `sinf` 即单精度

```cpp
void fillsinf(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);

  auto f = [](float *x, std::size_t n) {
    for (std::size_t i = 0; i < n; i++) {
      x[i] = sinf(i);
    }
  };

  for (auto _ : state) {
    f(x.data(), n);
    benchmark::DoNotOptimize(x);
  }
}
BENCHMARK(fillsinf);
```



#### 矢量化

首先给出矢量化三角函数计算的函数，是通过泰勒展开到 5 阶近似

```cpp
constexpr float M_PI = 3.14159265358979f;

static __m128 mm128_sincos_ps(__m128 xx, __m128 *cosret) {
  const __m128 DP1F = _mm_set1_ps(0.78515625f * 2.f);
  const __m128 DP2F = _mm_set1_ps(2.4187564849853515625E-4f * 2.f);
  const __m128 DP3F = _mm_set1_ps(3.77489497744594108E-8f * 2.f);
  const __m128 P0sinf = _mm_set1_ps(-1.6666654611E-1f);
  const __m128 P1sinf = _mm_set1_ps(8.3321608736E-3f);
  const __m128 P2sinf = _mm_set1_ps(-1.9515295891E-4f);
  const __m128 P0cosf = _mm_set1_ps(4.166664568298827E-2f);
  const __m128 P1cosf = _mm_set1_ps(-1.388731625493765E-3f);
  const __m128 P2cosf = _mm_set1_ps(2.443315711809948E-5f);

  __m128 xa = _mm_and_ps(xx, _mm_castsi128_ps(_mm_set1_epi32(0x7FFFFFFF)));
  __m128i q = _mm_cvtps_epi32(_mm_mul_ps(xa, _mm_set1_ps(2.f / M_PI)));
  __m128 y = _mm_cvtepi32_ps(q);
  __m128 x = _mm_fnmadd_ps(y, DP3F,
                           _mm_fnmadd_ps(y, DP2F, _mm_fnmadd_ps(y, DP1F, xa)));
  __m128 x2 = _mm_mul_ps(x, x);
  __m128 x3 = _mm_mul_ps(x2, x);
  __m128 x4 = _mm_mul_ps(x2, x2);
  __m128 s = _mm_fmadd_ps(
      x3, _mm_fmadd_ps(x2, P2sinf, _mm_fmadd_ps(x, P1sinf, P0sinf)), x);
  __m128 c = _mm_fmadd_ps(
      x4, _mm_fmadd_ps(x2, P2cosf, _mm_fmadd_ps(x, P1cosf, P0cosf)),
      _mm_fnmadd_ps(_mm_set1_ps(0.5f), x2, _mm_set1_ps(1.0f)));
  __m128 mask = _mm_castsi128_ps(_mm_cmpeq_epi32(
      _mm_setzero_si128(),
      _mm_castps_si128(_mm_and_ps(_mm_castsi128_ps(q),
                                  _mm_castsi128_ps(_mm_set1_epi32(0x1))))));
  __m128 sin1 = _mm_blendv_ps(c, s, mask);
  __m128 cos1 = _mm_blendv_ps(s, c, mask);
  sin1 = _mm_xor_ps(
      sin1, _mm_and_ps(_mm_xor_ps(_mm_castsi128_ps(_mm_slli_epi32(q, 30)), xx),
                       _mm_set1_ps(-0.0f)));
  cos1 = _mm_xor_ps(
      cos1, _mm_castsi128_ps(_mm_slli_epi32(
                _mm_castps_si128(_mm_and_ps(
                    _mm_castsi128_ps(_mm_add_epi32(q, _mm_set1_epi32(1))),
                    _mm_castsi128_ps(_mm_set1_epi32(0x2)))),
                30)));
  *cosret = cos1;
  return sin1;
}

static __m256 mm256_sincos_ps(__m256 xx, __m256 *cosret) {
  const __m256 DP1F = _mm256_set1_ps(0.78515625f * 2.f);
  const __m256 DP2F = _mm256_set1_ps(2.4187564849853515625E-4f * 2.f);
  const __m256 DP3F = _mm256_set1_ps(3.77489497744594108E-8f * 2.f);
  const __m256 P0sinf = _mm256_set1_ps(-1.6666654611E-1f);
  const __m256 P1sinf = _mm256_set1_ps(8.3321608736E-3f);
  const __m256 P2sinf = _mm256_set1_ps(-1.9515295891E-4f);
  const __m256 P0cosf = _mm256_set1_ps(4.166664568298827E-2f);
  const __m256 P1cosf = _mm256_set1_ps(-1.388731625493765E-3f);
  const __m256 P2cosf = _mm256_set1_ps(2.443315711809948E-5f);

  __m256 xa =
      _mm256_and_ps(xx, _mm256_castsi256_ps(_mm256_set1_epi32(0x7FFFFFFF)));
  __m256i q = _mm256_cvtps_epi32(_mm256_mul_ps(xa, _mm256_set1_ps(2.f / M_PI)));
  __m256 y = _mm256_cvtepi32_ps(q);
  __m256 x = _mm256_fnmadd_ps(
      y, DP3F, _mm256_fnmadd_ps(y, DP2F, _mm256_fnmadd_ps(y, DP1F, xa)));
  __m256 x2 = _mm256_mul_ps(x, x);
  __m256 x3 = _mm256_mul_ps(x2, x);
  __m256 x4 = _mm256_mul_ps(x2, x2);
  __m256 s = _mm256_fmadd_ps(
      x3, _mm256_fmadd_ps(x2, P2sinf, _mm256_fmadd_ps(x, P1sinf, P0sinf)), x);
  __m256 c = _mm256_fmadd_ps(
      x4, _mm256_fmadd_ps(x2, P2cosf, _mm256_fmadd_ps(x, P1cosf, P0cosf)),
      _mm256_fnmadd_ps(_mm256_set1_ps(0.5f), x2, _mm256_set1_ps(1.0f)));
  __m256 mask = _mm256_castsi256_ps(
      _mm256_cmpeq_epi32(_mm256_setzero_si256(),
                         _mm256_castps_si256(_mm256_and_ps(
                             _mm256_castsi256_ps(q),
                             _mm256_castsi256_ps(_mm256_set1_epi32(0x1))))));
  __m256 sin1 = _mm256_blendv_ps(c, s, mask);
  __m256 cos1 = _mm256_blendv_ps(s, c, mask);
  sin1 = _mm256_xor_ps(
      sin1,
      _mm256_and_ps(
          _mm256_xor_ps(_mm256_castsi256_ps(_mm256_slli_epi32(q, 30)), xx),
          _mm256_set1_ps(-0.0f)));
  cos1 = _mm256_xor_ps(
      cos1,
      _mm256_castsi256_ps(_mm256_slli_epi32(
          _mm256_castps_si256(_mm256_and_ps(
              _mm256_castsi256_ps(_mm256_add_epi32(q, _mm256_set1_epi32(1))),
              _mm256_castsi256_ps(_mm256_set1_epi32(0x2)))),
          30)));
  *cosret = cos1;
  return sin1;
}
```



然后可以执行矢量化

```cpp
void fillsin_ps(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);

  auto f = [](float *x, std::size_t n) {
    __m128i idx = _mm_set_epi32(0, 1, 2, 3);
    for (std::size_t i = 0; i < n; i += 4) {
      __m128 ii = _mm_cvtps_epi32(idx);
      __m128 cosi;
      __m128 sini = mm128_sincos_ps(ii, &cosi);
      _mm_store_ps(&x[i], sini);
      idx = _mm_add_epi32(idx, _mm_set1_epi32(4));
    }
  };

  for (auto _ : state) {
    f(x.data(), n);
    benchmark::DoNotOptimize(x);
  }
}
BENCHMARK(fillsin_ps);

void fillsin_ps_256(benchmark::State &state) {
  auto n = std::size_t(1 << shift);
  std::vector<float> x(n);

  auto f = [](float *x, std::size_t n) {
    __m256i idx = _mm256_set_epi32(0, 1, 2, 3, 4, 5, 6, 7);
    for (std::size_t i = 0; i < n; i += 8) {
      __m256 ii = _mm256_cvtps_epi32(idx);
      __m256 cosi;
      __m256 sini = mm256_sincos_ps(ii, &cosi);
      _mm256_store_ps(&x[i], sini);
      idx = _mm256_add_epi32(idx, _mm256_set1_epi32(8));
    }
  };

  for (auto _ : state) {
    f(x.data(), n);
    benchmark::DoNotOptimize(x);
  }
}
BENCHMARK(fillsin_ps_256);
```



计算耗时

```shell
|| fillsin          35744080 ns     30625000 ns           25
|| fillsinf         25843844 ns     25390625 ns           32
|| fillsin_ps        3223343 ns      3227700 ns          213
|| fillsin_ps_256    1690115 ns      1689189 ns          407
```



## MKL

### 基本介绍

MKL 是 Intel Math Kernel Library（英特尔数学核心库）的缩写。它是由英特尔开发的一套高度优化的数学函数库，主要用于科学计算、工程计算、金融建模和其他高性能计算应用中。MKL 提供了一系列用于线性代数、傅里叶变换、矢量数学、随机数生成等数学计算的函数，可以显著提高基于英特尔处理器的应用程序的性能。

```embed
title: "Accelerate Fast Math with Intel® oneAPI Math Kernel Library"
image: "https://www.intel.com/content/dam/develop/public/us/en/images/thumbnails/tool-thumbnail-beta-oneapi-logo.jpg"
description: "Use this library of math routines for compute-intensive tasks: linear algebra, FFT, RNG. Optimized for high-performance computing and data science."
url: "https://www.intel.com/content/www/us/en/developer/tools/oneapi/onemkl.html"
```

在官网下载 Stand-Alone 版本，启动安装器后直接使用默认的安装配置。



### Eigen

从 Eigen 版本 3.1 及更高版本开始，用户可以通过安装英特尔 MKL 10.3（或更高版本）副本从内置的英特尔®数学核心函数库（MKL）优化中受益。

```embed
title: "Eigen: Using Intel® MKL from Eigen"
image: "https://eigen.tuxfamily.org/dox/Eigen_Silly_Professor_64x64.png"
description: "Since Eigen version 3.1 and later, users can benefit from built-in Intel® Math Kernel Library (MKL) optimizations with an installed copy of Intel MKL 10.3 (or later)."
url: "https://eigen.tuxfamily.org/dox/TopicUsingIntelMKL.html"
```

可添加的预定义宏包括

| 预定义宏                       | 作用                                                              |
| -------------------------- | --------------------------------------------------------------- |
| `EIGEN_USE_BLAS`           | 支持使用外部 BLAS 2 级和 3 级例程                                          |
| `EIGEN_USE_LAPACKE`        | 允许通过 Lapack 的 Lapacke C 接口使用外部 Lapack 例程                        |
| `EIGEN_USE_LAPACKE_STRICT` | 与 `EIGEN_USE_LAPACKE` 相同，但禁用了稳健性较低的算法。                          |
| `EIGEN_USE_MKL_VML`        | 支持使用 Intel VML（矢量运算）                                            |
| `EIGEN_USE_MKL_ALL`        | 定义 `EIGEN_USE_BLAS` 、 `EIGEN_USE_LAPACKE` 和 `EIGEN_USE_MKL_VML` |

通常可以直接添加宏

```cpp
#define EIGEN_USE_MKL_ALL
#define EIGEN_VECTORIZE_SSE4_2
```



### 测试样例

给出下面的测试代码

```cpp
#define EIGEN_USE_MKL_ALL
#define EIGEN_VECTORIZE_SSE4_2

#include <Dense>
#include <Core>
#include <iostream>
#include <chrono>

using namespace Eigen;
using namespace std;
typedef MatrixXf mat;
typedef VectorXf vec;

constexpr int d_in = 20;
constexpr int d_inter = 100;
constexpr int d_out = 30;
constexpr int num_round = 1000;

vec prelu(vec in, const vec &a){
    for (int i=0; i<in.size(); i++){
        if (in[i] < 0)
            in[i] = in[i] * a[i];
    }
    return in;
}

double one_run(){
    // eigen has no normal distributed initialization. approximate it with uniform distribution
    // weights
    mat w1 = mat::Random(d_inter, d_in);
    mat w2 = mat::Random(d_inter, d_inter);
    mat w3 = mat::Random(d_inter, d_inter);
    mat w4 = mat::Random(d_inter, d_inter);
    mat w5 = mat::Random(d_inter, d_inter);
    mat w6 = mat::Random(d_inter, d_inter);
    mat w7 = mat::Random(d_out, d_inter);

    // bias
    vec b1 = vec::Random(d_inter);
    vec b2 = vec::Random(d_inter);
    vec b3 = vec::Random(d_inter);
    vec b4 = vec::Random(d_inter);
    vec b5 = vec::Random(d_inter);
    vec b6 = vec::Random(d_inter);
    vec b7 = vec::Random(d_out);

    // param for prelu
    vec a1 = vec::Random(d_inter);
    vec a2 = vec::Random(d_inter);
    vec a3 = vec::Random(d_inter);
    vec a4 = vec::Random(d_inter);
    vec a5 = vec::Random(d_inter);
    vec a6 = vec::Random(d_inter);

    auto t_start = std::chrono::high_resolution_clock::now();

    // random input
    vec input = vec::Random(d_in);

    // forward
    vec result;

    result = prelu(w1 * input + b1, a1);
    result = prelu(w2 * result + b2, a2);
    result = prelu(w3 * result + b3, a3);
    result = prelu(w4 * result + b4, a4);
    result = prelu(w5 * result + b5, a5);
    result = prelu(w6 * result + b6, a6);
    result = (w7 * result + b7).eval();         // force evaluation. just in case. 
    
    auto t_end = std::chrono::high_resolution_clock::now();
    double elapsed_time_us = std::chrono::duration<double, std::micro>(t_end-t_start).count();

    return elapsed_time_us;
}


int main(){
    VectorXd all = VectorXd::Random(num_round);

    for (int i=0; i< num_round; ++i){
        all[i] = one_run();
    }

    cout << "time in micro second" << endl;
    cout << "mean: " << all.mean() << endl;
    cout << "max: " << all.maxCoeff() << endl;
    cout << "min: " << all.minCoeff() << endl;
    
    VectorXd err = all - VectorXd::Constant(num_round, all.mean());
    err = err.array() * err.array();
    float std = err.mean();
    std = sqrt(std);
    cout << "std: " << std << endl;

    return 0;
}
```

项目 cmake 配置为

```cmake
cmake_minimum_required(VERSION 3.10)
project(main)

set(Eigen3_DIR "D:/lib/Eigen3/share/eigen3/cmake")
find_package(Eigen3 REQUIRED)
find_package(MKL REQUIRED)

add_executable(main main.cpp)

target_link_libraries(main PUBLIC MKL::MKL)
target_include_directories(main PUBLIC ${EIGEN3_INCLUDE_DIR})
```

对比未使用 MKL 加速和使用 MKL 加速的计算耗时

```shell
time in micro second
mean: 224.859
max: 409.3
min: 204.5
std: 21.751

time in micro second
mean: 72.4084
max: 4139
min: 53.4
std: 129.743
```

其中 `mean, max, min` 分别表示平均耗时、最大耗时和最小耗时。

