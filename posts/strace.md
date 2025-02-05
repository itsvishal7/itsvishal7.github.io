# strace 

-D, -DD, -DDD options
```
       -D
       --daemonize
       --daemonize=grandchild
                   Run  tracer  process  as  a grandchild, not as the parent of the tracee.
                   This reduces the visible effect of strace by keeping the tracee a direct
                   child of the calling process.

       -DD
       --daemonize=pgroup
       --daemonize=pgrp
                   Run tracer process as tracee's grandchild in a separate  process  group.
                   In addition to reduction of the visible effect of strace, it also avoids
                   killing of strace with kill(2) issued to the whole process group.

       -DDD
       --daemonize=session
                   Run  tracer  process as tracee's grandchild in a separate session ("true
                   daemonisation").  In addition to reduction  of  the  visible  effect  of
                   strace, it also avoids killing of strace upon session termination.

```

### htop output
#### without -D, -DD, -DDD
without `daemonize` option, killing `strace` would kill the `stress-ng`
too, in other words killing tracer will kill tracee too.
```
PID    PPID▽   PGRP     SID    TGID Command
113013  113012  113009  111158  113013 stress-ng-cpu [run]
113012  113009  113009  111158  113012 stress-ng -c 1
113009  111158  113009  111158  113009 strace -- stress-ng -c 1
```

<details><summary>ptrace output of strace</summary>

```
Attaching 5 probes...
speech-dispatch:91690: clone
bash:126787: clone
bash:136317: exec
strace:136317: clone
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00004206, target_pid: 0x0002147e, addr: 0x00000000, data: 0x00000000
strace:136317: clone
ptrace ENTER (tid=136319 pid=136319 comm=strace): request: 0x00000000, target_pid: 0x00000000, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00004200, target_pid: 0x0002147f, addr: 0x00000000, data: 0x00000001
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x0002147f, addr: 0x00000058, data: 0x7ffe89008990
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x0002147f, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x0002147f, addr: 0x00000058, data: 0x7ffe89008990
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x0002147f, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x0002147f, addr: 0x00000058, data: 0x7ffe89008990
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x0002147f, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x0002147f, addr: 0x00000058, data: 0x7ffe89008990
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x0002147f, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x0002147f, addr: 0x00000058, data: 0x7ffe89008990
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x0002147f, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x0002147f, addr: 0x00000058, data: 0x7ffe89008990
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x0002147f, addr: 0x00000000, data: 0x00000000
strace:136317: clone
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00004206, target_pid: 0x00021480, addr: 0x00000000, data: 0x00000051
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00004207, target_pid: 0x00021480, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00004208, target_pid: 0x00021480, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x00021480, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00004202, target_pid: 0x00021480, addr: 0x00000000, data: 0x5633c55ccd20
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x00021480, addr: 0x00000000, data: 0x00000012
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x00021480, addr: 0x00000058, data: 0x5633872353e0
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x00021480, addr: 0x00000000, data: 0x00000000
strace:136320: exec
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00004201, target_pid: 0x00021480, addr: 0x00000000, data: 0x5633c55ccd18
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x00000018, target_pid: 0x00021480, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136317 pid=136317 comm=strace): request: 0x0000420e, target_pid: 0x00021480, addr: 0x00000058, data: 0x5633872353e0
```
</details>

#### with -D
with -D, if the tracer gets killed as a side-effect tracee won't get
killed, but, if a signal sent to tracee's process group, it will be
received by the tracer as well. meaning tracee can affect tracer

To see this in action, use `kill` to send a dummy signal to the process
group id
```
strace -p 112660

## in another terminal
$ kill -18 -112656
```


```
   PID    PPID▽   PGRP     SID    TGID Command
112661  112656  112656  111158  112661 stress-ng-cpu [run]
112656  111158  112656  111158  112656 stress-ng -c 1
112660    3562  112656  111158  112660 strace -D -- stress-ng -c 1

```

<details><summary>ptrace output of strace</summary>

