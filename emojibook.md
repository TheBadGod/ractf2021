# Emojibook

The Service does `.replace("{{","").replace("}}","")` and so on
to filter bad stuff. Which generally is a bad idea to do this sequentially,
as we can just enter "{}}{" and after doing the above we would've
gotten "{{" because we only filter once. So by doing this
we can exploit the emoji include because os.path.join takes the second
path as an absolute path if it starts with a slash.
So the solution was simply to enter `({}}{/flag.txt}..})` to
get the flag which was readable by the current user: `ractf{dj4ng0_lfi}`
