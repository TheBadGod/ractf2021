# Eldritch Horror

First there are a lot of constant values:

```python
x=[
    0x9097e261415c7b7c,
    0xe086ce41c436ab32,
    0x855021613bc076fb,
    0x1733fb818c876108,
    0xbb23ba01baaf55d1,
    0x5ade3c21c2942d6a,
    0x3f1e1aa15a9a7d16,
    0xeb4a44c18787d133,
    0xaf4f76e1e225067c,
    0x456ff4e1c6317e77,
    0xa8a0af0120205322,
    0x276071214832742e,
    0x1ae23b4108400db7,
    0xeaeab9c1e55938b7,
    0xb0452be17d8a75bf,
    0x4811a9e1fa4a0c2e,
    0x2ce627e1f2d2cf85,
    0x2ec2a5e14f19749c,
    0x1da723e1cdf9c1cb,
    0xc993a1e116678ee9,
    0x97dabc1da229d66,
    0x7f529e21813abe42,
    0xcbf7d8812f614346,
    0xdeee5ae19353b421,
    0xee0a59213a91634c,
    0x54dbd38161eca046,
    0x10d6a1c17f4f271d,
    0x6db4142169f4053c,
]
```

Just after that is a loop which decodes those constants
to addresses. I recoded this in python to quickly get the
function addresses:

```python
adrs = []
for i in x:
    param_1 = (i << 0x20 | i >> 0x20) & 0xFFFFFFFFFFFFFFFF
    param_1 = i
    uVar1 = param_1 << 0x20 | param_1 >> 0x20
    uVar3 = (uVar1 * 3 ^ 2) & 0xFFFFFFFFFFFFFFFF
    lVar2 = ((2 - uVar3 * uVar1) * uVar3) & 0xFFFFFFFFFFFFFFFF
    lVar2 = ((2 - lVar2 * uVar1) * lVar2) & 0xFFFFFFFFFFFFFFFF
    lVar2 = ((2 - lVar2 * uVar1) * lVar2) & 0xFFFFFFFFFFFFFFFF
    lVar2 = ((2 - lVar2 * uVar1) * lVar2) & 0xFFFFFFFFFFFFFFFF
    #print(hex(((2 - uVar1 * lVar2) * lVar2 >> 1) & 0x7FFFFFFFFFFFFFFF))
    adrs.append(((2 - uVar1 * lVar2) * lVar2 >> 1) & 0x7FFFFFFFFFFFFFFF)
print([hex(a) for a in adrs])
```


Then I realized that all these addresses were already in the function itself
and were "obfuscated" right during the loading and ghidra just made them
"easier" to read, I just should've looked at the outgoing calls, but oh
well, like this I at least had them in order like the bytecode. Yes, bytecode
which is located at address 0x140016000 (We always load from there after the loop
and the first element is skipped, but the first instruction is set to that element,
kinda weird). Anyway, after extracting the code:

```python
code = [0x0d,0x01,0x0a,0x0d,0x02,0x0c,0x0d,0x03,0x00,0x0d,0x04,0x02,0x14,0x00,0x10,0x03,0x00,0x03,0x03,0x0b,0x00,0x01,0x1a,0x00,0x19,0x04,0x16,0x02,0x0d,0x06,0x10,0x0d,0x05,0x01,0x01,0x02,0x03,0x0b,0x02,0x06,0x1a,0x02,0x19,0x05,0x1b,0x0d,0x02,0x38,0x0d,0x03,0x00,0x0d,0x05,0x01,0x05,0x06,0x0f,0x00,0x03,0x0e,0x07,0x03,0x12,0x07,0x0b,0x07,0x00,0x1a,0x07,0x19,0x05,0x1b,0x03,0x03,0x01,0x00,0x03,0x0b,0x00,0x06,0x1a,0x00,0x19,0x04,0x16,0x02,0x0d,0x02,0x5c,0x0d,0x03,0x80,0x0e,0x00,0x03,0x15,0,3,3,0xb,0,1,0x1a,0,0x19,4,0x16,2,0x1b]
```

I started work on a basic disassembler, just looking at the function which would get
executed with that instruction and the four parameters of the indirect call in
the main function.

