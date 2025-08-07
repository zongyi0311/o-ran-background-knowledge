[CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)

**test example**
```
#include <stdio.h>

__global__ void helloFromGPU() {
    printf("Hello from GPU!\n");
}

int main() {
    // 啟動 1 個 block，1 個 thread 的 GPU 核心
    helloFromGPU<<<1, 1>>>();
    // 等待 GPU 執行完
    cudaDeviceSynchronize();
    return 0;
}
```
**result**

<img width="725" height="113" alt="image" src="https://github.com/user-attachments/assets/902ce700-ff00-4864-884a-e3a936c9fd92" />
