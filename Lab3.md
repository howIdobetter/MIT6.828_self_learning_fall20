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

Hint: Don't forget to add vmprint declaration in kernel/defs.h and add if(p->pid == 1) ... in kernel/exec.c;

## 2. A kernel page table per process(hard)

Too hard!!!!!

If you're right may be this:
```bash
$ usertests
usertests starting
test execout: OK
test copyin: OK
test copyout: OK
test copyinstr1: OK
test copyinstr2: OK
test copyinstr3: OK
test truncate1: OK
test truncate2: OK
test truncate3: OK
test reparent2: OK
test pgbug: OK
test sbrkbugs: usertrap(): unexpected scause 0x000000000000000c pid=3234
            sepc=0x00000000000053fc stval=0x00000000000053fc
usertrap(): unexpected scause 0x000000000000000c pid=3235
            sepc=0x00000000000053fc stval=0x00000000000053fc
OK
test badarg: OK
test reparent: OK
test twochildren: OK
test forkfork: OK
test forkforkfork: OK
test argptest: OK
test createdelete: OK
test linkunlink: OK
test linktest: OK
test unlinkread: OK
test concreate: OK
test subdir: OK
test fourfiles: OK
test sharedfd: OK
test exectest: OK
test bigargtest: OK
test bigwrite: OK
test bsstest: OK
test sbrkbasic: OK
test sbrkmuch: OK
test kernmem: usertrap(): unexpected scause 0x000000000000000d pid=6214
            sepc=0x000000000000201a stval=0x0000000080000000
usertrap(): unexpected scause 0x000000000000000d pid=6215
            sepc=0x000000000000201a stval=0x000000008000c350
usertrap(): unexpected scause 0x000000000000000d pid=6216
            sepc=0x000000000000201a stval=0x00000000800186a0
usertrap(): unexpected scause 0x000000000000000d pid=6217
            sepc=0x000000000000201a stval=0x00000000800249f0
usertrap(): unexpected scause 0x000000000000000d pid=6218
            sepc=0x000000000000201a stval=0x0000000080030d40
usertrap(): unexpected scause 0x000000000000000d pid=6219
            sepc=0x000000000000201a stval=0x000000008003d090
usertrap(): unexpected scause 0x000000000000000d pid=6220
            sepc=0x000000000000201a stval=0x00000000800493e0
usertrap(): unexpected scause 0x000000000000000d pid=6221
            sepc=0x000000000000201a stval=0x0000000080055730
usertrap(): unexpected scause 0x000000000000000d pid=6222
            sepc=0x000000000000201a stval=0x0000000080061a80
usertrap(): unexpected scause 0x000000000000000d pid=6223
            sepc=0x000000000000201a stval=0x000000008006ddd0
usertrap(): unexpected scause 0x000000000000000d pid=6224
            sepc=0x000000000000201a stval=0x000000008007a120
usertrap(): unexpected scause 0x000000000000000d pid=6225
            sepc=0x000000000000201a stval=0x0000000080086470
usertrap(): unexpected scause 0x000000000000000d pid=6226
            sepc=0x000000000000201a stval=0x00000000800927c0
usertrap(): unexpected scause 0x000000000000000d pid=6227
            sepc=0x000000000000201a stval=0x000000008009eb10
usertrap(): unexpected scause 0x000000000000000d pid=6228
            sepc=0x000000000000201a stval=0x00000000800aae60
usertrap(): unexpected scause 0x000000000000000d pid=6229
            sepc=0x000000000000201a stval=0x00000000800b71b0
usertrap(): unexpected scause 0x000000000000000d pid=6230
            sepc=0x000000000000201a stval=0x00000000800c3500
usertrap(): unexpected scause 0x000000000000000d pid=6231
            sepc=0x000000000000201a stval=0x00000000800cf850
usertrap(): unexpected scause 0x000000000000000d pid=6232
            sepc=0x000000000000201a stval=0x00000000800dbba0
usertrap(): unexpected scause 0x000000000000000d pid=6233
            sepc=0x000000000000201a stval=0x00000000800e7ef0
usertrap(): unexpected scause 0x000000000000000d pid=6234
            sepc=0x000000000000201a stval=0x00000000800f4240
usertrap(): unexpected scause 0x000000000000000d pid=6235
            sepc=0x000000000000201a stval=0x0000000080100590
usertrap(): unexpected scause 0x000000000000000d pid=6236
            sepc=0x000000000000201a stval=0x000000008010c8e0
usertrap(): unexpected scause 0x000000000000000d pid=6237
            sepc=0x000000000000201a stval=0x0000000080118c30
usertrap(): unexpected scause 0x000000000000000d pid=6238
            sepc=0x000000000000201a stval=0x0000000080124f80
usertrap(): unexpected scause 0x000000000000000d pid=6239
            sepc=0x000000000000201a stval=0x00000000801312d0
usertrap(): unexpected scause 0x000000000000000d pid=6240
            sepc=0x000000000000201a stval=0x000000008013d620
usertrap(): unexpected scause 0x000000000000000d pid=6241
            sepc=0x000000000000201a stval=0x0000000080149970
usertrap(): unexpected scause 0x000000000000000d pid=6242
            sepc=0x000000000000201a stval=0x0000000080155cc0
usertrap(): unexpected scause 0x000000000000000d pid=6243
            sepc=0x000000000000201a stval=0x0000000080162010
usertrap(): unexpected scause 0x000000000000000d pid=6244
            sepc=0x000000000000201a stval=0x000000008016e360
usertrap(): unexpected scause 0x000000000000000d pid=6245
            sepc=0x000000000000201a stval=0x000000008017a6b0
usertrap(): unexpected scause 0x000000000000000d pid=6246
            sepc=0x000000000000201a stval=0x0000000080186a00
usertrap(): unexpected scause 0x000000000000000d pid=6247
            sepc=0x000000000000201a stval=0x0000000080192d50
usertrap(): unexpected scause 0x000000000000000d pid=6248
            sepc=0x000000000000201a stval=0x000000008019f0a0
usertrap(): unexpected scause 0x000000000000000d pid=6249
            sepc=0x000000000000201a stval=0x00000000801ab3f0
usertrap(): unexpected scause 0x000000000000000d pid=6250
            sepc=0x000000000000201a stval=0x00000000801b7740
usertrap(): unexpected scause 0x000000000000000d pid=6251
            sepc=0x000000000000201a stval=0x00000000801c3a90
usertrap(): unexpected scause 0x000000000000000d pid=6252
            sepc=0x000000000000201a stval=0x00000000801cfde0
usertrap(): unexpected scause 0x000000000000000d pid=6253
            sepc=0x000000000000201a stval=0x00000000801dc130
OK
test sbrkfail: usertrap(): unexpected scause 0x000000000000000d pid=6265
            sepc=0x0000000000003e76 stval=0x0000000000012000
OK
test sbrkarg: OK
test validatetest: OK
test stacktest: usertrap(): unexpected scause 0x000000000000000d pid=6269
            sepc=0x0000000000002188 stval=0x000000000000fbc0
OK
test opentest: OK
test writetest: OK
test writebig: OK
test createtest: OK
test openiput: OK
test exitiput: OK
test iput: OK
test mem: OK
test pipe1: OK
test preempt: kill... wait... OK
test exitwait: OK
test rmdot: OK
test fourteen: OK
test bigfile: OK
test dirfile: OK
test iref: OK
test forktest: OK
test bigdir: OK
ALL TESTS PASSED
```

