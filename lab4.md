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