```python
i = 0
while i < len(code):
    instr = code[i]
    print(f"{i:2x}: ",end="")
    i+=1

    if instr == 0:
        print("nop")
    elif instr == 0x01:
        dst = code[i]
        src = code[i+1]
        i += 2
        print(f"mov r{reg}, r{src}")
    elif instr == 3:
        reg = code[i]
        i += 1
        print(f"inc r{reg}")
    elif instr == 5:
        reg = code[i]
        i += 1
        print(f"dec r{reg}")
    elif instr == 0x0d:
        reg = code[i]
        value = code[i+1]
        i += 2
        print(f"mov r{reg}, {hex(value)}")
    elif instr == 0x0b:
        reg = code[i]
        op = code[i+1]
        i += 2
        print(f"xor r{reg}, r{op}")
    elif instr == 0x0e:
        dst = code[i]
        src = code[i+1]
        i += 2
        print(f"mov r{dst}, &[r{src}]")
    elif instr == 0x0f:
        dst = code[i]
        src = code[i+1]
        i += 2
        print(f"mov r{dst}, [r{src}]")
    elif instr == 0x10:
        dst = code[i]
        src = code[i+1]
        i += 2
        print(f"mov [r{dst}], r{src}")
    elif instr == 0x12:
        dst = code[i]
        i += 1
        print(f"dostuff r{dst}") # 0x140003010, some sort of weird shifting and subtracting
    elif instr == 0x15:
        out = code[i%0x6d]
        i += 1
        print(f"outs {chr(out)}")
    elif instr == 0x14:
        reg = code[i]
        i += 1
        print(f"getc r{reg}")
    elif instr == 0x16:
        dst = code[i]
        i += 1
        print(f"jmp r{dst} # absolute jump")
    elif instr == 0x19:
        reg = code[i]
        i += 1
        print(f"je rip+r{reg} # relative jump")
    elif instr == 0x1a:
        reg = code[i]
        i += 1
        print(f"cmp r{reg}, 0")
    elif instr == 0x1b:
        print("exit")
    else:
        print(f"Unknown opcode: {hex(instr)} ==> {hex(adrs[instr])}")
        exit(1)
```

which gave me this code:

```
 0: mov r1, 0xa
 3: mov r2, 0xc
 6: mov r3, 0x0
 9: mov r4, 0x2
 c: getc r0
 e: mov [r3], r0
11: inc r3
13: xor r0, r1
16: cmp r0, 0
18: je rip+r4 # relative jump
1a: jmp r2 # absolute jump
1c: mov r6, 0x10
1f: mov r5, 0x1
22: mov r5, r3
25: xor r2, r6
28: cmp r2, 0
2a: je rip+r5 # relative jump
2c: exit
2d: mov r2, 0x38
30: mov r3, 0x0
33: mov r5, 0x1
36: dec r6
38: mov r0, [r3]
3b: mov r7, &[r3]
3e: dostuff r7
40: xor r7, r0
43: cmp r7, 0
45: je rip+r5 # relative jump
47: exit
48: inc r3
4a: mov r3, r3
4d: xor r0, r6
50: cmp r0, 0
52: je rip+r4 # relative jump
54: jmp r2 # absolute jump
56: mov r2, 0x5c
59: mov r3, 0x80
5c: mov r0, &[r3]
5f: outs 
61: inc r3
63: xor r0, r1
66: cmp r0, 0
68: je rip+r4 # relative jump
6a: jmp r2 # absolute jump
6c: exit
```

In retrospect the exit instruction is probably a "Halt and Catch Fire" or just exit function.

Then I started working on reversing the very short program. Until the first absolute jump
we just read the input and stop when we hit a newline (BufferOverflow? Just need a way
to leak the canary kappa). The next loop makes sure your input is 16 chars with the null
terminator, then xors the char with another byte generated based on a fixed key and that
needs to result in a zero. So let's extract that key: (Set in the main function
as a stack variable)

```python
stuff = [
        0xde,0xbe,0x7f,0x95,0x50,0x7c,0xbc,0xb2,
        0xb0,0xf1,0xde,0x99,0xf5,0xbc,0x33,
        0x43,0x6f,0x72,0x72,0x65,0x63,0x74,0x21, 0x10]
```

Only the first 16 elements are needed, but I just went on going :)
So now we just go through and generate the char and because it's xor we
will just get the flag this way:

```python
def a(x):
    x1 = (x<<4|x>>4)&0xFF
    x2 = (x1*3^2)&0xFF
    x3 = ((2-x2*x1) * x2) & 0xFF
    x3 = ((2-x3*x1) * x3) & 0xFF
    return (((2-x3*x1)*x3)>>1) & 0x7F
print(bytes([a(x) for i,x in enumerate(stuff)]))
```

which gives us: `ractf{qAQorTOq}`

And the last part of the program just prints "Correct!" if your input was correct,
thus easily locally verifyable if the flag was correct or not
