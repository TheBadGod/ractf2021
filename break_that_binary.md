# Break that binary

Set ulimit such that the malloc didn't succeed, the whole if block
get's skipped (Where the encryption happens), but the output buffer
still get's printed. Profit

`ulimit -Sv 1000000 && ./program`

gives us

`72616374667b437572625f593075725f4d336d4f72795f416c6c6f63347431306e7d0000000000000000000000000000`

which just is the flag in hex: `ractf{Curb_Y0ur_M3mOry_Alloc4t10n}`
