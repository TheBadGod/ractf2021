# I'm a fan

We quickly scripted the file upload to not have
to select files when uploading. There we noticed
the "external" in the request and tried setting it
to internal expecting a server side request forgery, but 
the server just responded with 500 when we tried that.

After a night of sleep there was suddenly news that
we can upload files with "internal" but there we need
to specify a path and not a link or upload a file, which
was unexpected, but with this we could read files.

We then spent way too long reading random files to
get information but we didn't try to do the obivous thing:
Read the source of the app. But when we finally did we
were greeted by a comment saying that there is
a note which can't be accessed by simply navigating there...
So we just "uploaded" a file with that path and were able to
read the note:

```
Dev Note to Admin
------------------

I've fixed the security issues now! No one will be able to steal your creds again!
I've even made sure they can't load this file from the WSGI server.

name: admin
$argon2id$v=19$m=102400,t=2,p=8$WmI1xK/4Z1fiLj20O2fDpA$7Oy7WAp9gERzT/T5RlDXCw
$argon2id$v=19$m=102400,t=2,p=8$vZaZ9D3ERgo/DOnoNvQkEA$IOxfMIF7cgKbnHalTLU9uA

Hahaha yes ultra secure solution go brrrrrrrr!
```

The I and the l were not very distinguishable in the font they used, but after
using [https://github.com/CyberKnight00/Argon2_Cracker]() with a wordlist we found
those hashes above as correct ones (Because they gave us "password" and "qwertyuiop"
respectively).

So now we had the full password of admin and could ssh into the machine, now we just
had to get the flag. After running find we didn't get any results, so it must
still be inaccessible for admin. But there is a random backup of the shadow file
which we can read. So get that, crack that to get password `ubisoft` for root
and read the flag in the /root directory.

`ractf{l4ws_0f_phys1cs_c4n_tak3_a_h1ke}`
