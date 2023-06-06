# Scramble

> oops, i think my siblings messed with the line order a little. The first three lines are given

We are given a python program with its lines scrambled, and we have to unscramble them to create a working program. Here is our initial file, with the first three lines given:

``` python
# Unscrambled
import random
seed = 1000
random.seed(seed)

# Scrambled
def recur(lst):
l2[i] = (l[i]*5+(l2[i]+n)*l[i])%l[i]
l2[i] += inp[i]
flag = ""
flag+=chr((l4[i]^l3[i]))
return flag
l.append(random.randint(6, 420))
l3[0] = l2[0]%mod
for i in range(1, n):
def decrypt(inp):
for i in range(n):
assert(len(l)==n)
return lst[0]
l = []
main()
def main():
l4 = [70, 123, 100, 53, 123, 58, 105, 109, 2, 108, 116, 21, 67, 69, 238, 47, 102, 110, 114, 84, 83, 68, 113, 72, 112, 54, 121, 104, 103, 41, 124]
l3[i] = (l2[i]^((l[i]&l3[i-1]+(l3[i-1]*l[i])%mod)//2))%mod
if(len(lst)==1):
assert(lst[0]>0)
for i in range(1, n):
for i in range(n):
return recur(lst[::2])/recur(lst[1::2])
print("flag is:", decrypt(inp))
l2[0] +=int(recur(l2[1:])*50)
l2 = [0]*n
flag_length = 31
mod = 256
print(l2)
n = len(inp)
inp = [1]*flag_length
l3 =[0]*n
```

## Solution

This file defines three functions: `recur`, `decrypt`, and `main`. I'll start by moving out these functions, so that I can fill them in as I unscramble the rest of the code.

I'm assuming that they are all defined at the top level of the file. I will also assume that the call to the `main` function is the only other line at the top level.

``` python
import random
seed = 1000
random.seed(seed)

def recur(lst):
	# todo

def decrypt(inp):
	# todo

def main():
	# todo

main()
```

### recur

Based on its name, I assume that `recur` is a recursive function. Therefore, I move the call to `recur` into its body.

``` python
def recur(lst):
	return recur(lst[::2])/recur(lst[1::2])
```

We can determine what other lines belong in this function by looking at which variables they access. `recur` defines the parameter `lst`, so I searched through the code for any occurrences of this identifier.

``` python
return lst[0]
if(len(lst)==1):
assert(lst[0]>0)
```

The only place it is ever defined is as the parameter of `recur`, so all uses of `lst` must be inside the `recur` function.

Since `recur` is a recursive function, it needs a base case, so we can assume that is what these three lines are for.

``` python
def recur(lst):
    if(len(lst)==1):
        assert(lst[0]>0)
        return lst[0]
	
    return recur(lst[::2])/recur(lst[1::2])
```

Since the result of the function is used as a divisor in the return statement, we assert that it is greater than zero so that we don't get any division-by-zero errors.

This completes the `recur` function.

### main

Next I'm going to assume that the line where the flag is printed belongs in the `main` function. This line calls `decrypt` and I doubt that function is recursive, and `recur` is complete, so it must go in `main`. This is the most logical place for the result of the program to be printed, anyways.

``` python
def main():
    print("flag is:", decrypt(inp))
```

This line accesses the variable `inp`, which is defined in two places: as a parameter of `decrypt`, which isn't helpful, and in `inp = [1]*flag_length`, which looks more promising.

``` python
def main():
    inp = [1]*flag_length
    print("flag is:", decrypt(inp))
```

This line uses the variable `flag_length`, so we use the same process. It is only defined in one location, so:

``` python
def main():
    flag_length = 31
    inp = [1]*flag_length
    print("flag is:", decrypt(inp))
```

### decrypt

Next, the `decrypt` function. All of the remaining lines will go here. It takes one parameter, `inp`, and seems to return the flag as a string.

Here are the remaining scrambled lines:

``` python
l2[i] = (l[i]*5+(l2[i]+n)*l[i])%l[i]
l2[i] += inp[i]
flag = ""
flag+=chr((l4[i]^l3[i]))
return flag
l.append(random.randint(6, 420))
l3[0] = l2[0]%mod
for i in range(1, n):
for i in range(n):
assert(len(l)==n)
l = []
l4 = [70, 123, 100, 53, 123, 58, 105, 109, 2, 108, 116, 21, 67, 69, 238, 47, 102, 110, 114, 84, 83, 68, 113, 72, 112, 54, 121, 104, 103, 41, 124]
l3[i] = (l2[i]^((l[i]&l3[i-1]+(l3[i-1]*l[i])%mod)//2))%mod
for i in range(1, n):
for i in range(n):
l2[0] +=int(recur(l2[1:])*50)
l2 = [0]*n
mod = 256
print(l2)
n = len(inp)
l3 =[0]*n
```

It looks like the bulk of this function is manipulating 4 lists, `l`-`l4`. Let's start with `l`. 

It is initially defined as an empty list (`l = []`), but the line `assert(len(l)==n)` suggests that it should have `n` elements. There is a line that appends an element to `l`, and a for loop that repeats `n` times, so lets combine all of that:

``` python
def decrypt(inp):
    n = len(inp)
    
    l = []
    
    for i in range(n):
        l.append(random.randint(6, 420))
    
    assert(len(l)==n)
```

Now on to `l2`. It is initially filled with `n` zeroes:

``` python
l2 = [0]*n
```

And there are three lines that seem to change its value:

