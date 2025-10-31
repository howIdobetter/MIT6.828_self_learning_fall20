## 0. xv6 book

read by yourself

## 1. RISC-V assembly(`easy`)

1. a0, a1, a2, a3, a4, a5, a6, a7; a2
2. inline optimizes
3. 0x628
4. 0x38
5. He110, World; 0x726c6c00; No;
6. trash, in fact the a2(but not change)

## 2. Backtrace(`moderate`)

This part is easy, you just know a frame stack is in a page.

backtrace():

```c
void
backtrace()
{
  uint64 fp = r_fp();
  uint64 stack_top = PGROUNDUP(fp);
  uint64 stack_bottom = PGROUNDDOWN(fp);

  printf("backtrace:\n");

  for (int i = 0; i < 10; i++) {
    if (fp < stack_bottom || fp >= stack_top) {
      break;
    }

    uint64 ra = *(uint64*)(fp - 8);
    if (ra == 0) break;

    printf("%p\n", ra);

    uint64 prev_fp = *(uint64*)(fp - 16);
    if (prev_fp <= fp) break;
    fp = prev_fp;
  }
}
```

Don't forget add it to defs.h and the beginning of sys_sleep() in sysproc.

## 3. Alarm(`hard`)

just provide the core code:

test0

```c
usertrap()
// give up the CPU if this is a timer interrupt.
  if(which_dev == 2) {
    struct proc *p = myproc();
    if (p && p->alarm_interval > 0) {
      p->alarm_ticks++;
      if (p->alarm_ticks >= p->alarm_interval) {
        p->trapframe->epc = (uint64)p->handler_function;
        p->alarm_ticks = 0;
      }
    }
    yield();
  }

uint64
sys_sigalarm(void)
{
  int ticks;
  uint64 handler;
  if (argint(0, &ticks) < 0)
    return -1;
  if (argaddr(1, &handler) < 0)
    return -1;
  
  myproc()->alarm_interval = ticks;
  myproc()->handler_function = (void *)handler;
  myproc()->alarm_ticks = 0;

  return 0;
}
```

test1/test2:

```c
uint64
sys_sigreturn(void)
{
  struct proc *p = myproc();
  p->alarm_going_off = 0;
  memmove(p->trapframe, &p->alarm_saved_trapframe, sizeof(struct trapframe));
  return 0;
}

usertrap()
// give up the CPU if this is a timer interrupt.
  if(which_dev == 2) {
    struct proc *p = myproc();
    if (p && p->alarm_interval > 0) {
      p->alarm_ticks++;
      if (p->alarm_ticks >= p->alarm_interval && !p->alarm_going_off) {
        memmove(&p->alarm_saved_trapframe, p->trapframe, sizeof(struct trapframe));
        p->trapframe->epc = (uint64)p->handler_function;
        p->alarm_ticks = 0;
        p->alarm_going_off = 1;
      }
    }
    yield();
  }

struct proc
=== add ===
int alarm_interval;
  void *handler_function;
  int alarm_ticks;
  struct trapframe alarm_saved_trapframe;
  int alarm_going_off;
=== add ===
```
