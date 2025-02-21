### ⏰🥢 CONFIG_NO_HZ 

- <ins>dynticks-idle CPUs</ins>: These are the CPUs that are not sent any scheduling-clock interrupts by the kernel, built with CONFIG_NO_HZ=y, if they are idle. \
- <ins>adaptive-ticks CPUs</ins>: These are the CPUs that are not sent any scheduling-clock interrupts by the kernel, built with CONFIG_NO_HZ_FULL=y, if they are running only a single runnable task.

### ⚾ Add base commit info in your patches using `git format-patch`
[checkout base tree info](https://git-scm.com/docs/git-format-patch#_base_tree_information)

1. set your branch to track upstream branch `git branch --set-upstream-to=master`
2. create patch using `git format-patch --base=auto -s -1`
