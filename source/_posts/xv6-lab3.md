---
title: page tables [os lab3]
date: 2020-12-19 21:22:11
index_img: /img/xv6lab.jpg
tags: [Operating System, xv6]
categories: Operating System
---

# Print a page table

To help you learn about RISC-V page tables, and perhaps to aid future debugging, your first task is to write a function that prints the contents of a page table.

{% note primary%}

Define a function called `vmprint()`. It should take a `pagetable_t` argument, and print that pagetable in the format described below. Insert `if(p->pid==1) vmprint(p->pagetable)` in exec.c just before the `return argc`, to print the first process's page table. You receive full credit for this assignment if you pass the `pte printout` test of `make grade`.

{% endnote %}

Now when you start xv6 it should print output like this, describing the page table of the first process at the point when it has just finished `exec()`ing `init`:

```
page table 0x0000000087f6e000
..0: pte 0x0000000021fda801 pa 0x0000000087f6a000
.. ..0: pte 0x0000000021fda401 pa 0x0000000087f69000
.. .. ..0: pte 0x0000000021fdac1f pa 0x0000000087f6b000
.. .. ..1: pte 0x0000000021fda00f pa 0x0000000087f68000
.. .. ..2: pte 0x0000000021fd9c1f pa 0x0000000087f67000
..255: pte 0x0000000021fdb401 pa 0x0000000087f6d000
.. ..511: pte 0x0000000021fdb001 pa 0x0000000087f6c000
.. .. ..510: pte 0x0000000021fdd807 pa 0x0000000087f76000
.. .. ..511: pte 0x0000000020001c0b pa 0x0000000080007000
  
```

The first line displays the argument to `vmprint`. After that there is a line for each PTE, including PTEs that refer to page-table pages deeper in the tree. Each PTE line is indented by a number of `" .."` that indicates its depth in the tree. Each PTE line shows the PTE index in its page-table page, the pte bits, and the physical address extracted from the PTE. Don't print PTEs that are not valid. In the above example, the top-level page-table page has mappings for entries 0 and 255. The next level down for entry 0 has only index 0 mapped, and the bottom-level for that index 0 has entries 0, 1, and 2 mapped.

Your code might emit different physical addresses than those shown above. The number of entries and the virtual addresses should be the same.

## Some hints

{% note success%}

- You can put `vmprint()` in `kernel/vm.c`.

{% endnote %}

{% note success%}

- Use the macros at the end of the file `kernel/riscv.h`.

{% endnote %}

{% note success%}

- The function `freewalk` may be inspirational.

{% endnote %}

??????????????????????????????recursively??????page table

```c
void
freewalk(pagetable_t pagetable)
{
  // there are 2^9 = 512 PTEs in a page table.
  for(int i = 0; i < 512; i++){
    pte_t pte = pagetable[i];
    if((pte & PTE_V) && (pte & (PTE_R|PTE_W|PTE_X)) == 0){
      // this PTE points to a lower-level page table.
      uint64 child = PTE2PA(pte);
      freewalk((pagetable_t)child);
      pagetable[i] = 0;
    } else if(pte & PTE_V){
      panic("freewalk: leaf");
    }
  }
  kfree((void*)pagetable);
}
```



{% note success%}

- Define the prototype for `vmprint` in `kernel/defs.h` so that you can call it from `exec.c`.

{% endnote %}

<img src="image-20201219212505432.png" alt="image-20201219212505432" style="zoom:80%;" />

{% note success%}

- Use `%p` in your printf calls to print out full 64-bit hex PTEs and addresses as shown in the example.

{% endnote %}



## Final Code

??????????????????????????????????????????`vmprintRecursive`???????????????page table, ????????????`.. `??????????????????????????????????????????level

????????????????????????????????????: ???0-511?????????page table entry

???????????????pte????????????valid???, ?????????`PTE2PA`???????????????physical address?????????????????????????????????????????????

???????????????????????????pte??????PTE_R, PTE_W, PTE_X??????????????????????????????pte??????????????????page table???physical address, ??????????????????

