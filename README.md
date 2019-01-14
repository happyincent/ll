# ll

[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/r-medina/ll/blob/master/LICENSE) 

This project implements a thread-safe linked list for the C language. The API is intended
to be intuitive and simple as well as robust and useful. The data in the linked-list is
stored as a `void *`. The user is not exposed to any of the underlying implementation (no
access to nodes). The mutex on a linked-list, however, is exposed, so the user has access
to that if need be.

## Installing

Running

```bash
$ git clone https://github.com/r-medina/ll.git && cd ./ll/ && make o && cd ..
```

will download this project and build the object code in `ll/obj/`, which can then be
linked or whatever.

`make exec` will build the executable that has the tests.

## Examples

## API

The `include/ll.h` documents all the features to which a user has access.

### Data Structure

The data structure that governs this project is a simple singly-linked-list with a
reader/writer mutex. The two functional attributes help the user print their linked list
and deconstruct the elements.

```c
// linked list
struct ll {
    // running length
    int len;

    // pointer to the first node
    ll_node_t *hd;

    // mutex for thread safety
    pthread_rwlock_t m;

    // a function that is called every time a value is deleted
    // with a pointer to that value
    gen_fun_t val_teardown;

    // a function that can print the values in a linked list
    gen_fun_t val_printer;
};
```

### Functions

```c
// returns a pointer to an allocated linked list.
// needs a taredown function that is called with
// a pointer to the value when it is being deleted.
ll_t *ll_new(gen_fun_t val_teardown);

// traverses the linked list, deallocated everything (including `list`)
void ll_delete(ll_t *list);

// puts a value at the end of the linked list.
// returns the new length of the linked list if successful, -1 otherwise
int ll_insert_last(ll_t *list, int val);

// given a function that tests the values in the linked list, the first element that
// satisfies that function is removed.
// returns the new length of the linked list if successful, -1 otherwise
int ll_remove_search(ll_t *list, int cond(int, int), int);

// runs f on all values of list
void ll_map(ll_t *list, gen_fun_t f);

// goes through all the values of a linked list and calls `list->val_printer` on them
void ll_print(ll_t list);

// a generic taredown function for values that don't need anything done
void ll_no_teardown(int n);
```

## Testing

```bash
$ make test
```

---

## Add `test.c` (ref: [concurrent-ll](https://github.com/jserv/concurrent-ll))

### Enviornments
```
$ cat <(echo "CPU:    " `lscpu | grep "Model name" | cut -d':' -f2 | sed "s/  //"`) <(echo "OS:     " `lsb_release -d | cut -f2`) <(echo "Kernel: " `uname -a | cut -d' ' -f1,3,14`) <(echo "gcc:    " `gcc --version | head -n1`)
CPU:     Intel(R) Xeon(R) CPU E5520 @ 2.27GHz
OS:      Ubuntu 16.04.5 LTS
Kernel:  Linux 4.15.0-43-generic x86_64
gcc:     gcc (Ubuntu 5.4.0-6ubuntu1~16.04.11) 5.4.0 20160609
```

### Result
```
itlab@ITLabHP:~/Desktop/ll$ make clean all
building object files...
gcc -g -O1  -Wall -Werror -Wextra -Wunused -std=gnu99 -D_GNU_SOURCE -pthread -fno-strict-aliasing -D_REENTRANT -pedantic -I"include" -o obj/ll.o -MMD -MF obj/ll.o.d -c src/ll.c
building object files...
gcc -g -O1  -Wall -Werror -Wextra -Wunused -std=gnu99 -D_GNU_SOURCE -pthread -fno-strict-aliasing -D_REENTRANT -pedantic -I"include" -o obj/test.o -MMD -MF obj/test.o.d -c src/test.c
building binary...
gcc -g -O1  -Wall -Werror -Wextra -Wunused -std=gnu99 -D_GNU_SOURCE -pthread -fno-strict-aliasing -D_REENTRANT -pedantic -I"include" -o bin/test obj/ll.o obj/test.o
itlab@ITLabHP:~/Desktop/ll$ gdb -q --args ./bin/test "-n 300"
Reading symbols from ./bin/test...done.
(gdb) run
Starting program: /home/itlab/Desktop/ll/bin/test -n\ 300
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7ffff77ef700 (LWP 5077)]
...
[Thread 0x7ffd9cfe1700 (LWP 5435) exited]
...
Thread 299
  #operations   : 26
  #inserts   : 2
  #removes   : 1
Duration      : 1000 (ms)
#txs     : 5819 (5819.000000 / s)
Expected size: 1237 Actual size: 1237
[Inferior 1 (process 5064) exited normally]
(gdb)
```

```
$ G_SLICE=always-malloc valgrind ./bin/test -n 300
...
==26508== HEAP SUMMARY:
==26508==     in use at exit: 19,200 bytes in 300 blocks
==26508==   total heap usage: 4,613 allocs, 4,313 frees, 414,560 bytes allocated
==26508==
==26508== LEAK SUMMARY:
==26508==    definitely lost: 19,200 bytes in 300 blocks
==26508==    indirectly lost: 0 bytes in 0 blocks
==26508==      possibly lost: 0 bytes in 0 blocks
==26508==    still reachable: 0 bytes in 0 blocks
==26508==         suppressed: 0 bytes in 0 blocks
==26508== Rerun with --leak-check=full to see details of leaked memory
==26508==
==26508== For counts of detected and suppressed errors, rerun with: -v
==26508== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```