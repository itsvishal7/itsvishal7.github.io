# Random facts about Linux kernel 😊

## quick tags ✨
[#syscallnr](#-system-call-numbers-and-where-to-find-them) 🚀

## 🔢 System Call Numbers and Where to find them
tl;dr `arch/*/include/generated/asm/syscall*.h` and `arch/*/**/syscall*.tbl`
- **System calls** let user space request kernel services (e.g. file I/O, memory allocation).
- Different architectures map system call numbers in `.tbl` files (e.g. `arch/x86/entry/syscalls/syscall_64.tbl` or `arch/powerpc/kernel/syscalls/syscall.tbl`).
- During compilation, these files auto-generate headers like `arch/powerpc/include/generated/asm/syscall_table_64.h` that list system call numbers in a machine-readable format. 😎
