## 0. xv6 book
我们可以使用懒分配来节省空间资源和时间资源，只有在进程需要分配内存的时候才通过page fault去disk取页面

## 1. Eliminate allocation from sbrk() `(easy)`

just remove growproc and modify

```c
uint64
sys_sbrk(void)
{
  int addr;
  int n;
  struct proc *p = myproc();

  if(argint(0, &n) < 0)
    return -1;
  addr = myproc()->sz;
  // if(growproc(n) < 0)
  //   return -1;
  if(n > 0 && p->sz + n < p->sz) {
    return -1;
  }
  
  if(n > 0){
    p->sz += n;
  } else if(n < 0){
    if(p->sz + n < 0)
      return -1;
    p->sz = uvmdealloc(p->pagetable, p->sz, p->sz + n);
  }
  return addr;
}
```

## 2. Lazy allocation (moderate)

```c
void
uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free)
{
  uint64 a;
  pte_t *pte;

  if((va % PGSIZE) != 0)
    panic("uvmunmap: not aligned");

  for(a = va; a < va + npages*PGSIZE; a += PGSIZE){
    if((pte = walk(pagetable, a, 0)) == 0)
      // panic("uvmunmap: walk");
      continue;
    if((*pte & PTE_V) == 0)
      // panic("uvmunmap: not mapped");
      continue;
    if(PTE_FLAGS(*pte) == PTE_V)
      panic("uvmunmap: not a leaf");
    if(do_free){
      uint64 pa = PTE2PA(*pte);
      kfree((void*)pa);
    }
    *pte = 0;
  }
}
```

```c
void
usertrap(void)
{
  int which_dev = 0;

  if((r_sstatus() & SSTATUS_SPP) != 0)
    panic("usertrap: not from user mode");

  // send interrupts and exceptions to kerneltrap(),
  // since we're now in the kernel.
  w_stvec((uint64)kernelvec);

  struct proc *p = myproc();
  
  // save user program counter.
  p->trapframe->epc = r_sepc();
  
  if(r_scause() == 8){
    // system call

    if(p->killed)
      exit(-1);

    // sepc points to the ecall instruction,
    // but we want to return to the next instruction.
    p->trapframe->epc += 4;

    // an interrupt will change sstatus &c registers,
    // so don't enable until done with those registers.
    intr_on();

    syscall();
  } else if((which_dev = devintr()) != 0){
    // ok
  } else {
    if(r_scause() == 13 || r_scause() == 15) {
      uint64 fault_va = r_stval();
      uint64 aligned_va = PGROUNDDOWN(fault_va);
      
      char *mem = kalloc();
      if(mem == 0) {
        printf("usertrap(): kalloc failed\n");
        p->killed = 1;
      } else {
        memset(mem, 0, PGSIZE);
        if(mappages(p->pagetable, aligned_va, PGSIZE, (uint64)mem, PTE_V|PTE_W|PTE_R|PTE_U) != 0) {
          kfree(mem);
          printf("usertrap(): mappages failed\n");
          p->killed = 1;
        }
      }
    } else {
      printf("usertrap(): unexpected scause %p pid=%d\n", r_scause(), p->pid);
      printf("            sepc=%p stval=%p\n", r_sepc(), r_stval());
      p->killed = 1;
    }
  }

  if(p->killed)
    exit(-1);

  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2)
    yield();

  usertrapret();
}
```
## 3. Lazytests and Usertests (moderate)

需要弄清楚的是，在这里，我们如何产生懒分配的page fault，在用户态，我们sbrk之后可能直接对那个地址进行赋值，但是实际上那个地址不存在（还没有分配），因此造成了page fault，事实上，以上我们解决的是这个问题，但是还有一种情况就是我们主动进行系统调用，同样造成了page fault，因此我们需要对sys_write和sys_read进行修改，同样需要检查那个地址满足以下条件：

1. Kill a process if it page-faults on a virtual memory address higher than any allocated with sbrk().
2. Handle out-of-memory correctly: if kalloc() fails in the page fault handler, kill the current process.
3. Handle faults on the invalid page below the user stack.

这里第二条非常好理解，就是kalloc失败杀死进程，第一条就是我们发生page-fault的va > p->sz，我们sbrk分配的堆空间，坑定不能大于啊，第三条就是page fault发生在保护页及以下，这显然查出了我们的用户栈。

最后在过程中还遇到了一个bug，就是remap的检查，注意要在lazy_alloc_page中添加对已分配的页面的检查。


