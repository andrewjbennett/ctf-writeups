# Fzz Buzz 2 - 200 points

Oh no! Two of my keys are broken! Please help me make the same Fzz Buzz program, sans that one letter and question marks.

As a side note, use of `eval()` and `exec()` is also frowned upon and will be marked invalid.

## Solution

The goal of this challenge was to write a fizz buzz program without using the letter `i`, or the question mark character `?`. You must read an integer `n` from stdin and print that many lines of fizz buzz. For example, if `n = 17`, output:

```
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
16
17
```

## Getting started

A simple FizzBuzz solution full of `i`s looks something like this:

```python
n = int(raw_input())
for x in range(1,n+1):
    if x % 15 == 0:
        print "FizzBuzz"
    elif x % 5 == 0:
        print "Buzz"
    elif x % 3 == 0:
        print "Fizz"
    else:
        print x
```

There are a couple of pretty essential `i`s in there:

Input/Output:
* `input`
* `print`

Looping:
* `in`

Logic:
* `if`

And, of course, FizzBuzz itself:
* FizzBuzz

Let's break it down category-by-category.

### Input/Output
This was my starting point, and iniital struggle -- how can I possibly do input and printing without the character `i`?
At this point, I hadn't decided on a language, and was trying to work out if I could possibly make it work in any of the accepted languages (C and Python, I think?)

I couldn't find a working solution to get around needing "main" in C -- I could write `_start()` myself, but without calling `exit()` myself at the end, it would just segfault.

