# GEMM·CUDA
这一题我感觉其实就是CPU版本的改编，思想几乎完全一样，只是需要用CUDA来写  

首先想法还是和CPU是一致的，就是每个thread计算C中的一个元素，因为C中一个元素需要A的一行和B的一列来计算，所以我们分配给每个CUDA thread的任务就是用A的一行每个元素乘以B的一列的每个元素，然后累加，乘以alpha，这个是前半部分；而后半部分只需要把原数字乘以beta即可。

所以思路是非常简单的。只是需要知道点CUDA的语法。我们知道CUDA的一个线程负责一个数字，我们需要知道现在这个线程负责的数字的位置。所以计算当前数字的row和col。row的计算只需要blockDim.y * blockIdx.y + threadIdx.y即可，col的计算只需要blockDim.x * blockIdx.x + threadIdx.x即可。

然后我们判断一下边界，也就是，超出(M, N)范围的我们不用算，直接return即可  

接下来很简单了，跟CPU的差不多，就是计算一行乘以一列。这道题还需要用到两个宏，就是half和float转换的宏，一个是**__half2float**一个是**__float2half**。

然后在调用kernel的时候还需要传入grid和block，我让block取一个(16, 16)，然后grid就根据block来计算就行，向上取整。

所以最终的代码就是  
```
#include <cuda_fp16.h>
#include <cuda_runtime.h>

__global__ void kernel(const half *A, const half *B, half *C, int M, int N, int K, float alpha, float beta) {

    // 根据block和thread找到现在要计算的row和col
    int row = blockDim.y * blockIdx.y + threadIdx.y;
    int col = blockDim.x * blockIdx.x + threadIdx.x;

    // 然后还需要判断一下row, col在不在范围内
    if(row >= M || col >= N) {
        return;
    }
    
    // 也就是我们要算的是C[row][col]
    float total = 0;

    for(int x = 0;x < K;x++) {
        total += __half2float(A[row * K + x]) * __half2float(B[x * N + col]);
    }

    C[row * N + col] *= beta;
    // 这里得转化一下half和float
    C[row * N + col] += __float2half(alpha * total);
}


extern "C" void solve(const half* A, const half* B, half* C, int M, int N, int K, float alpha,
                      float beta) {
    // 把这里改成你的实现
    dim3 block(16, 16);
    // 向上取整
    dim3 grid((N + block.x - 1) / block.x, (M + block.y - 1) / block.y);
    kernel<<< grid, block>>>(A, B, C, M, N, K, alpha, beta);
}
```

测试一下  
```
nscc@login:~$ nscc-run bash run_tests.sh
execution d055742d-06e1-4bab-a0cb-8acb20ed3340
  ok   example_2x2x3: m=2 n=2 k=3 time_ms=0.096
  ok   unaligned_37x41x29: m=37 n=41 k=29 time_ms=0.013
  ok   square_16_beta0: m=16 n=16 k=16 time_ms=0.008
  ok   square_16_beta1: m=16 n=16 k=16 time_ms=0.007
  ok   tall_32x16: m=32 n=16 k=16 time_ms=0.008
  ok   wide_16x32: m=16 n=32 k=16 time_ms=0.007
  ok   alpha0_beta1: m=16 n=16 k=32 time_ms=0.009
  ok   perf_1024: m=1024 n=1024 k=1024 time_ms=1.217
summary cases=8 failures=0 worst_case_ms=1.217
cuda-gemm-fp16_RESULT: PASS
WARNING: Overriding HOME environment variable with APPTAINERENV_HOME is not permitted
execution d055742d-06e1-4bab-a0cb-8acb20ed3340: completed
```

没有问题，当然这题也可以像CPU那题进行一点优化，也就是利用Locality来提高Cache Hit Rate，不过上次感觉提升不大，所以也就没有再尝试了。