# AGTH 用户手册

本汉化版原文提取自 AGTH 的帮助选项。[原文](./agth-help.md)中一些不标准的计算机术语将根据自己的理解转化为标准计算机术语进行翻译。

- 关于hook一词，用作动词时译为“提取”，用作名词时译为“钩子”（钩子函数）。
- 关于context一词，在被提取的文本中和Windows指令中含义不同。

## 加载选项

> 语法：`agth.exe  <options>  exe_file_to_load  command_line`

译者注：'locale_id' 可以通过[此网页](https://ss64.com/locale.html)取得。

- `/L[locale_id]` - 使用 AppLocale 将应用程序区域设置为 'locale_id'。（默认值：411）

- `/R[locale_id]` - 将应用程序区域设置为 'locale_id'。（默认值：411）

- `/P[{process_id|Nprocess_name}]` - 附加到进程，此选项会使`/L`、`/R`选项和`exe_file_to_load`、`command_line`参数无效。

## 接口选项

译者注：默认参数值中，半角冒号用于按顺序分隔多个参数值。

- `/B[split_mul][:[min_time][:unconditional_split_time]]` - 设置段落分隔符（默认值：4:24:1000）

- `/C[time]` - 在'time'毫秒后，将捕获的文本拷贝至剪贴板。 （默认值：150）

- `/Fnew_name@[context][:subcontext][;new_name2@...]` - 重命名'context'为'new_name'，并去掉所有的'subcontext'。（默认值：0:any)？？

- `/KF[len_base][:len_mul]` - 以单个短语最长'len_base'个字符、最多重复'len_mul'次为标准去除重复的短语。（例：no no no no no => no）（默认值： 32:16)

- `/KS[number]` - 去除重复'number'次的字符。(例：Nooooo => No)（默认参数值：1）

- `/NA` - 较不严格的文本传输访问控制。？？

- `/NF` - 禁止过滤某些字符。？？

- `/NX` - 当所有附加的进程结束时，退出AGTH程序。

- `/T` - 使AGTH程序窗口总在最前方。

- `/W[context][:subcontext]` - 自动选择上下文。（默认值：0:any)？？

## 提取选项

> `/H[X]{A|B|W|S|Q}[N][data_offset[*drdo]][:sub_offset[*drso]]@addr[:module[:{name|#ordinal}]]`

- `/NC` - 不提取子线程的文本。

- `/NH` - 不使用默认的提取方法。（例如GetWindowTextA）

- `/NJ` - 对非 Unicode 编码的文本，使用线程所使用的代码页代替 Shift-JIS。 (对非日文文本使用)

