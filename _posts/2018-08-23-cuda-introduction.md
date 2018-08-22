---
layout:     post
title:      Introduction to CUDA
summary:    Post aimed at those who want to learn what CUDA is and how it works.
date:       2018-08-22 15-23-37
categories: CUDA
comments: true
---

First of all, CUDA is an extension of the C programming language born in 2007. It allows us to execute parallel
programs in our NVIDIA GPU. Recent CPU's usually have (at most) 8 cores with which we can write multithreading
approaches to our solutions. Nevertheless, although you can start a thousand threads in a Java or C++ application
the real parallelism will always be limited by the number of cores we have in our CPU, which at most will be 8, as
we stated before. For example, as is shown is the following C snippet, many threads can be created (in this case 100)
but only 8 of them can run at the same time:

```c
#include <stdio.h>
#include <pthread.h>

#define N_THREADS 100

void *doSomething(void *vargp)
{
 sleep(1);
 printf("Hi from Thread! \n");
 return NULL;
}

int main()
{
 size_t i, j;
 pthread_t threads_id[N_THREADS];

 for(i = 0; i < N_THREADS; i++)
 {
  pthread_create(&threads_id[i], NULL, doSomething, NULL);
 }
 for(j = 0; j < N_THREADS; j++)
 {
  pthread_join(threads_id[j], NULL);
 }
 return 0;
}
```

The question that should be asked now, is: **If our real parallelism is limited by our CPU, how can we run more threads
than the number of cores it has?**. The cool thing about GPUs is that they have hundreds or thousands of cores,
depending on the model of the GPU, so more massively parallel approaches can be taken into account. Sounds good, but how
do we get started with CUDA then?

The first thing that should be taken into account is that scripts that run CUDA programs have their own extension.
C programs use the `.c` extension, C++ programs use the '.cpp' extension and CUDA programs use **.cu** as an extension.
Before we write an introductory snippet int CUDA, whe should define some methods and keywords we need to use:

* `__global__` -> Keyword used to indicate that a method will be ran in the GPU, we usually call them kernels.
* `cudaMalloc((void**) &pointer, size)` -> Same as malloc in C, but for the GPU. Allocates space.
* `cudaFree(pointer)` -> Same as free in C, but for the GPU.
* `cudaMemCopy(dst_pointer, src_pointer, size, direction)` -> Copies inputs to the device (GPU).

The typical example used to introduce people to CUDA is the addition of two vectors _A_ and _B_ and storing them in a
different vector called _result_. The sequential algorithm is as follows:

```c
#include<stdio.h>
#define SIZE 10

void random_ints(int* array)
{
 int i;
 for(i = 0; i < SIZE; i++)
  array[i] = rand() % 100;
}

int main() {

 int A[SIZE];
 int B[SIZE];
 int result[SIZE];

 random_ints(A); random_ints(B);

 for(int i = 0; i < SIZE; i++) {
  result[i] = A[i] + B[i];
 }

 printf("A    B    result\n");
 for(int i = 0; i < SIZE; i++) {
  printf("%d + %d = %d\n", A[i], B[i], result[i]);
 }

 return 0;
}
```

The output of this code is something like:

```
    A    B    result

    83 + 62 = 145
    86 + 27 = 113
    77 + 90 = 167
    15 + 59 = 74
    93 + 63 = 156
    35 + 26 = 61
    86 + 40 = 126
    92 + 26 = 118
    49 + 72 = 121
    21 + 36 = 57
```

This algorithm runs quite fast because the vectors only contain 10 elements. The program would go really slow if
the vectors holded something like one milion values or so.
Now we should ask ourselves: **Is there a parallel approach for this algorithm?**. The answer happens to be yes!
Below is shown a CUDA program to do the same tasks of adding two different vectors. The steps we make are commented as
we write the code:

```c
#include <stdio.h>
#define N (2048*2048)
#define THREADS_IN_BLOCK 512

void random_ints(int* array, int size);

// The __global__ tag indicates that this method will be ran by each thread of the GPU.
__global__ void add(int* a, int* b, int* result, int arraySize)
{
 int index = threadIdx.x + blockIdx.x * blockDim.x;
 if (index < arraySize)
  result[index] = a[index] + b[index];
}

int main(void)
{
 int *a, *b, *result; // host vectors
 int *d_a, *d_b, *d_result; // copies of host vectors for the device
 int size = N * sizeof(int);

 // Allocating space for the vectors in the device.
 cudaMalloc((void **) &d_a, size);
 cudaMalloc((void **) &d_b, size);
 cudaMalloc((void **) &d_result, size);

 // Allocating memory for host vectors.
 a = (int *) malloc(size); random_ints(a, N);
 b = (int *) malloc(size); random_ints(b, N);
 result = (int *) malloc(size);

 // Copying values from host to device. We write in d_a the values of a. Same with d_b
 cudaMemcpy(d_a, a, size, cudaMemcpyHostToDevice);
 cudaMemcpy(d_b, b, size, cudaMemcpyHostToDevice);

// We call the kernel
 add<<<N/THREADS_IN_BLOCK, THREADS_IN_BLOCK>>>(d_a, d_b, d_result, N);

 // Retrieving result in host variable 'result'.
 cudaMemcpy(result, d_result, size, cudaMemcpyDeviceToHost);

 // We free the memory both in the host and in the device
 free(a); free(b); free(result);
 cudaFree(d_a); cudaFree(d_b); cudaFree(d_result);

 return 0;
}
```

CUDA has much more options, methods and macros which I will probably cover on another post in the future. As stated in
the beginning, the goal of this post was to introduce CUDA to C programmers.

Keep in mind that nowadays, with the rise of machine learning and data science, CUDA is vastly used in many technologies
and frameworks such as `Tensorflow`. Convolutional Neural Networks (CNN) are highly parallelizable and CUDA is vastly used
for this discipline, specially a CUDA library called cuDNN.