```c
void vmprintRecursive(pagetable_t pagetable, int level) {
  for(int i = 0; i < 512; i++){
    pte_t pte = pagetable[i];
    if(pte & PTE_V) {
      uint64 child = PTE2PA(pte);
      for(int i = 0; i <= level; i++) {
        printf("..");
        if(i != level) printf(" ");
      }

      printf("%d: pte %p pa %p\n", i, pte, child);
      if((pte & (PTE_R | PTE_W | PTE_X)) == 0)
          vmprintRecursive((pagetable_t)child, level + 1);
    } 
  }
}

void
vmprint(pagetable_t pagetable) {
  printf("page table %p\n", pagetable);
  vmprintRecursive(pagetable, 0);
}
```



# A kernel page table per process

{% note primary%}

Xv6 has a single kernel page table that's used whenever it executes in the kernel. The kernel page table is a direct mapping to physical addresses, so that kernel virtual address *x* maps to physical address *x*. Xv6 also has a separate page table for each process's user address space, containing only mappings for that process's user memory, starting at virtual address zero. Because the kernel page table doesn't contain these mappings, user addresses are not valid in the kernel. Thus, when the kernel needs to use a user pointer passed in a system call (e.g., the buffer pointer passed to `write()`), the kernel must first translate the pointer to a physical address. The goal of this section and the next is to allow the kernel to directly dereference user pointers.



Xv6 has a single kernel page table that's used whenever it executes in the kernel. The kernel page table is a direct mapping to physical addresses, so that kernel virtual address *x* maps to physical address *x*. Xv6 also has a separate page table for each process's user address space, containing only mappings for that process's user memory, starting at virtual address zero. Because the kernel page table doesn't contain these mappings, user addresses are not valid in the kernel. Thus, when the kernel needs to use a user pointer passed in a system call (e.g., the buffer pointer passed to `write()`), the kernel must first translate the pointer to a physical address. The goal of this section and the next is to allow the kernel to directly dereference user pointers.

{% endnote %}



?????????????????????????????????`vm.c, proc.h, proc.c, defs.h`

## Some hints

{%note success%}

- Add a field to `struct proc` for the process's kernel page table.

{% endnote %}

<img src="image-20201222200731803.png" alt="image-20201222200731803" style="zoom: 50%;" />

{% note success %}

- A reasonable way to produce a kernel page table for a new process is to implement a modified version of `kvminit` that makes a new page table instead of modifying `kernel_pagetable`. You'll want to call this function from `allocproc`.

{% endnote %}

`allocproc`???????????????????????????struct proc, ???????????????struct proc????????????kernel page table, ???????????????????????????????????????

???`kernel/vm.c`???`kvminit`???????????????direct-map kernel page table??????`kernel/proc.c`?????????`kvminit`??????????????????`proc_kpagetable`???????????????process??????kernel page table

```c
pagetable_t
proc_kpagetable(struct proc *p)
{
  pagetable_t kpagetable;

  kpagetable = uvmcreate();
  if (kpagetable == 0)
    return 0;
  
  // Fill in the process's kernel page table, the same as kernel_pagetable
  ukvmmap(kpagetable, UART0, UART0, PGSIZE, PTE_R | PTE_W);
  ukvmmap(kpagetable, VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);
  ukvmmap(kpagetable, PLIC, PLIC, 0x400000, PTE_R | PTE_W);
  ukvmmap(kpagetable, KERNBASE, KERNBASE, (uint64)etext-KERNBASE, PTE_R | PTE_X);
  ukvmmap(kpagetable, (uint64)etext, (uint64)etext, PHYSTOP-(uint64)etext, PTE_R | PTE_W);
  ukvmmap(kpagetable, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);
  ukvmmap(kpagetable, TRAPFRAME, (uint64)(p->trapframe), PGSIZE, PTE_R | PTE_W);
  return kpagetable;
}
```

???`kernel/proc.c`???`allocproc`???????????????`proc_kpagetable`??????

```c
  // An empty user page table.
  p->pagetable = proc_pagetable(p);
  if(p->pagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  p->kpagetable = proc_kpagetable(p);
  if(p->kpagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }
```



{% note success %}

- Make sure that each process's kernel page table has a mapping for that process's kernel stack. In unmodified xv6, all the kernel stacks are set up in `procinit`. You will need to move some or all of this functionality to `allocproc`.

{% endnote %}

??????`kernel/proc.c`??????`procinit`??????, ???????????????kernel stack?????????

