# 5 [BX]和loop指令

## 5.1 约定符号()

首先，定义一个符号`()`来表示一个寄存器/内存单元中的内容，例如`(ax)`表示寄存器`ax`中的内容，`(2000H)`表示内存单元`2000H`中的内容。

在以后的内容中，为方便描述，都使用`()`来描述一个内存/寄存器中的内容。

## 5.2 [BX]

在前面的学习中，已经了解到 `MOV AX,[1000]`中的 `[1000]` 表示 `ds:1000H`内存单元中内容，即`((ds)*16+1000)`。

同理，`[BX]`表示 `((ds)*16+(bx))`。

## 5.3 loop指令

Loop指令在汇编中，用于控制循环，相当于高级语言中的`for`。

CPU在执行`loop`指令时，要进行两步操作：

1. `(cx) = (cx) - 1`；

2. 判断`(cx)`是否等于0，如果为0则向下执行，否则转到标号处继续执行。

例如，使用`loop` 指令实现`2^12`计算：

```masm
assume cs:codesg

codesg segment
    mov ax,2
    mov cx,11
s:  add ax,ax
    loop s   #(cs)=(cs-1),cs==0?

    mov ax,4c00H
    int 21H
codesg ends

end

```

## 5.3 debug loop指令

在loop指令debug时，可以使用指令来快速跳过循环：

1. `p` 指令，会自动重复执行loop标号中的指令，知道`(cx)==0`;

2. `g ip`指令，让debug指令快速执行到`cs:ip`处。

## 5.4 debug 和masm对内存访问指令的不同

在debug中，`mov ax,[0]` 表示`((ds)*16+0)`；但是，在masm中，却被解释为`mov ax,0`。造成这一差异的原因是不同编译器对指令的解释不同造成的。

在masm中，相实现类似`mov ax,[0]`的功能，有两种方法：

1. 使用 `[bx]`：

```masm
mov ax,2000h
mov ds,ax
mov bx,0
mov al.[bx]
```

2. 显示指定`ds`：

```masm
mov ax,2000h
mov ds,ax
mov ds:[0]
```

在显示指定ds的例子中，ds被称为是**段前缀**。可以作为段前缀的例子有：CS，DS，ES，SS。

## 5.5 实验：使用loop和[bx]

直接贴代码：

（1）向内存0:200～023F依此写入0～63
（2）在（1）基础上，只能使用9行代码

```masm
assume cs:codesg

codesg segment
        mov ax,0
        mov ds,ax
        mov cx,3fh

      s:mov bx,200h
        add bx,cx     ;复用loop中cx寄存器的递减
        mov ds:[bx],cl
        loop s
        mov ax,4c00h
        int 21h
codesg ends

end
```
