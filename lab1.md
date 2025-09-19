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


## primes
```c
#include "kernel/types.h"
#include "user/user.h"

/**
 * We create mutiply processes to finish the
 * primes.
 */

/**
 * Create a new process
 */
void new_process(int p[2]) {
  int n;
  int prime;
  close(p[1]);
  int count1 = read(p[0], &prime, 4);
  if (count1 != 4 && count1 != 0) {
    fprintf(2, "failed to received a prime\n");
    exit(1);
  }
  int count2 = read(p[0], &n, 4);
  if (count2 != 0 && count2 != 4) {
    fprintf(2, "count is not true\n");
    exit(1);
  }
  if (count1) {
    printf("prime %d\n", prime);
  }
  if (count2) {
    int newp[2];
    pipe(newp);
    if (fork() == 0) {
      new_process(newp);
    } else {
      close(newp[0]);
      if (n % prime != 0) {
        if (write(newp[1], &n, 4) != 4) {
          fprintf(2, "%d failed to write to newp[1]\n", n);
        }
      }
      while (read(p[0], &n, 4) == 4) {
        if (n % prime != 0) {
          if (write(newp[1], &n, 4) != 4) {
            fprintf(2, "%d failed to write to newp[1]\n", n);
          }
        }
      }
      close(p[0]);
      close(newp[1]);
      wait(0);
    }
  }
  exit(0);
}

int main() {
  int p[2];
  pipe(p);
  int n;

  if (fork() == 0) {
    new_process(p);
  } else {
    close(p[0]);
    for (int i = 2; i <= 35; i++) {
      n = i;
      if (write(p[1], &n, 4) != 4) {
        fprintf(2, "The first process failed to write number %d\n", i);
        exit(1);
      }
    }
    close(p[1]);
    wait(0);
    exit(0);
  }

  exit(0);
}
```

## find
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

void find(const char *path, const char *target) {
    char buf[512], *p;
    int fd;
    struct dirent de;
    struct stat st;

    if ((fd = open(path, 0)) < 0) {
        fprintf(2, "find: cannot open %s\n", path);
        return;
    }

    if (fstat(fd, &st) < 0) {
        fprintf(2, "find: cannot stat %s\n", path);
        close(fd);
        return;
    }

    if (st.type != T_DIR) {
        fprintf(2, "find: %s is not a directory\n", path);
        close(fd);
        return;
    }

    if (strlen(path) + 1 + DIRSIZ + 1 > sizeof buf) {
        printf("find: path too long\n");
        close(fd);
        return;
    }

    strcpy(buf, path);
    p = buf + strlen(buf);
    *p++ = '/';

    while (read(fd, &de, sizeof(de)) == sizeof(de)) {
        if (de.inum == 0)
            continue;
        if (strcmp(de.name, ".") == 0 || strcmp(de.name, "..") == 0)
            continue;

        memmove(p, de.name, DIRSIZ);
        p[DIRSIZ] = 0;

        if (stat(buf, &st) < 0) {
            printf("find: cannot stat %s\n", buf);
            continue;
        }

        if (st.type == T_DIR) {
            find(buf, target);
        }

        if (strcmp(de.name, target) == 0) {
            printf("%s\n", buf);
        }
    }

    close(fd);
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(2, "Usage: find <directory> <filename>\n");
        exit(1);
    }
    find(argv[1], argv[2]);
    exit(0);
}
```
