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
@rax : register a extended
@moveq : 8字节复制(quardword)
@将rdi中的内容复制到rax中，8字节操作。
"    movq  %rdi, %rax\n"
@andq : 8字节位与。
@ -16 : 16 1000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0001 0000
@     :取反 0111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1110 1111
@     :+1. 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 0000
@与-16位与，即对16求余。
@所以是将对rax存储的栈指针进行调整，16字节对齐。
"    andq  $-16, %rax\n"
@rax的值减去72字节，放入到rax中
"    leaq  -0x48(%rax), %rax\n"
@将rdx的值放入到rax值+56字节。
@bthread_make_fcontext(void* sp, size_t size, void (* fn)( intptr_t));
@%rdi，%rsi，%rdx，%rcx，%r8，%r9 用作函数参数，依次对应第1参数，第2参数
"    movq  %rdx, 0x38(%rax)\n"
%将mxcsr的状态存储到rax指向的内存
"    stmxcsr  (%rax)\n"
%保存控制器到rax+4指向的内存
"    fnstcw   0x4(%rax)\n"
%指令指针寄存器+finish存储到rcx。
"    leaq  finish(%rip), %rcx\n"
%将rcx的内容移动到rax+64.
"    movq  %rcx, 0x40(%rax)\n"
"    ret \n"
"finish:\n"
@清空rdi
"    xorq  %rdi, %rdi\n"
"    call  _exit@PLT\n"
"    hlt\n"
".size bthread_make_fcontext,.-bthread_make_fcontext\n"
".section .note.GNU-stack,\"\",%progbits\n"
);
```
```
8字节对齐
1、rax按字节对齐调整
2、rax-72
3、rax+56
4、将第三个参数函数指针fn放入到rax+56指向的内存
5、将寄存器状态保存到rax指向的内存。

+--------------+ 高地址
+              +
+--------------+
+              +
+--------------+ <- rax(step1) 
+              + 
+--------------+ 
+              +   将finish地址存储在这里。
+--------------+<- rax(step3)
+              +   这里存函数指针fn，第三个参数
+--------------+
+              +
+--------------+ 
+              +
+--------------+
+              +
+--------------+
+              +
+--------------+
+              +
+--------------+
+              +  控制寄存器的状态存储到低4个自己。
+--------------+<- rax(step2)
+              +  mxcsr的状态，占用高地址的4个字节。
+--------------+
+              +
+--------------+
+              +
+--------------+
+              +
+--------------+
+              +
+--------------+

```

```
#if defined(BTHREAD_CONTEXT_PLATFORM_linux_x86_64) && defined(BTHREAD_CONTEXT_COMPILER_gcc)
__asm (
".text\n"
".globl bthread_jump_fcontext\n"
".type bthread_jump_fcontext,@function\n"
".align 16\n"
"bthread_jump_fcontext:\n"
"    pushq  %rbp  \n"
"    pushq  %rbx  \n"
"    pushq  %r15  \n"
"    pushq  %r14  \n"
"    pushq  %r13  \n"
"    pushq  %r12  \n"
"    leaq  -0x8(%rsp), %rsp\n"
"    cmp  $0, %rcx\n"
"    je  1f\n"
"    stmxcsr  (%rsp)\n"
"    fnstcw   0x4(%rsp)\n"
"1:\n"
"    movq  %rsp, (%rdi)\n"
"    movq  %rsi, %rsp\n"
"    cmp  $0, %rcx\n"
"    je  2f\n"
"    ldmxcsr  (%rsp)\n"
"    fldcw  0x4(%rsp)\n"
"2:\n"
"    leaq  0x8(%rsp), %rsp\n"
"    popq  %r12  \n"
"    popq  %r13  \n"
"    popq  %r14  \n"
"    popq  %r15  \n"
"    popq  %rbx  \n"
"    popq  %rbp  \n"
"    popq  %r8\n"
"    movq  %rdx, %rax\n"
"    movq  %rdx, %rdi\n"
"    jmp  *%r8\n"
".size bthread_jump_fcontext,.-bthread_jump_fcontext\n"
".section .note.GNU-stack,\"\",%progbits\n"
);
```