However, in Python I worked out I could do some magic via the `__builtins__` object, with the help of "dial a friend"<sup id="a1">[1](#f1):

----

*`import foo`  
`method_to_call = getattr(foo, 'bar')`  
`result = method_to_call()`*  
*Apparently works  
I don't think you can directly call strings  
But you might be able to use `getattr()` on `globals()['__builtin__']`  
And that takes the string of the method*

----


And indeed, this was my starting point:

```
In [4]: getattr(globals()['__builtins__'], 'raw_input')
Out[4]: <function raw_input>
```

Although I needed two `i`s there, for `__builtins__` and `raw_input`, because they were both strings, I could easily construct them:

```python
eye = chr(0x69) 
bultn = '__bu' + eye + 'lt' + eye + 'ns__'
raw_n = 'raw_' + eye + 'nput'
```

We can use our new-found powers to print:

```python
prnt = 'pr' + eye + 'nt'
_raw_1nput = getattr(globals()[bultn], raw_n)
_pr1nt = getattr(globals()[bultn], prnt)
```

```python
>> _pr1nt("hello, world!")
hello, world!
```

Of course, we can also construct the strings Fizz and FizzBuzz in a similar way:
```python
>>> print "F"+eye+"zz", "Buzz", "F"+eye+"izzBuzz"
Fizz Buzz FizzBuzz
```

---

<sup><sub><a id="f1">1</b>: I don't consider "ask a friend for ideas on how to do this" to be cheating, on the basis that they weren't playing for another team in the competition, and my team had free slots; if it came down to it, said friend could just register for my team.)  [↩](#a1)</sub></sup>

----

### Logic


So, next up, we need a way to actually apply the logic. 

Something along the lines of `if x % 3 == 0` won't work, so how can we do this without using `if`?

It turns out I actually (re)learned a really cool trick from my students recently, who were trying to solve the challenge of "sort three numbers without using if statements": in C, you can do something like this:

```C
// equivalent to "if (a < b) smallest = a;"
a < b && smallest = a;
```

Is there some way I can apply something similar here?

Turns out I couldn't do something like that in python, but I **could** do something like this:

```python
# x = 5, y = 3
>>> print ["no", "yes"][x > y]
yes
```

And carrying on along that line of thought, with another trick from my students:
```C
// if (a < b) is true, then it will evaluate to 1
// so, this will evaluate to either 'a' or 'b', depending on which is smallest
// (1*a + 0*b, or 0*a + 1*b)
smallest = (a < b) * a + (b < a) * b;
```

Applying that here:

```python
# prints '2' if x is divisible by 5; '1' if x is divisible by 3, and
# '3' (2+1) if x is divisible by both 5 and 3
>>> print (x % 5 == 0) * 2 + (x % 3 == 0) * 1
```

We can use this to index into an array of possible values....

```python
# written using the banned character 'i' for ease of understanding
[x, "Fizz", "Buzz", "FizzBuzz"][(x % 5 == 0)*2 + (x % 3 == 0)*1]
```

So, if x was not divisible by 3 or 5, it would end up being `0*2 + 0*1`, which is zero, which is the index of `x` in our array.  
If x was divisible by 3, we'd end up with `0*2 + 1*1` = 1 = the index of `Fizz`, and so on.

-------

My thought-dump, as I was working through this section:
```
so in c I can do like "a == b && c = d"
but in python I can't? 😢
ooh
print ["no", "yes"][x > y]
print (x % 5 == 0) * 2 + (x % 3 == 0) * 1
I've learned so many useful tricks from my 1511 students doing the challenge exercise 😛
```

-------

### Looping

So, we've solved printing, we've solved conditionals... what about looping?

There were two ways I could think of to loop in python: `while`, and `for x in`, both of which unfortunately contained the letter `i`.

I had a look through the [Python built-in functions documentation page](https://docs.python.org/2/library/functions.html), looking for any functions that might possibly be useful, and I came across `any()`:

----

**any(iterable)**  
Return True if any element of the iterable is true. If the iterable is empty, return False. Equivalent to:

```
def any(iterable):
    for element in iterable:
        if element:
            return True
    return False
```
----

So, this looked like it could be pretty handy: I could just build myself an iterable way of going through all of my numbers, then call the `any` function on that to get it to loop through them all.

(note: I totally failed to remember that recursion existed. whoops.)

So, creating my class that will loop through all of the objects:

```python
class Foo(object):
    val = -1
    def __next__(self):
    	self.val += 1
        # mod3: 1, mod5: 2, mod15: 3, none: 0
     	_pr1nt([self.val, "F"+eye+"zz", "Buzz", "F"+eye+"zzBuzz"][(self.val % 5 == 0) * 2 + (self.val % 3 == 0) * 1])
    next = __next__ # python2 support
```

This worked great... except that I had no way to make it stop.

Oh, and also, I had no way to create an iterator (`__iter__`) without the letter `i`. At least that one was easily fixable:

```python
eter = '__' + eye + 'ter__'
setattr(Foo, eter, __1ter__)
```

To make it stop, I realised I could just make it iterate through a fixed size array of input numbers, thus throwing an exception once it hits the end and can't iterate any further:

```
setattr(Foo, 'mx', range(1,n+1))
```

Now we're getting somewhere!

----

My thought-dump, as I was working through this section:
```
okay, but... what about 'while'
and 'for x in'
hm
okay I have created a class which is an iterator which does fizzbuzz if you call eg any(foo())
unfortunately iterator involves i...
also __init__...
wonder if it'll make a blank __init__ that I can set
```

And then...

```
oh baby I did it
https://gist.github.com/andrewjbennett/8d99fcf0f7fc01c6ef9a365f4a1f84da
```


# FizzBuzz


```python
# andrew bennett
# F.zzBuzz w.thout any eyes
# EasyCTF 2017

# Variable naming convention: 
# strings have their 'i' removed
# objects start with '_' and have their 'i' replaced with '1'

eye = chr(0x69) 
bultn = '__bu' + eye + 'lt' + eye + 'ns__'
raw_n = 'raw_' + eye + 'nput'
ent = eye + 'nt'
prnt = 'pr' + eye + 'nt'

_raw_1nput = getattr(globals()[bultn], raw_n)
_1nt = getattr(globals()[bultn], ent)
_pr1nt = getattr(globals()[bultn], prnt)

n = _1nt(_raw_1nput())

def __1ter__(x):
  return x

class Foo(object):
    val = -1
    def __next__(self):
    	self.val += 1
      # mod3: 1, mod5: 2, mod15: 3, none: 0
     	_pr1nt([self.mx[self.val], "F"+eye+"zz", "Buzz", "F"+eye+"zzBuzz"][(self.mx[self.val] % 5 == 0) * 2 + (self.mx[self.val] % 3 == 0) * 1])
    next = __next__
	     

eter = '__' + eye + 'ter__'
setattr(Foo, eter, __1ter__)
setattr(Foo, 'mx', range(1,n+1))

try:
    any(Foo())
except:
    pass

```
