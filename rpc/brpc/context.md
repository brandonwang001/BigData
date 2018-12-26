```
#if defined(BTHREAD_CONTEXT_PLATFORM_linux_x86_64) && defined(BTHREAD_CONTEXT_COMPILER_gcc)
__asm (
@.text为代码段
".text\n"
@.global是符号bthread_make_fcontext对连接器可见
".globl bthread_make_fcontext\n"
".type bthread_make_fcontext,@function\n"
@16字节对齐
".align 16\n"
@label
"bthread_make_fcontext:\n"
@rdi : register destination index (destination for data copies)
@
@rax : register a extended
@temporary register; when we call a syscal, rax must contain syscall number
@moveq : 
"    movq  %rdi, %rax\n"
"    andq  $-16, %rax\n"
"    leaq  -0x48(%rax), %rax\n"
"    movq  %rdx, 0x38(%rax)\n"
"    stmxcsr  (%rax)\n"
"    fnstcw   0x4(%rax)\n"
"    leaq  finish(%rip), %rcx\n"
"    movq  %rcx, 0x40(%rax)\n"
"    ret \n"
"finish:\n"
"    xorq  %rdi, %rdi\n"
"    call  _exit@PLT\n"
"    hlt\n"
".size bthread_make_fcontext,.-bthread_make_fcontext\n"
".section .note.GNU-stack,\"\",%progbits\n"
);
```
