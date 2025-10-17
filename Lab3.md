# Lab3: page tables

## 0. xv6 book

`page table`，也就是页表，它用来存储`virtual memory`到 `physical memory`的映射，在RISC-V中，它以一种三级页表的形式存在，在cs61c中，我们了解到了TLB，就是类似于cache缓存的东西，但是不同的是，cache用于完成CPU到DRMA（实际上就是memory）之间的快速写存，而TLB则是完成快速查找映射，实际上CPU上传出的地址都是虚拟地址，我们都需要经过TLB（然后是页表）的查找转换成物理地址。

## 1. Print page table(`easy`)
```c
void
vmprint_helper(pagetable_t pagetable, int level) {
  if (level >= 3) {
    return;
  }
  for (int i = 0; i < 512; ++i) {
    pte_t pte = pagetable[i];
    if (pte & PTE_V) {
      for (int j = level; j >=0; --j ) {
        printf("..");
        j ? printf(" ") : 0;
      }
      printf("%d: pte %p pa %p\n", i, pte, PTE2PA(pte));
      if ((pte & (PTE_R | PTE_W | PTE_X)) == 0) {
        vmprint_helper((pagetable_t)PTE2PA(pte), level + 1);
      }
    }
  }
}

void
vmprint(pagetable_t pagetable) {
  printf("page table %p\n", pagetable);
  vmprint_helper(pagetable, 0);
}
```

为什么要pagetable[i]？非常简单，每次给你的pagetable都是起始地址，而每个pagetable都分配了 `2^9`也就是512个pte。然后依次类推，三级页表都是这么设计的。

## 2. A kernel page table per process(hard)

Hint: Don't forget to add vmprint declaration in kernel/defs.h and add if(p->pid == 1) ... in kernel/exec.c;
