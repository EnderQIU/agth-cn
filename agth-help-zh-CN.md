# AGTH 用户手册

使用方法：

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

## 钩子选项

> `/H[X]{A|B|W|S|Q}[N][data_offset[*drdo]][:sub_offset[*drso]]@addr[:module[:{name|#ordinal}]]`

- `/NC` - don't hook child processes

- `/NH` - no default hooks

- `/NJ` - use thread code page instead of Shift-JIS for non-unicode text (should be specified for capturing non-japanese text)

- `/NS` - don't use subcontexts

- `/S[IP_address]` - send text to custom computer (default parameter: local computer)


- `/V` - process text threads from system contexts

- `/X[sets_mask]` - extended sets of hooked functions (default parameter: 1; number of available sets: 2)


> All numbers in `/L`, `/R`, `/F`, `/W`, `/X`, `/H` (except ordinal) are hexadecimal without any prefixes

## Set additional custom hook

`/H[X]{A|B|W|S|Q}[N][data_offset[*drdo]][:sub_offset[*drso]]@addr[:[module[:{name|#ordinal}]]]`

### Hook types:

- `A` - DBCS char

- `B` - DBCS char (big-endian)

- `W` - UCS2 char

- `S` - MBCS string

- `Q` - UTF-16 string

### Parameters:

- `X` - use hardware breakpoints

- `N` - don't use contexts

- `data_offset` - stack offset to char / string pointer

- `drdo` - add a level of indirection to data_offset

- `sub_offset` - stack offset to subcontext

- `drso` - add a level of indirection to sub_offset

- `addr` - address of the hook

- `module` - name of the module to use as base for 'addr'

- `name` - name of the 'module' export to use as base for 'addr'

- `ordinal` - number of the 'module' export ordinal to use as base for 'addr'

### Negative values of 'data_offset' and 'sub_offset' refer to registers:

- `-4` for EAX

- `-8` for ECX

- `-C` for EDX

- `-10` for EBX

- `-14` for ESP

- `-18` for EBP

- `-1C` for ESI

- `-20` for EDI


> "Add a level of indirection" means in C/C++ style: (*(ESP+data_offset)+drdo) insted of (ESP+data_offset)

> All numbers except ordinal are hexadecimal without any prefixes

## Set additional hook with user supplied handler

`/H[X]C<code>@addr[:[module[:{name|#ordinal}]]]`

- `<code>` - raw x86 position-independent code, last byte is checksum, encoded in BASE64

### Start of code is called as hook handler function with following environment:

- `eax`, `ecx`, `edx`, `ebp` - original values

- `ebx` - address of handler start

- `esi` - original esp

- `edi` - pointer to AGTH service functions

- `[esp+04h]` - address of hooked location

- `[esp+08h]` - original ebx

- `[esp+0Ch]` - original esi

- `[esp+10h]` - original edi


> User function shall return with ret and not change global memory, including stack above its return address.

> Preserving general purpose registers besides esp is not required.

> Note: all handler memory is RWE, so can be used to store global variables, but be aware of threading issues.

## AGTH service functions pointers, stdcall calling convention:

- `[edi+00h]` - `void SendText(char *Name, DWORD Context, DWORD SubContext, char/wchar_t *Text, DWORD TextLenLimit, DWORD UnicodeFlag)` - sends text; Name is in UTF-8 and can be NULL; Text is in MBCS when UnicodeFlag is 0, and in UTF-16 when 1

- `[edi+04h]` - `BOOL SetHook(void *Handler, void *Location, DWORD HardwareFlag)` - installs hook at given Location; current limit: 4 hooks; HardwareFlag 1 forces hardware hook; returns 0 on fail

- `[edi+08h]` - `BOOL RemoveHook(void *Handler, void *Location)`- removes hooks; Handler and/or Location can be 0 for any; returns 0 on fail

- `[edi+0Ch]` - `BOOL IsBadReadPtr(void *Ptr, DWORD Len)`- returns 0 if all Len bytes starting from location Ptr can be read

- `[edi+10h]` - `void *memmem(void *Haystack, DWORD HaystackLen, void *Needle, DWORD NeedleLen)` - searches Needle in Haystack; returns address or 0 if not found

- `[edi+14h]` - `DWORD:HMODULE GetModuleHandle(wchar_t *ModuleName)` - returns address of ModuleName (for exe NULL can be used) in eax and its size in edx; 0:0 if not found

- `[edi+18h]` - `void *GetProcAddress(HMODULE Module, char *ProcName)` - returns address of function ProcName exported by Module; 0 if not found
