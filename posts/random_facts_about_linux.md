# Random facts about Linux kernel

## quick tags
[#syscallnr](#system-call-numbers)

## System Call Numbers
- **System calls** are how user space requests services from the kernel (e.g., opening a file, 
  allocating memory, etc.). Different architectures maintain their own mapping of system call 
  numbers in `.tbl` files, like `arch/x86/entry/syscalls/syscall_64.tbl` or 
  `arch/powerpc/kernel/syscalls/syscall.tbl`.

  - During kernel compilation, these `.tbl` files get processed to auto-generate headers like 
    `arch/powerpc/include/generated/asm/syscall_table_64.h`. These headers keep a *machine-readable* 
    list of system call numbers.