- `/NS` - 不使用子线程的上下文（[CONTEXT](https://docs.microsoft.com/zh-cn/windows/desktop/api/winnt/ns-winnt-_arm64_nt_context)）。

- `/S[IP_address]` - 将文本发送到自定义的计算机。（默认值：local computer）

- `/V` - 使用系统级上下文（system context）处理文本线程。

- `/X[sets_mask]` - 使用扩展的提取方法。（默认值：1；最大值：2）

> 选项 `/L`, `/R`, `/F`, `/W`, `/X`, `/H` 中的所有数字（除了序数#ordinal之外）都是无前缀的十六进制表示。

## 设置额外的自定义提取方法

> `/H[X]{A|B|W|S|Q}[N][data_offset[*drdo]][:sub_offset[*drso]]@addr[:[module[:{name|#ordinal}]]]`

### 提取类型

- `A` - 双字节字符集字符（[DBCS](https://en.wikipedia.org/wiki/DBCS)）

- `B` - 双字节字符集字符（大端模式）

- `W` - 通用字符集字符（[UCS](https://en.wikipedia.org/wiki/Universal_Coded_Character_Set)）

- `S` - 变长字节编码字符串（[MBCS](https://en.wikipedia.org/wiki/Variable-width_encoding#MBCS)）

- `Q` - UTF-16 字符串

### 参数

- `X` - 使用硬件断点。

- `N` - 不使用上下文。

- `data_offset` - 子符或字符串指针的栈偏移量。

- `drdo` - `date_offset`的引用层次。

- `sub_offset` - 子上下文的栈偏移量。

- `drso` - 对`sub_offset`增加引用层次。

- `addr` - 钩子的地址。

- `module` - 钩子的地址所属的模块名称。

- `name` - 'module'导出时所使用的名称。

- `ordinal` - 'name'的引用次数。

> 'data_offset'和'sub_offset'的值若为负值，表示引用寄存器中的值：

- `-4` 使用 EAX 的值。

- `-8` 使用 ECX 的值。

- `-C` 使用 EDX 的值。

- `-10` 使用 EBX 的值。

- `-14` 使用 ESP 的值。

- `-18` 使用 EBP 的值。

- `-1C` 使用 ESI 的值。

- `-20` 使用 EDI 的值。

> "Add a level of indirection" means in C/C++ style: (*(ESP+data_offset)+drdo) insted of (ESP+data_offset)
> All numbers except ordinal are hexadecimal without any prefixes

## 使用提供的回调函数设置额外的提取方法

> `/H[X]C<code>@addr[:[module[:{name|#ordinal}]]]`

- `<code>` - 使用BASE64编码的原始的 x86 地址无关代码，最后一位是校验和。

> `<code>`以一个回调函数开始，它可以使用以下变量：

- `eax`, `ecx`, `edx`, `ebp` - 寄存器的原始值。

- `ebx` - 回调函数的起始地址。

- `esi` - 寄存器`esi`的值。

- `edi` - 指向AGTH提供的回调函数首地址的指针。

- `[esp+04h]` - 被提取的指令的地址。

- `[esp+08h]` - 寄存器`ebx`的值。

- `[esp+0Ch]` - 寄存器`esi`的值。

- `[esp+10h]` - 寄存器`edi`的值。

> 你编写的函数应该以`ret`指令结束，并且不允许修改包括调用栈在内的全局变量。
> 除了`esp`之外，你不需要保存通用寄存器的值。
> 注意：所有回调函数的内存访问权限都是RWE（读、写、执行），所以可以用于存储全局变量，但是请注意线程间的互斥与同步。

## AGTH 提供的回调函数指针，遵守[stdcall函数调用约定](https://docs.microsoft.com/en-us/cpp/cpp/stdcall?view=vs-2019)

- `[edi+00h]` 函数原型 `void SendText(char *Name, DWORD Context, DWORD SubContext, char/wchar_t *Text, DWORD TextLenLimit, DWORD UnicodeFlag)` - 发送文本； `Name`是可以为NULL的UTF-8字符串；当`UnicodeFlag`为0时，`Text` 以多字节字符集（[MBCS](https://docs.microsoft.com/en-us/cpp/text/unicode-and-mbcs?view=vs-2019)）表示，当`UnicodeFlag`为1时为用 UTF-16 编码的字符串。

- `[edi+04h]` 函数原型 `BOOL SetHook(void *Handler, void *Location, DWORD HardwareFlag)` - 对`Location`地址设置回调函数； 目前最多设置4个回调函数；如果设置`HardwareFlag`为1会强制使用硬件断点；如果设置失败，将会返回0。

- `[edi+08h]` 函数原型 `BOOL RemoveHook(void *Handler, void *Location)`- 移除回调函数；参数`Handler`或`Location`为0时将移除所有回调函数；如果移除失败，将会返回0。

- `[edi+0Ch]` 函数原型 `BOOL IsBadReadPtr(void *Ptr, DWORD Len)`- 如果从`Ptr`地址开始、长度为`len`的内存都是可读的，则返回0；

- `[edi+10h]` 函数原型 `void *memmem(void *Haystack, DWORD HaystackLen, void *Needle, DWORD NeedleLen)` - 使用[字符串搜索算法](https://en.wikipedia.org/wiki/String-searching_algorithm)从`Haystack`地址开始搜索指针`Needle`指向的字符串；返回第一个匹配的字符串的首地址，如果在指定的范围内没有匹配，则返回0。

- `[edi+14h]` - `DWORD:HMODULE GetModuleHandle(wchar_t *ModuleName)` - 返回在eax寄存器中`ModuleName`模块的地址和它的长度（位移edx寄存器中），对于exe可执行文件可以使用NULL；如果没有找到，返回0:0。

- `[edi+18h]` - `void *GetProcAddress(HMODULE Module, char *ProcName)` - 返回`Module`中`ProcName`函数的地址；如果没有找到，返回0。
