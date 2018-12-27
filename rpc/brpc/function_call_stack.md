# x86-64的函数调用


## 从一个main函数说起

我们下面通过TwoNumSum和ThreeNumSum的函数调用，
来分析函数调用过程中栈和寄存器值的变化。

```
#include <iostream>

int TwoNumSum(int a, int b) {
  int c = a + b;
  return c;
}


int ThreeNumSum(int a, int b, int c) {
  return TwoNumSum(TwoNumSum(a, b), c);
}

int main() {
  int a = 1;
  int b = 2;
  int c = 3;
  int d = ThreeNumSum(a, b, c);
  return 0;
}
```

上面代码对应的汇编代码(使用objdump -d反汇编获得)：

```
@第一列为相对地址
@第二列为机器码
@第三列为汇编指令

0000000000400700 <main>:
  400700: 55                    push   %rbp
  400701: 48 89 e5              mov    %rsp,%rbp
  400704: 48 83 ec 10           sub    $0x10,%rsp
  400708: c7 45 fc 01 00 00 00  movl   $0x1,-0x4(%rbp)
  40070f: c7 45 f8 02 00 00 00  movl   $0x2,-0x8(%rbp)
  400716: c7 45 f4 03 00 00 00  movl   $0x3,-0xc(%rbp)
  40071d: 8b 55 f4              mov    -0xc(%rbp),%edx
  400720: 8b 4d f8              mov    -0x8(%rbp),%ecx
  400723: 8b 45 fc              mov    -0x4(%rbp),%eax
  400726: 89 ce                 mov    %ecx,%esi
  400728: 89 c7                 mov    %eax,%edi
  40072a: e8 a1 ff ff ff        callq  4006d0 <_Z11ThreeNumSumiii>
  40072f: 89 45 f0              mov    %eax,-0x10(%rbp)
  400732: b8 00 00 00 00        mov    $0x0,%eax
  400737: c9                    leaveq
  400738: c3                    retq

0000000000400776 <_GLOBAL__sub_I__Z9TwoNumSumii>:
  400776: 55                    push   %rbp
  400777: 48 89 e5              mov    %rsp,%rbp
  40077a: be ff ff 00 00        mov    $0xffff,%esi
  40077f: bf 01 00 00 00        mov    $0x1,%edi
  400784: e8 b0 ff ff ff        callq  400739 <_Z41__static_initialization_and_destruction_0ii>
  400789: 5d                    pop    %rbp
  40078a: c3                    retq
  40078b: 0f 1f 44 00 00        nopl   0x0(%rax,%rax,1)

00000000004006d0 <_Z11ThreeNumSumiii>:
  4006d0: 55                    push   %rbp
  4006d1: 48 89 e5              mov    %rsp,%rbp
  4006d4: 48 83 ec 10           sub    $0x10,%rsp
  4006d8: 89 7d fc              mov    %edi,-0x4(%rbp)
  4006db: 89 75 f8              mov    %esi,-0x8(%rbp)
  4006de: 89 55 f4              mov    %edx,-0xc(%rbp)
  4006e1: 8b 55 f8              mov    -0x8(%rbp),%edx
  4006e4: 8b 45 fc              mov    -0x4(%rbp),%eax
  4006e7: 89 d6                 mov    %edx,%esi
  4006e9: 89 c7                 mov    %eax,%edi
  4006eb: e8 c6 ff ff ff        callq  4006b6 <_Z9TwoNumSumii>
  4006f0: 89 c2                 mov    %eax,%edx
  4006f2: 8b 45 f4              mov    -0xc(%rbp),%eax
  4006f5: 89 c6                 mov    %eax,%esi
  4006f7: 89 d7                 mov    %edx,%edi
  4006f9: e8 b8 ff ff ff        callq  4006b6 <_Z9TwoNumSumii>
  4006fe: c9                    leaveq
  4006ff: c3                    retq
```

通过观察上面的汇编代码不难发现有下面类似的结构:

```
<function name>:
    push %rbp
    move %rsp,%rbp
    ...
    ...
    leaveq
    req
```
