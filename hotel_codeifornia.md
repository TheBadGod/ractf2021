# Hotel Codefornia

At first it looked really like a crypto challenge and
my colleague extracted the public key information and
made this very beautiful diagram:

![https://cdn.discordapp.com/attachments/875973866856534056/875997201694883880/unknown.png]()

and with that we started investigating what could go wrong
during the process of the encryption with gmp and anything else
before eventually after an hour or so returning to the basics
and realizing that the strncmp doesn't compare all 0x20 bytes,
but only up to that many, which meant that if the hash
contained a null byte at the start (which for a good hash algo
one every 256 hashes would contain). So we first tried to generate
a hash with a null in it's first place, that worked, we got a payload
which would be `';sh #...` and well... the program segfaulted because
gmp didn't like the value being zero when it should export it. Next we
tried to get a value of 1, which required two bytes to be correct in the
hash, but still easily doable. So now we just had to make sure the 
value exported from gmp is also 1... Which somehow wasn't when we entered
a 1 (It seemed to allocate something on the heap or something?) So now
I got the idea to use another value to get a 1, which required again
two bytes to be correct (And because of the endianess the upper two bytes),
which after a quick and dirty sage script gave us the value of `0xba` and
a payload of `';sh #'~wKP`.

Then we just exploited that on the server.
`ractf{W3lComE_t0_Th3_LA_B3d_n_br3Akfast}`