```
Attaching 5 probes...
speech-dispatch:91690: clone
telemetryd_v2:135594: clone
telemetryd_v2:135594: clone
speech-dispatch:91690: clone
speech-dispatch:91690: clone
bash:126787: clone
bash:136387: exec
strace:136387: clone
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00004206, target_pid: 0x000214c4, addr: 0x00000000, data: 0x00000000
strace:136387: clone
ptrace ENTER (tid=136389 pid=136389 comm=strace): request: 0x00000000, target_pid: 0x00000000, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00004200, target_pid: 0x000214c5, addr: 0x00000000, data: 0x00000001
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x0000420e, target_pid: 0x000214c5, addr: 0x00000058, data: 0x7ffea1e83a90
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00000018, target_pid: 0x000214c5, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x0000420e, target_pid: 0x000214c5, addr: 0x00000058, data: 0x7ffea1e83a90
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00000018, target_pid: 0x000214c5, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x0000420e, target_pid: 0x000214c5, addr: 0x00000058, data: 0x7ffea1e83a90
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00000018, target_pid: 0x000214c5, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x0000420e, target_pid: 0x000214c5, addr: 0x00000058, data: 0x7ffea1e83a90
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00000018, target_pid: 0x000214c5, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x0000420e, target_pid: 0x000214c5, addr: 0x00000058, data: 0x7ffea1e83a90
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00000018, target_pid: 0x000214c5, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x0000420e, target_pid: 0x000214c5, addr: 0x00000058, data: 0x7ffea1e83a90
ptrace ENTER (tid=136387 pid=136387 comm=strace): request: 0x00000018, target_pid: 0x000214c5, addr: 0x00000000, data: 0x00000000
strace:136387: clone
strace:136390: clone
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00004206, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000051
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00004207, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00004204, target_pid: 0x000214c3, addr: 0x00000001, data: 0x559a2ddc70c0
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00000018, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x0000420e, target_pid: 0x000214c3, addr: 0x00000058, data: 0x559a2ddc73e0
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00000018, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00000018, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00004202, target_pid: 0x000214c3, addr: 0x00000000, data: 0x559a529bd150
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00000018, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000011
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x0000420e, target_pid: 0x000214c3, addr: 0x00000058, data: 0x559a2ddc73e0
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00000018, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x00000018, target_pid: 0x000214c3, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136391 pid=136391 comm=strace): request: 0x0000420e, target_pid: 0x000214c3, addr: 0x00000058, data: 0x559a2ddc73e0
```
</details>

#### with -DD
with -DD, tracer runs in a separate process group, but, in same session as
tracee,
and signal sent to tracee's session will be sent to the tracer as well
```
$ strace -p 112827

## in another terminal
pkill -s 111158 --signal 18
```
```
   PID    PPID▽   PGRP     SID    TGID Command
112828  112823  112823  111158  112828 stress-ng-cpu [run]
112823  111158  112823  111158  112823 stress-ng -c 1
112827    3562  112827  111158  112827 strace -DD -- stress-ng -c 1

```

<details><summary>ptrace output of strace</summary>

```
Attaching 5 probes...
speech-dispatch:91690: clone
bash:126787: clone
bash:136432: exec
strace:136432: clone
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00004206, target_pid: 0x000214f1, addr: 0x00000000, data: 0x00000000
strace:136432: clone
ptrace ENTER (tid=136434 pid=136434 comm=strace): request: 0x00000000, target_pid: 0x00000000, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00004200, target_pid: 0x000214f2, addr: 0x00000000, data: 0x00000001
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x0000420e, target_pid: 0x000214f2, addr: 0x00000058, data: 0x7ffd71a89560
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00000018, target_pid: 0x000214f2, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x0000420e, target_pid: 0x000214f2, addr: 0x00000058, data: 0x7ffd71a89560
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00000018, target_pid: 0x000214f2, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x0000420e, target_pid: 0x000214f2, addr: 0x00000058, data: 0x7ffd71a89560
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00000018, target_pid: 0x000214f2, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x0000420e, target_pid: 0x000214f2, addr: 0x00000058, data: 0x7ffd71a89560
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00000018, target_pid: 0x000214f2, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x0000420e, target_pid: 0x000214f2, addr: 0x00000058, data: 0x7ffd71a89560
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00000018, target_pid: 0x000214f2, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x0000420e, target_pid: 0x000214f2, addr: 0x00000058, data: 0x7ffd71a89560
ptrace ENTER (tid=136432 pid=136432 comm=strace): request: 0x00000018, target_pid: 0x000214f2, addr: 0x00000000, data: 0x00000000
strace:136432: clone
strace:136435: clone
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00004206, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000051
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00004207, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00004204, target_pid: 0x000214f0, addr: 0x00000001, data: 0x562822dec0c0
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x0000420e, target_pid: 0x000214f0, addr: 0x00000058, data: 0x562822dec3e0
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00004202, target_pid: 0x000214f0, addr: 0x00000000, data: 0x562861fcd150
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000011
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x0000420e, target_pid: 0x000214f0, addr: 0x00000058, data: 0x562822dec3e0
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x0000420e, target_pid: 0x000214f0, addr: 0x00000058, data: 0x562822dec3e0
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
strace:136432: exec
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00004201, target_pid: 0x000214f0, addr: 0x00000000, data: 0x562861fcd148
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x0000420e, target_pid: 0x000214f0, addr: 0x00000058, data: 0x562822dec3e0
ptrace ENTER (tid=136436 pid=136436 comm=strace): request: 0x00000018, target_pid: 0x000214f0, addr: 0x00000000, data: 0x00000000
```
</details>

