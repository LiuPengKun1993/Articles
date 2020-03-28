> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 35 libffi动态调用和定义C函数

#### libffi 原理分析

[libffi](https://sourceware.org/libffi/) 中ffi的全称是 Foreign Function Interface(外部函数接口)，提供最底层的接口，在不确定参数个数和类型的情况下，根据相应规则，完成所需数据的准备，生成相应汇编指令的代码来完成函数调用。

libffi 还提供了可移植的高级语言接口，可以不使用函数签名间接调用 C 函数。比如，脚本语言 Python 在运行时会使用 libffi 高级语言的接口去调用 C 函数。libffi的作用类似于一个动态的编译器，在运行时就能够完成编译时所做的调用惯例函数调用代码生成。

libffi 通过调用 ffi_call(函数调用) 来进行函数调用，ffi_call 的输入是 ffi_cif(模板)、函数指针、参数地址。其中，ffi_cif 由 ffi_type(参数类型) 和参数个数生成，也可以是 ffi_closure(闭包)。

##### ffi_type(参数类型)

ffi_type的作用是，描述 C 语言的基本类型，比如 uint32、void *、struct 等，定义如下:

```
typedef struct _ffi_type
{
	size_t size; // 所占大小
	unsigned short alignment; //对⻬大小
	unsigned short type; // 标记类型的数字
	struct _ffi_type **elements; // 结构体中的元素
} ffi_type;
```

其中，size 表述该类型所占的大小，alignment 表示该类型的对齐大小，type 表示标记类型的数字，element 表示结构体的元素。

当类型是 uint32 时，size 的值是 4，alignment 也是 4，type 的值是 9，elements 是空。

##### ffi_cif(模板)

ffi_cif由参数类型(ffi_type) 和参数个数生成，定义如下:

```
typedef struct {
	ffi_abi abi; // 不同 CPU 架构下的 ABI，一般设置为 FFI_DEFAULT_ABI 
	unsigned nargs; // 参数个数
	ffi_type **arg_types; // 参数类型
	ffi_type *rtype; // 返回值类型
	unsigned bytes; // 参数所占空间大小，16的倍数
	unsigned flags; // 返回类型是结构体时要做的标记
  #ifdef FFI_EXTRA_CIF_FIELDS
    FFI_EXTRA_CIF_FIELDS;
  #endif
} ffi_cif;
```

如代码所示，ffi_cif 包含了函数调用时需要的一些信息。

abi 表示的是不同 CPU 架构下的 ABI，一般设置为 FFI_DEFAULT_ABI:在移动设备上 CPU 架构是 ARM64 时，FFI_DEFAULT_ABI 就是 FFI_SYSV;使用苹果公司笔记本CPU 架构是 X86_DARWIN 时， FFI_DEFAULT_ABI 就是 FFI_UNIX64。

nargs 表示输入参数的个数。arg_types 表示参数的类型，比如 ffi_type_uint32。rtype 表示返回类型，如果 返回类型是结构体，字段 flags 需要设置数值作为标记，以便在 ffi_prep_cif_machdep 函数中处理，如果返 回的不是结构体，flags 不做标记。

bytes 表示输入参数所占空间的大小，是 16 的倍数。

ffi_cif 是由ffi_prep_cif 函数生成的，而ffi_prep_cif 实际上调用的又是 ffi_prep_cif_core 函数。

##### ffi_call(函数调用)

ffi_call 函数的主要处理都交给了 ffi_call_SYSV 这个汇编函数。ffi_call_SYSV 的实现代码，你可以点击[这个链接](https://github.com/libffi/libffi/blob/master/src/aarch64/sysv.S)，其所在文件路径是 libffi/src/aarch64/sysv.S。

首先，我们一起看看 ffi_call_SYSV 函数的定义:

```
extern void ffi_call_SYSV (void *stack, void *frame,
                    void (*fn)(void), void *rvalue,
                    int flags, void *closure);
```

可以看到，通过 ffi_call_SYSV 函数，我们可以得到 stack、frame、fn、rvalue、flags、closure 参数。

各参数会依次保存在参数寄存器中，参数栈 stack 在 x0 寄存器中，参数地址 frame 在x1寄存器中，函数指 针 fn 在x2寄存器中，用于存放返回值的 rvalue 在 x3 里，结构体标识 flags 在x4寄存器中，闭包 closure 在 x5 寄存器中。

然后，我们再看看ffi_call_SYSV 处理的核心代码:

```
	/* Use a stack frame allocated by our caller.  */
	cfi_def_cfa(x1, 32);
	stp	x29, x30, [x1]
	mov	x29, x1
	mov	sp, x0
	cfi_def_cfa_register(x29)
	cfi_rel_offset (x29, 0)
	cfi_rel_offset (x30, 8)

	mov	x9, x2			/* save fn */
	mov	x8, x3			/* install structure return */
#ifdef FFI_GO_CLOSURES
	mov	x18, x5			/* install static chain */
#endif
	stp	x3, x4, [x29, #16]	/* save rvalue and flags */

	/* Load the vector argument passing registers, if necessary.  */
	tbz	w4, #AARCH64_FLAG_ARG_V_BIT, 1f
	ldp     q0, q1, [sp, #0]
	ldp     q2, q3, [sp, #32]
	ldp     q4, q5, [sp, #64]
	ldp     q6, q7, [sp, #96]
1:
	/* Load the core argument passing registers, including
	   the structure return pointer.  */
	ldp     x0, x1, [sp, #16*N_V_ARG_REG + 0]
	ldp     x2, x3, [sp, #16*N_V_ARG_REG + 16]
	ldp     x4, x5, [sp, #16*N_V_ARG_REG + 32]
	ldp     x6, x7, [sp, #16*N_V_ARG_REG + 48]

	/* Deallocate the context, leaving the stacked arguments.  */
	add	sp, sp, #CALL_CONTEXT_SIZE

	BLR(x9)				/* call fn */

	ldp	x3, x4, [x29, #16]	/* reload rvalue and flags */

	/* Partially deconstruct the stack frame.  */
	mov     sp, x29
	cfi_def_cfa_register (sp)
	ldp     x29, x30, [x29]

	/* Save the return value as directed.  */
	adr	x5, 0f
	and	w4, w4, #AARCH64_RET_MASK
	add	x5, x5, x4, lsl #3
	br	x5
```

如上面代码所示，ffi_call_SYSV 处理过程分为下面几步:

- 第一步，ffi_call_SYSV 会先分配 stack 和 frame，保存记录 fn、rvalue、closure、flags。
- 第二步，将向量参数传到寄存器，按照参数放置规则，调整 sp 的位置，
- 第三步，将参数放入寄存器，存放完毕，就开始释放上下文，留下栈里的参数。
- 第四步，通过 blr 指令调用 x9 中的函数指针 fn ，以调用函数。
- 第五步，调用完函数指针，就重新读取 rvalue 和 flags，析构部分栈指针。
- 第六步，保存返回值。

#### 如何使用libffi?

孙源在 GitHub 上有个 [Demo](https://github.com/sunnyxx/libffi-iOS)，已经集成了 iOS 可以用的 libffi 库，你可以将这个库集成到自己的工程中。


--- 


> 这一篇课程理解起来很麻烦，还需要反复阅读。。。
> 
> 这里先做下记录，待回头工作需要的时候再进行深入研究吧。。。