Firstly, we should create proc->kpagetable for every thread(or called process)。。。
```c
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  struct proc *parent;         // Parent process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)

  pagetable_t kpagetable;      //<- this!
};
```

Secondly, we just need to create kvminit_new and ukvmmap in vm.c.

```c
pagetable_t
kvminit_new()
{
  pagetable_t kpagetable = uvmcreate();
  if (kpagetable == 0) {
    return 0;
  }
  ukvmmap(kpagetable, UART0, UART0, PGSIZE, PTE_R | PTE_W);
  ukvmmap(kpagetable, VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);
  ukvmmap(kpagetable, CLINT, CLINT, 0x10000, PTE_R | PTE_W);
  ukvmmap(kpagetable, PLIC, PLIC, 0x400000, PTE_R | PTE_W);
  ukvmmap(kpagetable, KERNBASE, KERNBASE, (uint64)etext-KERNBASE, PTE_R | PTE_X);
  ukvmmap(kpagetable, (uint64)etext, (uint64)etext, PHYSTOP-(uint64)etext, PTE_R | PTE_W);
  ukvmmap(kpagetable, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);

  return kpagetable;
}

void
ukvmmap(pagetable_t kpagetable, uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kpagetable, va, sz, pa, perm) != 0)
    panic("kvmmap");
}
```

