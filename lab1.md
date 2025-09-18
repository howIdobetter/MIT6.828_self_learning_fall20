## This lab1

## sleep
```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
  if (argc != 2) {
    printf("Usage: sleep <ticks>\n");
    exit(1);
  }

  int ticks = atoi(argv[1]);
  if (ticks <= 0) {
    printf("Error: ticks must be a positive integer\n");
    exit(1);
  }

  sleep(ticks);

  exit(0);
}
```

## pingpong
```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, int *argv[]) {
  int p1[2]; // parent -> child
  int p2[2]; // child  -> parent
  pipe(p1);
  pipe(p2);

  int pid = fork();
  if (pid < 0) {
    fprintf(2, "failed to fork\n");
    exit(1);
  }

  if (pid == 0) {
    /* child fork */
    char buf;
    close(p1[1]); // parent write
    close(p2[0]); // parent read

    if (read(p1[0], &buf, 1) != 1) {
      fprintf(2, "child: fail to recive ping\n");
      exit(1);
    }
    close(p1[0]);

    printf("%d: received ping\n", getpid());

    if (write(p2[1], &buf, 1) != 1) {
      fprintf(2, "child: fail to write pong\n");
      exit(1);
    }
    close(p2[1]);

    exit(0);
  } else {
    /* parent fork */
    char buf = 'p';
    close(p1[0]);
    close(p2[1]);

    if (write(p1[1], &buf, 1) != 1) {
      fprintf(2, "parent: fail to write ping\n");
      exit(1);
    }
    close(p1[1]);

    wait(0);

    if (read(p2[0], &buf, 1) != 1) {
      fprintf(2, "parent: fail to receive pong\n");
      exit(1);
    }
    close(p2[0]);

    printf("%d: received pong\n", getpid());
    exit(0);
  }
}
```
