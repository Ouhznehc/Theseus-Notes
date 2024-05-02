# Initial Assembly Code

Theseus starts from the `_start` function in `kernel/nano_core/src/asm/bios/boot.asm`, which is an assembly file. Next, we will progressively dissect the contents of this file.

```asm
%include "defines.asm"

global _start

; Section must have the permissions of .text
section .init.text32 progbits alloc exec nowrite
bits 32 ;We are still in protected mode

extern set_up_SSE

%ifdef ENABLE_AVX
extern set_up_AVX
%endif ; ENABLE_AVX
```

`boot.asm` starts by including necessary header files, which typically contain global definitions such as constants and macro definitions used across multiple files. Then, it defines a section named `.init.text32` designed for initialization under 32-bit protected mode, preparing the operating system for transitioning to a higher functionality state.

```asm
_start:
	; The bootloader has loaded us into 32-bit protected mode. 
	; Interrupts are disabled. Paging is disabled.

	; To set up a stack, we set the esp register to point to the top of our
	; stack (as it grows downwards on x86 systems). This is necessarily done
	; in assembly as languages such as Rust cannot function without a stack.
	;
	; We subtract KERNEL_OFFSET from the stack address because we are not yet
	; mapped to the higher half
	mov esp, initial_bsp_stack_top - KERNEL_OFFSET

	; The multiboot2 specification requires the bootloader to load a pointer
	; to the multiboot2 information structure in the `ebx` register. Here we
	; mov it to `edi` so that rust can take it as a register. Because of this
	; we cannot clobber the edi register in any code before nano_core_start
	mov edi, ebx

	call check_multiboot
	call check_cpuid
	call check_long_mode

	call set_up_SSE
%ifdef ENABLE_AVX
	call set_up_AVX
%endif ; ENABLE_AVX

	call set_up_page_tables
	call unmap_guard_page
	call enable_paging

	; Load the 64-bit GDT
	lgdt [GDT.ptr_low - KERNEL_OFFSET]

	; Load the code selector with a far jmp
	; From now on instructions are 64 bits and this file is invalid
	jmp GDT.code:long_mode_start; -> !
```

Theseus starts from `_start`. We will go through the code line by line to understand its functionality. The first line `mov esp, initial_bsp_stack_top - KERNEL_OFFSET` sets the initial top of the stack. The `initial_bsp_stack_top` is defined in the section shown below. As for why `KERNEL_OFFSET` is subtracted, although there is a comment provided in the code, the author has limited understanding of this aspect and cannot provide an explanation, so this part will be omitted for now.

## Stack Layout

```asm
; Note that the linker script (`linker_higher_half.lf`) inserts a 2MiB space here 
; in order to provide stack guard pages beneath the .stack section afterwards.
; We don't really *need* to specify the section itself here, but it helps for clarity's sake.
section .guard_huge_page nobits noalloc noexec nowrite

; Although x86 only requires 16-byte alignment for its stacks, 
; we use page alignment (4096B) for convenience and compatibility 
; with Theseus's stack abstractions in Rust. 
; We place the stack in its own sections for loading/parsing convenience.
; Currently, the stack is 16 pages in size, with a guard page beneath the bottom.
; ---
; Note that the `initial_bsp_stack_guard_page` is actually mapped by the boot-time page tables,
; but that's okay because we have real guard pages above. 
section .stack nobits alloc noexec write  ; same section flags as .bss
align 4096 
global initial_bsp_stack_guard_page
initial_bsp_stack_guard_page:
	resb 4096
global initial_bsp_stack_bottom
initial_bsp_stack_bottom:
	resb 4096 * INITIAL_STACK_SIZE
global initial_bsp_stack_top
initial_bsp_stack_top:
	resb 4096
initial_double_fault_stack_top:
```

The above code defines the layout of the stack, including two sections: `.guard_huge_page` and `.stack`. In the `.stack` section, memories are allocated and several constants are defined to point to specific memory addresses. The `.guard_huge_page` section is used to provide stack guard pages, which are essential for protecting the stack from overflow and other vulnerabilities.

<figure><img src="stack.png" alt=""><figcaption></figcaption></figure>