Don't forget to add them to defs.h.

Thirdly, add allcoproc in proc.c

```c
static struct proc*
allocproc(void)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    acquire(&p->lock);
    if(p->state == UNUSED) {
      goto found;
    } else {
      release(&p->lock);
    }
  }
  return 0;

found:
  p->pid = allocpid();

  // Allocate a trapframe page.
  if((p->trapframe = (struct trapframe *)kalloc()) == 0){
    release(&p->lock);
    return 0;
  }

  // An empty user page table.
  p->pagetable = proc_pagetable(p);
  if(p->pagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  // An empty kernel page table.
  p->kpagetable = kvminit_new();
  if(p->kpagetable == 0) {
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  // init the kernel stack.
  char *pa = kalloc();
  if(pa == 0)
    panic("stack kalloc");
  uint64 va = KSTACK((int) (p - proc));
  ukvmmap(p->kpagetable, va, (uint64)pa, PGSIZE, PTE_R | PTE_W);
  p->kstack = va;

  // Set up new context to start executing at forkret,
  // which returns to user space.
  memset(&p->context, 0, sizeof(p->context));
  p->context.ra = (uint64)forkret;
  p->context.sp = p->kstack + PGSIZE;

  return p;
}
```

we also add kstack from procinit in proc.c, and maybe we should remove it from procinit.

Then, we finish scheduler in proc.c, we will learn it in interrupt chapter.

```c
void
scheduler(void)
{
  struct proc *p;
  struct cpu *c = mycpu();
  
  c->proc = 0;
  for(;;){
    // Avoid deadlock by ensuring that devices can interrupt.
    intr_on();
    kvminithart();
    int found = 0;
    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Switch to chosen process.  It is the process's job
        // to release its lock and then reacquire it
        // before jumping back to us.
        p->state = RUNNING;
        c->proc = p;

        w_satp(MAKE_SATP(p->kpagetable));
        sfence_vma();
        swtch(&c->context, &p->context);

        // Process is done running for now.
        // It should have changed its p->state before coming back.
        c->proc = 0;
        found = 1;
      }
      release(&p->lock);
    }

    if (found == 0) {
      // Switch back to the kernel's page table.
      kvminithart();
      intr_on();
      asm volatile("wfi");
    }
  }
}
```

Lastly, we finish the free.

```c
void
proc_kfreepagetable(pagetable_t kpagetable)
{
  for (int i = 0; i < 512; i++) {
		pte_t pte = kpagetable[i];
		if (pte & PTE_V) {
			if ((pte & (PTE_R|PTE_W|PTE_X)) == 0) {
				uint64 child = PTE2PA(pte);
				proc_kfreepagetable((pagetable_t)child);
			}
		}
	}
	kfree((void*)kpagetable);
}

static void
freeproc(struct proc *p)
{
  if(p->trapframe)
    kfree((void*)p->trapframe);
  p->trapframe = 0;
  // free kstack
  pte_t *pte = walk(p->kpagetable, p->kstack, 0);
  if(pte == 0)
    panic("freeproc: free kstack");
  kfree((void*)PTE2PA(*pte));
  p->kstack = 0;
  if(p->pagetable)
    proc_freepagetable(p->pagetable, p->sz);
  p->pagetable = 0;
  if(p->kpagetable)
    proc_kfreepagetable(p->kpagetable);
  p->kpagetable = 0;
  p->sz = 0;
  p->pid = 0;
  p->parent = 0;
  p->name[0] = 0;
  p->chan = 0;
  p->killed = 0;
  p->xstate = 0;
  p->state = UNUSED;
}
```

