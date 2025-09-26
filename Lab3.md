# Lab3: page tables

## 0. xv6 book

`page table`，也就是页表，它用来存储`virtual memory`到 `physical memory`的映射，在RISC-V中，它以一种三级页表的形式存在，在cs61c中，我们了解到了TLB，就是类似于cache缓存的东西，但是不同的是，cache用于完成CPU到DRMA（实际上就是memory）之间的快速写存，而TLB则是完成快速查找映射，实际上CPU上传出的地址都是虚拟地址，我们都需要经过TLB（然后是页表）的查找转换成物理地址。

## 1. Print page table(`easy`)
```c
void
vmprinthelper(pagetable_t pagetable, int l) {
  if (l > 2) return;
  for (int i = 0; i < 512; i++) {
    pte_t pte = pagetable[i];
    if (pte & PTE_V) {
      for (int i = 0; i <= l; i++) printf(".. ");
      pte_t child = PTE2PA(pte);
      printf("%d: pte %p pa %p\n", i, pte, child);
      vmprinthelper((pagetable_t)child, l + 1);
    }
  }
}

void
vmprint(pagetable_t pagetable) {
  printf("page table %p\n", pagetable);
  vmprinthelper(pagetable, 0);
}
```

Hint: Don't forget to add vmprint declaration in kernel/defs.h and add if(p->pid == 1) ... in kernel/exec.c;