#### with -DDD
with -DDD, tracer runs in a separate process group and session than tracee
```
   PID    PPID▽   PGRP     SID    TGID Command
112925  112920  112920  111158  112925 stress-ng-cpu [run]
112920  111158  112920  111158  112920 stress-ng -c 1
112924    3562  112924  112924  112924 strace -DDD -- stress-ng

```

<details><summary>ptrace output of strace</summary>

```
Attaching 5 probes...
telemetryd_v2:135594: clone
speech-dispatch:91690: clone
bash:126787: clone
bash:136577: exec
strace:136577: clone
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00004206, target_pid: 0x00021582, addr: 0x00000000, data: 0x00000000
strace:136577: clone
ptrace ENTER (tid=136579 pid=136579 comm=strace): request: 0x00000000, target_pid: 0x00000000, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00004200, target_pid: 0x00021583, addr: 0x00000000, data: 0x00000001
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x0000420e, target_pid: 0x00021583, addr: 0x00000058, data: 0x7ffdcf68f770
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00000018, target_pid: 0x00021583, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x0000420e, target_pid: 0x00021583, addr: 0x00000058, data: 0x7ffdcf68f770
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00000018, target_pid: 0x00021583, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x0000420e, target_pid: 0x00021583, addr: 0x00000058, data: 0x7ffdcf68f770
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00000018, target_pid: 0x00021583, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x0000420e, target_pid: 0x00021583, addr: 0x00000058, data: 0x7ffdcf68f770
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00000018, target_pid: 0x00021583, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x0000420e, target_pid: 0x00021583, addr: 0x00000058, data: 0x7ffdcf68f770
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00000018, target_pid: 0x00021583, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x0000420e, target_pid: 0x00021583, addr: 0x00000058, data: 0x7ffdcf68f770
ptrace ENTER (tid=136577 pid=136577 comm=strace): request: 0x00000018, target_pid: 0x00021583, addr: 0x00000000, data: 0x00000000
strace:136577: clone
strace:136580: clone
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00004206, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000051
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00004207, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00004204, target_pid: 0x00021581, addr: 0x00000001, data: 0x55cb66f320c0
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x0000420e, target_pid: 0x00021581, addr: 0x00000058, data: 0x55cb66f323e0
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00004202, target_pid: 0x00021581, addr: 0x00000000, data: 0x55cb6e5fa150
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000011
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x0000420e, target_pid: 0x00021581, addr: 0x00000058, data: 0x55cb66f323e0
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x0000420e, target_pid: 0x00021581, addr: 0x00000058, data: 0x55cb66f323e0
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
strace:136577: exec
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00004201, target_pid: 0x00021581, addr: 0x00000000, data: 0x55cb6e5fa148
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x0000420e, target_pid: 0x00021581, addr: 0x00000058, data: 0x55cb66f323e0
ptrace ENTER (tid=136581 pid=136581 comm=strace): request: 0x00000018, target_pid: 0x00021581, addr: 0x00000000, data: 0x00000000
```
</details>

