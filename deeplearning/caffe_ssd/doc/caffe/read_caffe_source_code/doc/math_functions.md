# `math_functions` —— `caffe` 中的代数计算
一、`cuda`的设计技巧

在`common.hpp`中用到一个宏定义`CUDA_KERNEL_LOOP`      
```cpp
/*
* CUDA: grid stride looping
*   caffe采取的线程格和线程块的维数设计
*   (blockDim.x* gridDim.x) 表示的是该线程格所有线程的数量
*   (n) 表示核函数总共要处理的元素个数
*/

#define CUDA_KERNEL_LOOP(i, n) \
  for (int i = blockIdx.x * blockDim.x + threadIdx.x; \
       i < (n); \
       i += blockDim.x * gridDim.x)
```
先看看caffe采取的线程格和线程块的维数设计，还是从common.hpp可以看到     
```
CAFFE_CUDA_NUM_THREADS
CAFFE_GET_BLOCKS(constintN)
```
明显都是一维的。    

整理一下`CUDA_KERNEL_LOOP`格式看看，    

`blockDim.x * gridDim.x`表示的是该线程格所有线程的数量。   
`n`表示核函数总共要处理的元素个数。   
有时候，`n`会大于`blockDim.x * gridDim.x`，因此并不能一个线程处理一个元素。   
由此通过上面的方法，让一个线程串行（`for循环`）处理几个元素。   
再来看一下这个核函数的实现。         
```cpp
template <typename Dtype>
__global__ void mul_kernel(const int n, const Dtype* a,
    const Dtype* b, Dtype* y) {
  CUDA_KERNEL_LOOP(index, n) {
    y[index] = a[index] * b[index];
  }
}
```
就是算两个向量的点积了。由于向量的维数可能大于该`kernel函数`线程格的总线程数量。   

二、`caffe`中使用的函数
  这些函数的实现是在 `caffe/src/caffe/util/math_functions.cpp`中实现的。    