```c
void
procinit(void)
{
  struct proc *p;
  
  initlock(&pid_lock, "nextpid");
  for(p = proc; p < &proc[NPROC]; p++) {
      initlock(&p->lock, "proc");
      // Allocate a page for the process's kernel stack.
      // Map it high in memory, followed by an invalid
      // guard page.
      //char *pa = kalloc();
      //if(pa == 0)
      //    panic("kalloc");
      //uint64 va = KSTACK((int) (p - proc));
      //kvmmap(p->kpagetable, va, (uint64)pa, PGSIZE, PTE_R | PTE_W);
      //p->kstack = va;
  }
  kvminithart();
}
```

???????????????????????????`allocproc`???

```c
  // Allocate a page for the process's kernel stack.
  // Map it high in memory, followed by an invalid
  // guard page.
  char *pa = kalloc();
  if(pa == 0)
    panic("kalloc");
  uint64 va = KSTACK((int) (p - proc));
  ukvmmap(p->kpagetable, va, (uint64)pa, PGSIZE, PTE_R | PTE_W);
  p->kstack = va;
```

??????: ?????????`kvmmap`???????????????`ukvmmap`??????, `kvmmap`??????????????????`kernel_pagetable`, ????????????????????????process???`kpagetable`?????????????????? ?????????????????????`ukvmmap`???????????????????????????`pagetable`?????????

```c
// add a mapping to the kernel page table.
// only used when booting.
// does not flush TLB or enable paging.
void
kvmmap(uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kernel_pagetable, va, sz, pa, perm) != 0)
    panic("kvmmap");
}

void
ukvmmap(pagetable_t kpagetable, uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kpagetable, va, sz, pa, perm) != 0)
    panic("ukvmmap");
}
```

{% note success %}

- Modify `scheduler()` to load the process's kernel page table into the core's `satp` register (see `kvminithart` for inspiration). Don't forget to call `sfence_vma()` after calling `w_satp()`.

{% endnote %}

<img src="image-20201222202345524.png" alt="image-20201222202345524" style="zoom:50%;" />

{% note success %}

- `scheduler()` should use `kernel_pagetable` when no process is running.

{% endnote %}

<img src="image-20201222202559087.png" alt="image-20201222202559087" style="zoom:50%;" />

{% note warning %}

??????, ???`kernel/vm.c`??????`kvmpa`?????????????????????????????????????????????????????????????????????kernel page table, ???????????????????????????kernel page table

{% endnote %}

```c
// translate a kernel virtual address to
// a physical address. only needed for
// addresses on the stack.
// assumes va is page aligned.
uint64
kvmpa(uint64 va)
{
  uint64 off = va % PGSIZE;
  pte_t *pte;
  uint64 pa;
  
  pte = walk(myproc()->kpagetable, va, 0);
  if(pte == 0)
    panic("kvmpa");
  if((*pte & PTE_V) == 0)
    panic("kvmpa");
  pa = PTE2PA(*pte);
  return pa+off;
}
```

????????????`myproc()`?????? ???????????????????????????

```c
#include "spinlock.h"
#include "proc.h"
```



{% note success %}

- Free a process's kernel page table in `freeproc`.
- You'll need a way to free a page table without also freeing the leaf physical memory pages.

{% endnote %}

??????`kernel/proc.c`??????`freeproc`?????????free kernel stack, free kernel page table

* free kstack?????????kpage table?????????kstack??????????????????????????????free
* free kernel page table?????????`kernel/proc.c`????????????????????????`proc_freekpagetable`?????????????????????????????????`kernel/vm.c`??????`freewalk`?????? ??????????????????free kernel page table, ?????????page table???????????????free?????????????????????free leaves

```c
void
proc_freekpagetable(pagetable_t kpagetable)
{
  // there are 2^9 = 512 PTEs in a page table.
  for(int i = 0; i < 512; i++) {
    pte_t pte = kpagetable[i];
    if(pte & PTE_V) {
      kpagetable[i] = 0;

      if((pte & (PTE_R | PTE_W | PTE_X)) == 0) {
        uint64 child = PTE2PA(pte);
        proc_freekpagetable((pagetable_t)child);
      }
    }
  }
  kfree((void*)kpagetable);
}
```



