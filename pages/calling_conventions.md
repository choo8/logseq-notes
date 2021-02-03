---
title: Calling Conventions
---

## Definition
### An implementation-level scheme for how subroutines receive parameters from their caller and how they return a result
### Differences in various implementations include where parameters, return values, return addresses and scope links are placed (registers, stack or memory etc.), and how the tasks of preparing for a function call and restoring the environment afterwards are divided between the caller and the callee
## x86 instruction set architecture
### cdecl (C declaration), also known as
#### 32-bit
##### e.g. a simple C example
```C
int callee(int, int, int);

int caller(void)
{
	return callee(1, 2, 3) + 5;
}
```
##### Save old call frame + create a new call frame
```x86
push ebp
mov ebp, esp
```
##### Allocate space for local variable if any
```x86
sub esp, 12
```
##### Push call arguments onto stack, from right to left (rightmost argument will have largest address value)
```x86
push 3
push 2
push 1
```
##### Call the callee function
```x86
call callee
```
##### Place return value in `eax` register
```x86
add eax, 5
```
##### Restore old call frame
```x86
mov esp, ebp
pop ebp
```
##### Return from function
```x86
ret
```
#### 64-bit
##### Calling convention is very similar to that of cdecl 32-bit, with some differences
##### Call arguments are now passed in registers first. First 6 integer or pointer arguments will be passed in `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`. First floating point arguments will be passed in `xmm0`, `xmm1`, `xmm2`, `xmm3`, `xmm4`, `xmm5`, `xmm6` and `xmm7`. Additional arguments will be passed on the stack, similar to the case of 32-bit.


##### Return value is placed in `rax` instead of `eax` and if the return value is 128 bit, it will be placed in `rdx:rax`. Floating-point return values are similarly stored in `xmm0` and `xmm1`.
##### If the callee wishes to clobber `rbx`, `rsp`, `rbp` or `r12–r15`, it must save them and restore them before return to the caller. All other registers are caller-saved if their values are to be preserved by the caller.
#### Intuition
##### Caller will clean the stack after the callees all return
```x86
mov esp, ebp
pop ebp
```
When the above is done, all local variables and previously used call arguments will be considered "undefined" as they will be in memory locations above the current `esp`. This is so as the cdecl calling convention relies on the caller to perform stack clean-up
##### If the caller requires its registers to not be clobbered, it has to save registers `eax`, `edx` and `ecx`
```x86
push eax
push edx
push ecx
```
However, this may not be done often as values can usually be saved in local variables on the stack before calling any callee functions.
##### If the callee wishes to clobber any registers other than `eax`, `edx` and `ecx`, it will have to push them onto the stack before clobbering them and pop them back afterwards. However, this is not always done as the 3 caller-saved registers may be sufficient for the callee function to perform its calculations.
### stdcall
#### 32-bit
##### Calling convention is similar to that of cdecl 32-bit, with some differences
##### stdcall is a callee clean-up calling convention, hence the callee will clean up the call argument from the stack when it returns control flow back to the caller
```x86
ret 12
```
When performing the instruction `ret`, it also pops off a specific number of bytes on the stack, corresponding to the number of call arguments of the callee function, which the caller pushed onto the stack before calling the callee function.
#### 64-bit
##### Calling convention is similar to that of stdcall 32-bit, with some differences
##### Call arguments are now passed in registers first. First 6 integer or pointer arguments will be passed in `rcx`, `rdx`, `r8`, `r9`. First floating point arguments will be passed in `xmm0`, `xmm1`, `xmm2`, `xmm3`. Additional arguments will be passed on the stack, similar to the case of 32-bit.
##### If the callee wishes to clobber `rbx`, `rbp`, `rdi`, `rsi`, `rsp` or `r12-15`, it must save them and restore them before return to the caller. All other registers are caller-saved if their values are to be preserved by the caller.
## References
### https://en.wikipedia.org/wiki/Calling_convention
### https://en.wikipedia.org/wiki/X86_calling_conventions