``` python
l2[i] = (l[i]*5+(l2[i]+n)*l[i])%l[i]
l2[i] += inp[i]

l2[0] +=int(recur(l2[1:])*50)
```

The first two reference a variable `i`, so they must go inside a for loop. However, it's unclear whether they belong in the same for loop, and which type of for loop they belong in—we have one that iterates over `range(n)`, and one that iterates over `range(1, n)`. Therefore, we'll move on to `l3` and come back.

It is also initially filled with `n` zeroes:

``` python
l3 = [0]*n
```

And is modified on two other lines:

``` python
l3[0] = l2[0]%mod
l3[i] = (l2[i]^((l[i]&l3[i-1]+(l3[i-1]*l[i])%mod)//2))%mod
```

The latter references a variable `i`, so it must go in a for loop. Additionally, it accesses an array at index `i-1`, meaning `i` must always be greater than zero so that the index is never negative and stays valid. Therefore, this line must go in the for loop that counts from 1. (Technically, this line having a negative index is possible because python treats negative indices as counting from the end of the list, and not an error, however I decided that this was unlikely enough to rule out.)

The other line references the variable `mod`, so its definition is also included.

``` python
def decrypt(inp):
    # ...
    
    mod = 256
    l3 = [0]*n
    l3[0] = l2[0]%mod
    
    for i in range(1, n):
        l3[i] = (l2[i]^((l[i]&l3[i-1]+(l3[i-1]*l[i])%mod)//2))%mod
```

`l4` is the most straightforward:

``` python
def decrypt(inp):
    # ...
    
    l4 = [70, 123, 100, 53, 123, 58, 105, 109, 2, 108, 116, 21, 67, 69, 238, 47, 102, 110, 114, 84, 83, 68, 113, 72, 112, 54, 121, 104, 103, 41, 124]
```

Finally, we create the flag. It is initially defined as an empty string:

``` python
flag = ""
```

It is modified on one line, which adds a single character to the flag:

``` python
flag+=chr((l4[i]^l3[i]))
```

Again, it references `i`, so it goes in a for loop. There are still both kinds of for loop left to use, but we can figure out which to use. We know that the length of `inp` is equal to the flag length based on the [main function]
, and `n` is the length of `inp`, so the flag must have `n` characters. This means this line should be in the for loop that runs `n` times.

``` python
def decrypt(inp):
    # ...
    
    flag = ""
    
    for i in range(n):
        flag+=chr((l4[i]^l3[i]))
```

We can now go back to `l2`. We only have one for loop left, so we'll put both lines inside of it. The other line is placed after `l2` is populated, since it references `l2`.

``` python
def decrypt(inp):
	# ...
    
    l2 = [0]*n
    
    for i in range(1, n):
        l2[i] = (l[i]*5+(l2[i]+n)*l[i])%l[i]
        l2[i] += inp[i]
    
    l2[0] +=int(recur(l2[1:])*50)
    
    # ...
```

All that's left is returning the flag, and we have the final `decrypt` function:

``` python
def decrypt(inp):
	n = len(inp)
    
	l = []
    
	for i in range(n):
        l.append(random.randint(6, 420))
    assert(len(l)==n)
    
    mod = 256
    l3 = [0]*n
    l3[0] = l2[0]%mod
    
    l2 = [0]*n
    
    for i in range(1, n):
        l2[i] = (l[i]*5+(l2[i]+n)*l[i])%l[i]
        l2[i] += inp[i]
    
    l2[0] +=int(recur(l2[1:])*50)
    
    for i in range(1, n):
        l3[i] = (l2[i]^((l[i]&l3[i-1]+(l3[i-1]*l[i])%mod)//2))%mod
    
    l4 = [70, 123, 100, 53, 123, 58, 105, 109, 2, 108, 116, 21, 67, 69, 238, 47, 102, 110, 114, 84, 83, 68, 113, 72, 112, 54, 121, 104, 103, 41, 124]
    
    flag = ""
    
    for i in range(n):
        flag+=chr((l4[i]^l3[i]))
    
    return flag
```

### Final Code

Putting everything together, we get the unscrambled program:

``` python
import random
seed = 1000
random.seed(seed)

def recur(lst):
    if(len(lst)==1):
        assert(lst[0]>0)
        return lst[0]
    
    return recur(lst[::2])/recur(lst[1::2])

def decrypt(inp):
    n = len(inp)
    
    l = []
    
    for i in range(n):
        l.append(random.randint(6, 420))
    assert(len(l)==n)
    
    l2 = [0]*n
    
    for i in range(1, n):
        l2[i] = (l[i]*5+(l2[i]+n)*l[i])%l[i]
        l2[i] += inp[i]
    
    l2[0] +=int(recur(l2[1:])*50)
    
    mod = 256
    l3 = [0]*n
    l3[0] = l2[0]%mod
    
    for i in range(1, n):
        l3[i] = (l2[i]^((l[i]&l3[i-1]+(l3[i-1]*l[i])%mod)//2))%mod
    
    l4 = [70, 123, 100, 53, 123, 58, 105, 109, 2, 108, 116, 21, 67, 69, 238, 47, 102, 110, 114, 84, 83, 68, 113, 72, 112, 54, 121, 104, 103, 41, 124]
    
    flag = ""
    
    for i in range(n):
        flag+=chr((l4[i]^l3[i]))
    
    return flag

def main():
    flag_length = 31
    inp = [1]*flag_length
    print("flag is:", decrypt(inp))

main()
```

Running this gives the flag:
```
tjctf{unshuffling_scripts_xdfj}
```
