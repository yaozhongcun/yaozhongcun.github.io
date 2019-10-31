---
layout: post
title:  "intel assembly初见"
date:   2019-10-27 08:14:12 +0800
categories: 技术 
---
## 序
开发过程总遇到一个奇怪的bug。 在定位过程中， 接触了下intel汇编。记录下。
## 崩溃
崩溃本身就是一个段错误。没有空指针，也没有明显的越界。请教了下组里的同事，用下面的gdb指令看了下汇编。

(gdb) info frame

(gdb) disassemble 


所以问题来了， 汇编应该怎么看？

## 汇编

汇编大学里还是学过一些的， 但是gcc编译出的汇编已经和教科书里的8086汇编大不一样了。他使用的是intel x64 assembly。

如果你有一些汇编的基础，只需要了解下面几点，就可以理解汇编代码
0. 首先汇编的语法区分AT&T， 及intel两种写法。
0. 对于AT&T， $1234, 代表了常量1234
0. %ebx 代表了寄存器
0. -0x4（%rbp）, ()表示取rbp寄存器里数值作为地址，前面的-0x4表示偏移4个字节。

<table class="wikitable">
<tbody><tr>
<th></th>
<th>AT&amp;T</th>
<th>Intel
</th></tr>
<tr>
<th scope="row">Parameter order
</th>
<td>Source before the destination. <div class="mw-highlight mw-content-ltr" dir="ltr"><pre><span></span><span class="nf">mov</span> <span class="no">$5</span><span class="p">,</span> <span class="nv">%eax</span>
</pre></div>
</td>
<td>Destination before source. <div class="mw-highlight mw-content-ltr" dir="ltr"><pre><span></span><span class="nf">mov</span> <span class="nb">eax</span><span class="p">,</span> <span class="mi">5</span>
</pre></div>
</td></tr>
<tr>
<th scope="row">Parameter size
</th>
<td>Mnemonics are suffixed with a letter indicating the size of the operands: <i>q</i> for qword, <i>l</i> for long (dword), <i>w</i> for word, and <i>b</i> for byte.<div class="mw-highlight mw-content-ltr" dir="ltr"><pre><span></span><span class="nf">addl</span> <span class="no">$4</span><span class="p">,</span> <span class="nv">%esp</span>
</pre></div>
</td>
<td>Derived from the name of the register that is used (e.g. <i>rax, eax, ax, al</i> imply <i>q, l, w, b</i>, respectively). <div class="mw-highlight mw-content-ltr" dir="ltr"><pre><span></span><span class="nf">add</span> <span class="nb">esp</span><span class="p">,</span> <span class="mi">4</span>
</pre></div>
</td></tr>
<tr>
<th scope="row">Sigils
</th>
<td>Immediate values prefixed with a "$", registers prefixed with a "%".
</td>
<td>The assembler automatically detects the type of symbols; i.e., whether they are registers, constants or something else.
</td></tr>
<tr>
<th scope="row">Effective addresses
</th>
<td>General syntax of <i>DISP(BASE,INDEX,SCALE)</i>. Example: <div class="mw-highlight mw-content-ltr" dir="ltr"><pre><span></span><span class="nf">movl</span> <span class="no">mem_location</span><span class="p">(</span><span class="nv">%ebx</span><span class="p">,</span><span class="nv">%ecx</span><span class="p">,</span><span class="mi">4</span><span class="p">),</span> <span class="nv">%eax</span>
</pre></div>
</td>
<td>Arithmetic expressions in square brackets; additionally, size keywords like <i>byte</i>, <i>word</i>, or <i>dword</i> have to be used if the size cannot be determined from the operands. Example: <div class="mw-highlight mw-content-ltr" dir="ltr"><pre><span></span><span class="nf">mov</span> <span class="nb">eax</span><span class="p">,</span> <span class="p">[</span><span class="nb">ebx</span> <span class="o">+</span> <span class="nb">ecx</span><span class="o">*</span><span class="mi">4</span> <span class="o">+</span> <span class="nv">mem_location</span><span class="p">]</span>
</pre></div>
</td></tr></tbody></table>


## 测试代码

```
.data
.globl hello
hello:
.string "Hi World\n"

.text
.global main
main:
    pushq   %rbp
    movq    %rsp,       %rbp
    mov     0x1c,%ebx
    movq    $hello,     %rdi
    call    puts
    movq    $0,         %rax
    leave
    ret
```


```
Program received signal SIGSEGV, Segmentation fault.
0x00000000004005f4 in main ()
Missing separate debuginfos, use: debuginfo-install glibc-2.17-157.tl2.3.x86_64 libgcc-4.8.5-4.el7.x86_64 libstdc++-4.8.5-4.el7.x86_64
(gdb) bt
#0  0x00000000004005f4 in main ()
(gdb) info frame
Stack level 0, frame at 0x7fffffffe3e0:
 rip = 0x4005f4 in main; saved rip 0x7ffff721cb35
 Arglist at 0x7fffffffe3d0, args: 
 Locals at 0x7fffffffe3d0, Previous frame's sp is 0x7fffffffe3e0
 Saved registers:
  rbp at 0x7fffffffe3d0, rip at 0x7fffffffe3d8
(gdb) info registers 

```

```
(gdb) disassemble 
Dump of assembler code for function main:
   0x00000000004005f0 <+0>:     push   %rbp
   0x00000000004005f1 <+1>:     mov    %rsp,%rbp
=> 0x00000000004005f4 <+4>:     mov    0x1c,%ebx
   0x00000000004005fb <+11>:    mov    $0x601034,%rdi
   0x0000000000400602 <+18>:    callq  0x4004e0 <puts@plt>
   0x0000000000400607 <+23>:    mov    $0x0,%rax
   0x000000000040060e <+30>:    leaveq 
   0x000000000040060f <+31>:    retq   
End of assembler dump.
```

### 结论

综合以上， 可以看到问题处的代码表示的是， mov 0x1c这个地址的内容到寄存器ebx。 0x1c这个地址明显是不合法的位置。

0x1c前面没有$,说明不是常量（之前怀疑是常量）。找了一些资料，也没有明确说是地址，但这里只有这样一个合理的解释。
所以此处应该是编译器优化，导致的一个bug。

后期优化了下代码，重新编译，解决了该崩溃。 
