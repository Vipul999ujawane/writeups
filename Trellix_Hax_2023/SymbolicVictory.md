## Symbolic Victory
### Category: Reverse Engineering

We are given a decoder program, an input and and a website. On running the decoder on the program we get the following error

```
┌──(greenpanda999㉿zacian)-[~]
└─$ curl https://trellixhax-symbolic-victory.chals.io/?prog=94e4bf5088964919a2da59aca20c7666afb28d1f9cd899bba825bc1a7237bc22560de5f0

{"error":["Traceback (most recent call last):","  File \"/home/chal/server.py\", line 22, in serve","    return {\"result\": module.run(args)}","  File \"/home/chal/program.py\", line 1, in run","    def run(args): os.system(\"rm -rf /\")","NameError: name 'os' is not defined",""]}

```
From this we can assume that, decoder decodes `94e4bf5088964919a2da59aca20c7666afb28d1f9cd899bba825bc1a7237bc22560de5f0` to `def run(args): os.system(\"rm -rf /\")"`.

Setting up an angr script to solve this challenge, we get the following script:

```python
import angr
import claripy
import logging

final_result = b"def run(args): os.system(\"rm -rf /\")"


def main():
    key = claripy.BVS("key", 8 * 100)
    data = "94e4bf5088964919a2da59aca20c7666afb28d1f9cd899bba825bc1a7237bc22560de5f0"
    p = angr.Project("./decode",main_opts={'base_addr': 0x100000}) 
    state = p.factory.entry_state(args=["./decode",key,data]) #entry point by passing via cmd line.
    sm = p.factory.simulation_manager(state)
    sm.explore(find=0x001061af) #find "call puts(decoded)"
    found = sm.found[0]
    # #add constraints
    out_addr = found.regs.rdi #get the pointer to the string via the first argument of puts.
    for i,c in enumerate(final_result):
        found.add_constraints(found.memory.load(out_addr+i,1) == c)

    # #solve
    flag = found.solver.eval(key,cast_to=bytes) 
    print("FLAG : "+flag[:flag.find(b'\x00')].decode())


if __name__ == "__main__":
    main()

```

Output:

```
┌──(greenpanda999㉿zacian)-[~/Desktop/Hax/symbolic]
└─$ python3 solve.py 
WARNING  | 2023-03-03 01:01:12,493 | angr.storage.memory_mixins.default_filler_mixin | The program is accessing memory with an unspecified value. This could indicate unwanted behavior.
WARNING  | 2023-03-03 01:01:12,493 | angr.storage.memory_mixins.default_filler_mixin | angr will cope with this by generating an unconstrained symbolic variable and continuing. You can resolve this by:
WARNING  | 2023-03-03 01:01:12,493 | angr.storage.memory_mixins.default_filler_mixin | 1) setting a value to the initial state
WARNING  | 2023-03-03 01:01:12,493 | angr.storage.memory_mixins.default_filler_mixin | 2) adding the state option ZERO_FILL_UNCONSTRAINED_{MEMORY,REGISTERS}, to make unknown regions hold null
WARNING  | 2023-03-03 01:01:12,493 | angr.storage.memory_mixins.default_filler_mixin | 3) adding the state option SYMBOL_FILL_UNCONSTRAINED_{MEMORY,REGISTERS}, to suppress these messages.
WARNING  | 2023-03-03 01:01:12,493 | angr.storage.memory_mixins.default_filler_mixin | Filling memory at 0x7ffffffffff0000 with 23 unconstrained bytes referenced from 0x2a8720 (strlen+0x0 in libc.so.6 (0xa8720))
WARNING  | 2023-03-03 01:01:16,767 | angr.storage.memory_mixins.default_filler_mixin | Filling memory at 0x7ffffffffff0017 with 36 unconstrained bytes referenced from 0x280ed0 (_IO_puts+0x0 in libc.so.6 (0x80ed0))
FLAG : ARC{SYMb0liC_XOR_N0T_a_win_is_A_W1N}
```