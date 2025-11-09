## 0. xv6 book
我们可以使用懒分配来节省空间资源和时间资源，只有在进程需要分配内存的时候才通过page fault去disk取页面

## 1. Eliminate allocation from sbrk() `(easy)`

just remove growproc

## 2. Lazy allocation (moderate)