```c
// free a proc structure and the data hanging from it,
// including user pages.
// p->lock must be held.
static void
freeproc(struct proc *p)
{
  if(p->trapframe)
    kfree((void*)p->trapframe);
  p->trapframe = 0;
  // free kernel stack
  if(p->kstack) {
    pte_t* pte = walk(p->kpagetable, p->kstack, 0);
    if (pte == 0)
      panic("freeproc: walk");
    kfree((void*)PTE2PA(*pte));
  }
  p->kstack = 0;

  if(p->pagetable)
    proc_freepagetable(p->pagetable, p->sz);

  if(p->kpagetable)
    proc_freekpagetable(p->kpagetable);
  p->pagetable = 0;
  p->kpagetable = 0;
  p->sz = 0;
  p->pid = 0;
  p->parent = 0;
  p->name[0] = 0;
  p->chan = 0;
  p->killed = 0;
  p->xstate = 0;
  p->state = UNUSED;
}
```

??????????????????free kstack????????????free pagetable, ???free kstack, ????????????error

{% note success %}

- `vmprint` may come in handy to debug page tables.

{% endnote %}

{% note success %}

- It's OK to modify xv6 functions or add new functions; you'll probably need to do this in at least `kernel/vm.c` and `kernel/proc.c`. (But, don't modify `kernel/vmcopyin.c`, `kernel/stats.c`, `user/usertests.c`, and `user/stats.c`.)

{% endnote %}

{% note success %}

- A missing page table mapping will likely cause the kernel to encounter a page fault. It will print an error that includes `sepc=0x00000000XXXXXXXX`. You can find out where the fault occurred by searching for `XXXXXXXX` in `kernel/kernel.asm`.

{% endnote %}

## Final Code

* ???`kernel/defs.h`??????????????????????????????

{% gi 2 2%}
  ![pic1](image-20201222204213555.png)
  ![pic1](image-20201222204251376.png)

{% endgi %}

* `kernel/proc.h`

```c
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  struct proc *parent;         // Parent process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t kpagetable;      // Kernel page table
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};

```

* `kernel/vm.c`

```c
#include "spinlock.h"
#include "proc.h"
```

```c
void
ukvmmap(pagetable_t kpagetable, uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kpagetable, va, sz, pa, perm) != 0)
    panic("ukvmmap");
}
```

```c
uint64
kvmpa(uint64 va)
{
  uint64 off = va % PGSIZE;
  pte_t *pte;
  uint64 pa;
  
  pte = walk(myproc()->kpagetable, va, 0);
  if(pte == 0)
    panic("kvmpa");
  if((*pte & PTE_V) == 0)
    panic("kvmpa");
  pa = PTE2PA(*pte);
  return pa+off;
}
```

* `kernel/proc.c`

```c
extern char etext[];  // kernel.ld sets this to end of kernel code.
extern pagetable_t kernel_pagetable;
```

```c
void
procinit(void)
{
  struct proc *p;
  
  initlock(&pid_lock, "nextpid");
  for(p = proc; p < &proc[NPROC]; p++) {
      initlock(&p->lock, "proc");
  }
  kvminithart();
}
```

```c
static struct proc*
allocproc(void)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    acquire(&p->lock);
    if(p->state == UNUSED) {
      goto found;
    } else {
      release(&p->lock);
    }
  }
  return 0;

found:
  p->pid = allocpid();

  // Allocate a trapframe page.
  if((p->trapframe = (struct trapframe *)kalloc()) == 0){
    release(&p->lock);
    return 0;
  }

  // An empty user page table.
  p->pagetable = proc_pagetable(p);
  if(p->pagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  p->kpagetable = proc_kpagetable(p);
  if(p->kpagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }


  // Allocate a page for the process's kernel stack.
  // Map it high in memory, followed by an invalid
  // guard page.
  char *pa = kalloc();
  if(pa == 0)
    panic("kalloc");
  uint64 va = KSTACK((int) (p - proc));
  ukvmmap(p->kpagetable, va, (uint64)pa, PGSIZE, PTE_R | PTE_W);
  p->kstack = va;

  // Set up new context to start executing at forkret,
  // which returns to user space.
  memset(&p->context, 0, sizeof(p->context));
  p->context.ra = (uint64)forkret;
  p->context.sp = p->kstack + PGSIZE;

  return p;
}
```

```c
static void
freeproc(struct proc *p)
{
  if(p->trapframe)
    kfree((void*)p->trapframe);
  p->trapframe = 0;
  // free kernel stack
  if(p->kstack) {
    pte_t* pte = walk(p->kpagetable, p->kstack, 0);
    if (pte == 0)
      panic("freeproc: walk");
    kfree((void*)PTE2PA(*pte));
  }
  p->kstack = 0;

  if(p->pagetable)
    proc_freepagetable(p->pagetable, p->sz);

  if(p->kpagetable)
    proc_freekpagetable(p->kpagetable);
  p->pagetable = 0;
  p->kpagetable = 0;
  p->sz = 0;
  p->pid = 0;
  p->parent = 0;
  p->name[0] = 0;
  p->chan = 0;
  p->killed = 0;
  p->xstate = 0;
  p->state = UNUSED;
}
```

```c
pagetable_t
proc_kpagetable(struct proc *p)
{
  pagetable_t kpagetable;

  kpagetable = uvmcreate();
  if (kpagetable == 0)
    return 0;
  
  // Fill in the process's kernel page table, the same as kernel_pagetable
  ukvmmap(kpagetable, UART0, UART0, PGSIZE, PTE_R | PTE_W);
  ukvmmap(kpagetable, VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);
  ukvmmap(kpagetable, PLIC, PLIC, 0x400000, PTE_R | PTE_W);
  ukvmmap(kpagetable, KERNBASE, KERNBASE, (uint64)etext-KERNBASE, PTE_R | PTE_X);
  ukvmmap(kpagetable, (uint64)etext, (uint64)etext, PHYSTOP-(uint64)etext, PTE_R | PTE_W);
  ukvmmap(kpagetable, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);
  ukvmmap(kpagetable, TRAPFRAME, (uint64)(p->trapframe), PGSIZE, PTE_R | PTE_W);
  return kpagetable;
}
```

```c
void
proc_freekpagetable(pagetable_t kpagetable)
{
  // there are 2^9 = 512 PTEs in a page table.
  for(int i = 0; i < 512; i++) {
    pte_t pte = kpagetable[i];
    if(pte & PTE_V) {
      kpagetable[i] = 0;

      if((pte & (PTE_R | PTE_W | PTE_X)) == 0) {
        uint64 child = PTE2PA(pte);
        proc_freekpagetable((pagetable_t)child);
      }
    }
  }
  kfree((void*)kpagetable);
}
```

```c
void
scheduler(void)
{
  struct proc *p;
  struct cpu *c = mycpu();
  
  c->proc = 0;
  for(;;){
    // Avoid deadlock by ensuring that devices can interrupt.
    intr_on();
    
    int found = 0;
    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Switch to chosen process.  It is the process's job
        // to release its lock and then reacquire it
        // before jumping back to us.
        p->state = RUNNING;
        c->proc = p;

        // load process's kernel page table 
        w_satp(MAKE_SATP(p->kpagetable));
        sfence_vma();

        swtch(&c->context, &p->context);

        // Process is done running for now.
        // It should have changed its p->state before coming back.
        c->proc = 0;

        found = 1;
      }
      release(&p->lock);
    }
#if !defined (LAB_FS)
    if(found == 0) {
      w_satp(MAKE_SATP(kernel_pagetable));
      sfence_vma();
      intr_on();
      asm volatile("wfi");
    }
#else
    ;
#endif
  }
}
```





# Simplify `copyin/copyinstr`

{% note primary %}

The kernel's `copyin` function reads memory pointed to by user pointers. It does this by translating them to physical addresses, which the kernel can directly dereference. It performs this translation by walking the process page-table in software. Your job in this part of the lab is to add user mappings to each process's kernel page table (created in the previous section) that allow`copyin` (and the related string function `copyinstr`) to directly dereference user pointers.

Replace the body of `copyin` in `kernel/vm.c` with a call to `copyin_new` (defined in `kernel/vmcopyin.c`); do the same for `copyinstr` and `copyinstr_new`. Add mappings for user addresses to each process's kernel page table so that `copyin_new` and `copyinstr_new` work. You pass this assignment if `usertests`runs correctly and all the `make grade` tests pass.

This scheme relies on the user virtual address range not overlapping the range of virtual addresses that the kernel uses for its own instructions and data. Xv6 uses virtual addresses that start at zero for user address spaces, and luckily the kernel's memory starts at higher addresses. However, this scheme does limit the maximum size of a user process to be less than the kernel's lowest virtual address. After the kernel has booted, that address is `0xC000000` in xv6, the address of the PLIC registers; see `kvminit()` in `kernel/vm.c`, `kernel/memlayout.h`, and Figure 3-4 in the text. You'll need to modify xv6 to prevent user processes from growing larger than the PLIC address.

{% endnote %}



?????????kernel page table????????????translate user space address, ?????????user pagetable??????????????????kernel pagetable??????

???????????????????????????

* ?????????user pagetable??????????????????kernel page table (??????`ukvmcopy`???????????????????????????)
* ???????????????????????????/?????????address mapping, ?????????user pagetable??????????????????kernel page table (`userinit`, `fork`, `exec`, `sbrk`)



## Some hints

{%note success%}

- Replace `copyin()` with a call to `copyin_new` first, and make it work, before moving on to `copyinstr`.

{% endnote %}

??????`kernel/vm.c`??????`copyin`???`copyin_new`??????

<img src="image-20201227210648285.png" alt="image-20201227210648285" style="zoom:80%;" />

{% note success%}

- At each point where the kernel changes a process's user mappings, change the process's kernel page table in the same way. Such points include `fork()`, `exec()`, and `sbrk()`.
- What permissions do the PTEs for user addresses need in a process's kernel page table? (A page with `PTE_U` set cannot be accessed in kernel mode.)
- Don't forget about the above-mentioned PLIC limit

{%endnote%}

???????????????????????????user space???????????????????????????kernel page table????????????`kernel/vm.c`?????????????????????`ukvmcopy`????????????????????????

???user space???virtual address??????????????????????????????`walk`?????????pagetable????????????????????????page table entry, ?????????kernel page table????????????page????????????kernel page table entry???????????????flag??????????????????PTE_U????????????kernel mode????????????????????????

```c
void ukvmcopy(pagetable_t pagetable, pagetable_t kpagetable, uint64 beginsz, uint64 endsz) {
  pte_t *upte, *kpte;
  uint64 pa;
  uint flags;
  
  if(beginsz > endsz) return;
  for(uint64 i = beginsz; i < endsz; i += PGSIZE) {
    if((upte = walk(pagetable, i, 0)) == 0)
      panic("ukvmcopy: pte should exist");
    if((kpte = walk(kpagetable, i, 1)) == 0)
      panic("ukvmcopy: walk fails");
    pa = PTE2PA(*upte);
    flags = (PTE_FLAGS(*upte) & (~PTE_U));
    *kpte = PA2PTE(pa) | flags;
  }
}
```



????????????`fork`, `exec`, `sbrk`?????????????????????????????????user page table??????????????????kernel page table.

??????`fork()`, ??????`kernel/proc.c`??????`fork`??????

<img src="image-20201227203900952.png" alt="image-20201227203900952" style="zoom:80%;" />

??????`exec`, ??????`kernel/exec.c`??????`exec`??????

<img src="image-20201227205027041.png" alt="image-20201227205027041" style="zoom:80%;" />

??????`sbrk`

`kernel/sysproc.c`??????`sys_sbrk`??????????????????

```c
uint64
sys_sbrk(void)
{
  int addr;
  int n;

  if(argint(0, &n) < 0)
    return -1;
  addr = myproc()->sz;
  if(growproc(n) < 0)
    return -1;
  return addr;
}
```

??????????????????`kernel/proc.c`???`growproc`????????????????????? ?????????????????????`growproc`???

`growproc`?????????????????????????????????????????????PLIC???

??????`ukvmcopy`???????????????????????????`sz`?????????????????????n bytes?????????`sz`

<img src="image-20201227210053052.png" alt="image-20201227210053052" style="zoom:80%;" />



{%note success%}

- Don't forget that to include the first process's user page table in its kernel page table in `userinit`.

{% endnote %}

??????????????????user process??????????????????user page table?????????kernel page table???

<img src="image-20201227210335814.png" alt="image-20201227210335814" style="zoom:80%;" />



????????????????????????`kernel/defs.h`?????????`ukvmcopy`, `copyin_new`, `copyinstr_new`??????

<img src="image-20201227211533018.png" alt="image-20201227211533018" style="zoom:80%;" />

# Reference

[1]https://github.com/gaofanfei/xv6-riscv-fall20/tree/pgtbl

[2]https://blog.csdn.net/u013577996/article/details/109582932

[3]https://zhuanlan.zhihu.com/p/336091300