wait, maybe you get error, we must solve it, try to change this

```c
uint64
kvmpa(pagetable_t pgtbl, uint64 va)
{
  uint64 off = va % PGSIZE;
  pte_t *pte;
  uint64 pa;

  pte = walk(pgtbl, va, 0);
  if(pte == 0)
    panic("kvmpa");
  if((*pte & PTE_V) == 0)
    panic("kvmpa");
  pa = PTE2PA(*pte);
  return pa+off
}
```

and this

```c
void
virtio_disk_rw(struct buf *b, int write)
{
  uint64 sector = b->blockno * (BSIZE / 512);

  acquire(&disk.vdisk_lock);

  // the spec says that legacy block operations use three
  // descriptors: one for type/reserved/sector, one for
  // the data, one for a 1-byte status result.

  // allocate the three descriptors.
  int idx[3];
  while(1){
    if(alloc3_desc(idx) == 0) {
      break;
    }
    sleep(&disk.free[0], &disk.vdisk_lock);
  }
  // format the three descriptors.
  // qemu's virtio-blk.c reads them.

  struct virtio_blk_outhdr {
    uint32 type;
    uint32 reserved;
    uint64 sector;
  } buf0;

  if(write)
    buf0.type = VIRTIO_BLK_T_OUT; // write the disk
  else
    buf0.type = VIRTIO_BLK_T_IN; // read the disk
  buf0.reserved = 0;
  buf0.sector = sector;

  // buf0 is on a kernel stack, which is not direct mapped,
  // thus the call to kvmpa().
  disk.desc[idx[0]].addr = (uint64) kvmpa(myproc()->kpagetable, (uint64) &buf0);
  disk.desc[idx[0]].len = sizeof(buf0);
  disk.desc[idx[0]].flags = VRING_DESC_F_NEXT;
  disk.desc[idx[0]].next = idx[1];

  disk.desc[idx[1]].addr = (uint64) b->data;
  disk.desc[idx[1]].len = BSIZE;
  if(write)
    disk.desc[idx[1]].flags = 0; // device reads b->data
  else
    disk.desc[idx[1]].flags = VRING_DESC_F_WRITE; // device writes b->data
  disk.desc[idx[1]].flags |= VRING_DESC_F_NEXT;
  disk.desc[idx[1]].next = idx[2];

  disk.info[idx[0]].status = 0;
  disk.desc[idx[2]].addr = (uint64) &disk.info[idx[0]].status;
  disk.desc[idx[2]].len = 1;
  disk.desc[idx[2]].flags = VRING_DESC_F_WRITE; // device writes the status
  disk.desc[idx[2]].next = 0;

  // record struct buf for virtio_disk_intr().
  b->disk = 1;
  disk.info[idx[0]].b = b;

  // avail[0] is flags
  // avail[1] tells the device how far to look in avail[2...].
  // avail[2...] are desc[] indices the device should process.
  // we only tell device the first index in our chain of descriptors.
  disk.avail[2 + (disk.avail[1] % NUM)] = idx[0];
  __sync_synchronize();
  disk.avail[1] = disk.avail[1] + 1;

  *R(VIRTIO_MMIO_QUEUE_NOTIFY) = 0; // value is queue number

  // Wait for virtio_disk_intr() to say request has finished.
  while(b->disk == 1) {
    sleep(b, &disk.vdisk_lock);
  }

  disk.info[idx[0]].b = 0;
  free_chain(idx[0]);

  release(&disk.vdisk_lock);
}
```

Ok, that's all, too hard.  -----qwq

## 3. Simplify copyin/copyinstr (hard)

Firstly, we replace copyin and copyinstr in vm.c, and it's easy to implement.

Then we can create copy_user_to_kernel.

Last change userinit, growproc, exec, fork.

That's all.