```cpp
#include <boost/math/special_functions/next.hpp>
#include <boost/random.hpp>
#include <limits>
#include "caffe/common.hpp"
#include "caffe/util/math_functions.hpp"
#include "caffe/util/rng.hpp"
namespace caffe {
/*计算矩阵乘法的函数之一是 cblas_sgemm，使用单精度实数，另外还有对应双精度实数，
单精度复数和双精度复数的函数。在此以 cblas_sgemm为例。
函数定义为：
void cblas_sgemm(const enum CBLAS_ORDER Order, const enum CBLAS_TRANSPOSE TransA,
const enum CBLAS_TRANSPOSE TransB, const int M, const int N,
const int K, const float alpha, const float  *A,
const int lda, const float  *B, const int ldb,
const float beta, float  *C, const int ldc)
得到的结果是:
C = alpha*op( A )*op( B ) + beta*C
const enum CBLAS_ORDER Order，是数据的存储形式，在CBLAS的函数中无论一维还是二维数据
都是用一维数组存储，这就要涉及是行主序还是列主序，在C语言中数组是用 行主序，fortran中是列
主序。我还是习惯于是用行主序，所以这个参数是用CblasRowMajor，如果是列主序的话就是 CblasColMajor。
const int M，矩阵A的行，矩阵C的行
const int N，矩阵B的列，矩阵C的列
const int K，矩阵A的列，矩阵B的行
const float alpha， const float beta，计算公式中的两个参数值，如果只是计算C=A*B，则alpha=1,beta=0
const float  *A， const float  *B， const float  *C，矩阵ABC的数据
const int lda， const int ldb， const int ldc，在BLAS的文档里，这三个参数分别为ABC的行数，
但是实际使用发现，在CBLAS里应该是列数。
*/
template<>
void caffe_cpu_gemm<float>(const CBLAS_TRANSPOSE TransA,
    const CBLAS_TRANSPOSE TransB, const int M, const int N, const int K,
    const float alpha, const float* A, const float* B, const float beta,
    float* C) {
  int lda = (TransA == CblasNoTrans) ? K : M;
  int ldb = (TransB == CblasNoTrans) ? N : K;
  cblas_sgemm(CblasRowMajor, TransA, TransB, M, N, K, alpha, A, lda, B,
      ldb, beta, C, N);
}
template<>
void caffe_cpu_gemm<double>(const CBLAS_TRANSPOSE TransA,
    const CBLAS_TRANSPOSE TransB, const int M, const int N, const int K,
    const double alpha, const double* A, const double* B, const double beta,
    double* C) {
  int lda = (TransA == CblasNoTrans) ? K : M;
  int ldb = (TransB == CblasNoTrans) ? N : K;
  cblas_dgemm(CblasRowMajor, TransA, TransB, M, N, K, alpha, A, lda, B,
      ldb, beta, C, N);
}
/*
功能： y=alpha*A*x+beta*y
其中x和y是向量，A 是矩阵
M：A 的行数
N：A 的列数
cblas_sgemv 中的参数1 表示对x和y的每个元素都进行操作
*/  
template <>
void caffe_cpu_gemv<float>(const CBLAS_TRANSPOSE TransA, const int M,
    const int N, const float alpha, const float* A, const float* x,
    const float beta, float* y) {
  cblas_sgemv(CblasRowMajor, TransA, M, N, alpha, A, N, x, 1, beta, y, 1);
}
template <>
void caffe_cpu_gemv<double>(const CBLAS_TRANSPOSE TransA, const int M,
    const int N, const double alpha, const double* A, const double* x,
    const double beta, double* y) {
  cblas_dgemv(CblasRowMajor, TransA, M, N, alpha, A, N, x, 1, beta, y, 1);
}
/*
功能： Y=alpha*X+Y
N：为X和Y中element的个数
*/  
template <>
void caffe_axpy<float>(const int N, const float alpha, const float* X,
    float* Y) { cblas_saxpy(N, alpha, X, 1, Y, 1); }
template <>
void caffe_axpy<double>(const int N, const double alpha, const double* X,
    double* Y) { cblas_daxpy(N, alpha, X, 1, Y, 1); }
/*
功能：用常数 alpha 对 Y 进行初始化
使用memset函数来初始化数组或者结构体比其他初始化方法更快一点
*/  
template <typename Dtype>
void caffe_set(const int N, const Dtype alpha, Dtype* Y) {
  if (alpha == 0) {
    memset(Y, 0, sizeof(Dtype) * N);  // NOLINT(caffe/alt_fn)
    return;
  }
  for (int i = 0; i < N; ++i) {
    Y[i] = alpha;
  }
}
template void caffe_set<int>(const int N, const int alpha, int* Y);
template void caffe_set<float>(const int N, const float alpha, float* Y);
template void caffe_set<double>(const int N, const double alpha, double* Y);
//功能： 给 Y 的每个 element 加上常数 alpha
template <>
void caffe_add_scalar(const int N, const float alpha, float* Y) {
  for (int i = 0; i < N; ++i) {
    Y[i] += alpha;
  }
}
template <>
void caffe_add_scalar(const int N, const double alpha, double* Y) {
  for (int i = 0; i < N; ++i) {
    Y[i] += alpha;
  }
}
/* 功能：拷贝初始化
memcpy()用来拷贝src所指的内存内容前n个字节到dest所指的内存地址上。与strcpy()不同的是,memcpy()会完整的复制n个字节,不会因为遇到字符串结束'\0'而结束
*/
template <typename Dtype>
void caffe_copy(const int N, const Dtype* X, Dtype* Y) {
  if (X != Y) {
    if (Caffe::mode() == Caffe::GPU) {
#ifndef CPU_ONLY
      // NOLINT_NEXT_LINE(caffe/alt_fn)
      CUDA_CHECK(cudaMemcpy(Y, X, sizeof(Dtype) * N, cudaMemcpyDefault));
#else
      NO_GPU;
#endif
    } else {
      memcpy(Y, X, sizeof(Dtype) * N);  // NOLINT(caffe/alt_fn)
    }
  }
}
template void caffe_copy<int>(const int N, const int* X, int* Y);
template void caffe_copy<unsigned int>(const int N, const unsigned int* X,
    unsigned int* Y);
template void caffe_copy<float>(const int N, const float* X, float* Y);
template void caffe_copy<double>(const int N, const double* X, double* Y);
/*
功能：X = alpha*X
N： X中element的个数
*/
template <>
void caffe_scal<float>(const int N, const float alpha, float *X) {
  cblas_sscal(N, alpha, X, 1);
}
template <>
void caffe_scal<double>(const int N, const double alpha, double *X) {
  cblas_dscal(N, alpha, X, 1);
}
/*
功能：Y= alpha*X+beta*Y
*/
template <>
void caffe_cpu_axpby<float>(const int N, const float alpha, const float* X,
                            const float beta, float* Y) {
  cblas_saxpby(N, alpha, X, 1, beta, Y, 1);
}
template <>
void caffe_cpu_axpby<double>(const int N, const double alpha, const double* X,
                             const double beta, double* Y) {
  cblas_daxpby(N, alpha, X, 1, beta, Y, 1);
}
/*
功能：这四个函数分别实现element-wise的加减乘除（y[i] = a[i] + - * \ b[i]）
*/
template <>
void caffe_add<float>(const int n, const float* a, const float* b,
    float* y) {
  vsAdd(n, a, b, y);
}
template <>
void caffe_add<double>(const int n, const double* a, const double* b,
    double* y) {
  vdAdd(n, a, b, y);
}
template <>
void caffe_sub<float>(const int n, const float* a, const float* b,
    float* y) {
  vsSub(n, a, b, y);
}
template <>
void caffe_sub<double>(const int n, const double* a, const double* b,
    double* y) {
  vdSub(n, a, b, y);
}
template <>
void caffe_mul<float>(const int n, const float* a, const float* b,
    float* y) {
  vsMul(n, a, b, y);
}
template <>
void caffe_mul<double>(const int n, const double* a, const double* b,
    double* y) {
  vdMul(n, a, b, y);
}
template <>
void caffe_div<float>(const int n, const float* a, const float* b,
    float* y) {
  vsDiv(n, a, b, y);
}
template <>
void caffe_div<double>(const int n, const double* a, const double* b,
    double* y) {
  vdDiv(n, a, b, y);
}
/*
功能 : 同样是element-wise操作
分别是y[i] = a[i] ^ b，y[i] = a[i]^2，y[i] = sqrt(a[i])，y[i] = exp(a[i] )，y[i] = |a[i] |
*/  
template <>
void caffe_powx<float>(const int n, const float* a, const float b,
    float* y) {
  vsPowx(n, a, b, y);
}
template <>
void caffe_powx<double>(const int n, const double* a, const double b,
    double* y) {
  vdPowx(n, a, b, y);
}
template <>
void caffe_sqr<float>(const int n, const float* a, float* y) {
  vsSqr(n, a, y);
}
template <>
void caffe_sqr<double>(const int n, const double* a, double* y) {
  vdSqr(n, a, y);
}
template <>
void caffe_sqrt<float>(const int n, const float* a, float* y) {
  vsSqrt(n, a, y);
}
template <>
void caffe_sqrt<double>(const int n, const double* a, double* y) {
  vdSqrt(n, a, y);
}
template <>
void caffe_exp<float>(const int n, const float* a, float* y) {
  vsExp(n, a, y);
}
template <>
void caffe_exp<double>(const int n, const double* a, double* y) {
  vdExp(n, a, y);
}
template <>
void caffe_log<float>(const int n, const float* a, float* y) {
  vsLn(n, a, y);
}
template <>
void caffe_log<double>(const int n, const double* a, double* y) {
  vdLn(n, a, y);
}
template <>
void caffe_abs<float>(const int n, const float* a, float* y) {
    vsAbs(n, a, y);
}
template <>
void caffe_abs<double>(const int n, const double* a, double* y) {
    vdAbs(n, a, y);
}
/*
功能：返回一个随机数
*/  
unsigned int caffe_rng_rand() {
  return (*caffe_rng())();
}
/*
功能 ： 返回 b 最大方向上可以表示的最接近的数值。
*/
template <typename Dtype>
Dtype caffe_nextafter(const Dtype b) {
  return boost::math::nextafter<Dtype>(
      b, std::numeric_limits<Dtype>::max());
}
template
float caffe_nextafter(const float b);
template
double caffe_nextafter(const double b);
/*
功能 ：返回符合均匀分布的数据。
*/
template <typename Dtype>
void caffe_rng_uniform(const int n, const Dtype a, const Dtype b, Dtype* r) {
  CHECK_GE(n, 0);
  CHECK(r);
  CHECK_LE(a, b);
  boost::uniform_real<Dtype> random_distribution(a, caffe_nextafter<Dtype>(b));
  boost::variate_generator<caffe::rng_t*, boost::uniform_real<Dtype> >
      variate_generator(caffe_rng(), random_distribution);
  for (int i = 0; i < n; ++i) {
    r[i] = variate_generator();
  }
}
template
void caffe_rng_uniform<float>(const int n, const float a, const float b,
                              float* r);
template
void caffe_rng_uniform<double>(const int n, const double a, const double b,
                               double* r);
/*
功能 ：返回符合高斯分布(正态分布)的数据。
*/
template <typename Dtype>
void caffe_rng_gaussian(const int n, const Dtype a,
                        const Dtype sigma, Dtype* r) {
  CHECK_GE(n, 0);
  CHECK(r);
  CHECK_GT(sigma, 0);
  boost::normal_distribution<Dtype> random_distribution(a, sigma);
  boost::variate_generator<caffe::rng_t*, boost::normal_distribution<Dtype> >
      variate_generator(caffe_rng(), random_distribution);
  for (int i = 0; i < n; ++i) {
    r[i] = variate_generator();
  }
}
template
void caffe_rng_gaussian<float>(const int n, const float mu,
                               const float sigma, float* r);
template
void caffe_rng_gaussian<double>(const int n, const double mu,
                                const double sigma, double* r);
/*
功能 ：返回符合伯努利分布的数据。
       伯努利分布是一个离散型几率分布，是二项分布的特殊情况
*/
template <typename Dtype>
void caffe_rng_bernoulli(const int n, const Dtype p, int* r) {
  CHECK_GE(n, 0);
  CHECK(r);
  CHECK_GE(p, 0);
  CHECK_LE(p, 1);
  boost::bernoulli_distribution<Dtype> random_distribution(p);
  boost::variate_generator<caffe::rng_t*, boost::bernoulli_distribution<Dtype> >
      variate_generator(caffe_rng(), random_distribution);
  for (int i = 0; i < n; ++i) {
    r[i] = variate_generator();
  }
}
template
void caffe_rng_bernoulli<double>(const int n, const double p, int* r);
template
void caffe_rng_bernoulli<float>(const int n, const float p, int* r);
template <typename Dtype>
void caffe_rng_bernoulli(const int n, const Dtype p, unsigned int* r) {
  CHECK_GE(n, 0);
  CHECK(r);
  CHECK_GE(p, 0);
  CHECK_LE(p, 1);
  boost::bernoulli_distribution<Dtype> random_distribution(p);
  boost::variate_generator<caffe::rng_t*, boost::bernoulli_distribution<Dtype> >
      variate_generator(caffe_rng(), random_distribution);
  for (int i = 0; i < n; ++i) {
    r[i] = static_cast<unsigned int>(variate_generator());
  }
}
template
void caffe_rng_bernoulli<double>(const int n, const double p, unsigned int* r);
template
void caffe_rng_bernoulli<float>(const int n, const float p, unsigned int* r);
/*
功能： 返回 vector X 和 vector Y 的内积。
incx， incy ： 步长，即每隔incx 或 incy 个element 进行操作。
*/  
template <>
float caffe_cpu_strided_dot<float>(const int n, const float* x, const int incx,
    const float* y, const int incy) {
  return cblas_sdot(n, x, incx, y, incy);
}
template <>
double caffe_cpu_strided_dot<double>(const int n, const double* x,
    const int incx, const double* y, const int incy) {
  return cblas_ddot(n, x, incx, y, incy);
}
template <typename Dtype>
Dtype caffe_cpu_dot(const int n, const Dtype* x, const Dtype* y) {
  return caffe_cpu_strided_dot(n, x, 1, y, 1);
}
template
float caffe_cpu_dot<float>(const int n, const float* x, const float* y);
template
double caffe_cpu_dot<double>(const int n, const double* x, const double* y);
/*
功能：计算 vector x 的所有element的绝对值之和。
*/  
template <>
float caffe_cpu_asum<float>(const int n, const float* x) {
  return cblas_sasum(n, x, 1);
}
template <>
double caffe_cpu_asum<double>(const int n, const double* x) {
  return cblas_dasum(n, x, 1);
}
template <>
void caffe_cpu_scale<float>(const int n, const float alpha, const float *x,
                            float* y) {
  cblas_scopy(n, x, 1, y, 1);
  cblas_sscal(n, alpha, y, 1);
}
template <>
void caffe_cpu_scale<double>(const int n, const double alpha, const double *x,
                             double* y) {
  cblas_dcopy(n, x, 1, y, 1);
  cblas_dscal(n, alpha, y, 1);
}
}  // namespace caffe

