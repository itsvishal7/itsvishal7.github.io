# Random facts about Linux kernel

## quick tags
[#syscallnr](#system-call-numbers)

asfdadsfasdfasf


adsfasdf

asfadsf

asfasfdas

fa

sdfasf

asdf

safd

sadfa

sf

asdf

as

fas

f



adsf

as

fas

fa

sdf

af

a

sdf

saf

asfd
as
fa
sf
asf
sadf
sa
fsa
f
asf
as
fsa
as
asdfadsf
## System Call Numbers
- **System calls** are how user space requests services from the kernel (e.g., opening a file, 
  allocating memory, etc.). Different architectures maintain their own mapping of system call 
  numbers in `.tbl` files, like `arch/x86/entry/syscalls/syscall_64.tbl` or 
  `arch/powerpc/kernel/syscalls/syscall.tbl`.

  - During kernel compilation, these `.tbl` files get processed to auto-generate headers like 
    `arch/powerpc/include/generated/asm/syscall_table_64.h`. These headers keep a *machine-readable* 
    list of system call numbers.
