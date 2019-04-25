# AGTH User Manual

## Loader options

> Syntax:  `agth.exe  <options>  exe_file_to_load  command_line`

- `/L[locale_id]` - fix application locale to 'locale_id' by AppLocale (default parameter: 411)

- `/R[locale_id]` - fix application locale to 'locale_id' (default parameter: 411)

- `/P[{process_id|Nprocess_name}]` - attach to process, invalidates /L, /R keys, 'exe_file_to_load' and 'command_line'

## Interface options

- `/B[split_mul][:[min_time][:unconditional_split_time]]` - set paragraph split parameters (default parameters: 4:24:1000)

- `/C[time]` - copy captured text to clipboard after pause of 'time' milliseconds (default parameter: 150)

- `/Fnew_name@[context][:subcontext][;new_name2@...]` - rename specified and hide all other contexts (default parameters: 0:any)

- `/KF[len_base][:len_mul]` - suppress repetition of phrases, 'len_base' and 'len_mul' are tracing parameters (default parameters: 32:16)

- `/KS[number]` - remove 'number' repetitions of each symbol (default parameter: 1)

- `/NA` - less strict access control for text transfer

- `/NF` - disable filtering of some characters

- `/NX` - toggle auto-exit on close of all hooked processes

- `/T` - always on top

- `/W[context][:subcontext]` - autoselect context (default parameters: 0:any)

## Hook options

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
