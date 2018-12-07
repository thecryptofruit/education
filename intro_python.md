https://github.com/thecryptofruit/education# 
# Very quick intro to Python
This content is to help students enjoy using Python, a powerful and popular programming language, used in web applications, AI, data processing etc. It is a general purpose language, introduced in 1989. There are many Python tutorials already written, this one is tailored to our blockchain development course. Corrections and additional requests are welcomed.
Code blocks represent commands and their outputs are in new lines.

## Contents
* [Python environment](#python-environment)
* [Basic operations](#basic-operations)
* [Managing source files](#managing-source-files)
* [Working with files](#working-with-files)
* [Accessing the Internet](#accessing-the-internet)
* [Interacting with Bitcoin node](#interacting-with-bitcoin-node)


# Python environment
There are two major Python releases and even the older one is used often. Chances are, your system has installed both version 2.7 and version 3.x. Either way, there is nothing wrong if you install both, though one should be enough :) Version 2.7 is the last of the 2.x releases, though! Check your Python versions like so:
```
> python --version
Python 2.7.15

> python3 --version
Python 3.6.7
```
We can use Anaconda (www.anaconda.org) with Jupyter Notebook for a more GUI approach, or PyCharm by JetBrains.
We can also use any text editor, preferably one that is capable of Python syntax highlighting, such as Sublime Text.

# Basic operations
Python console is a powerful environment. Just run Python and once you are in the console, recognized by `>>>`, you can hack away. To quit, type `quit()` or press `Ctrl + D`.
For more complex scripts, `.py` files are usually used, see [Managing source files](#managing-source-files).

## Math
For example, let's calculate a sum of two numbers:
```
> python3
>>> 1 + 2
3
```
More involved mathematical operations are available in the `math` package, which you need to import using the keyword `import`:
```
>>> import math
>>> math.sqrt(4)
2
>>> math.ceil(1.1)
2
>>> math.floor(1.1)
1
```
To work with complex numbers, use `cmath`:
```
>>> import cmath
>>> cmath.sqrt(-2)
1.4142135623730951j
```
## Variables
There is no special declaration of variables. Once you use a variable for the first time, it is automagically created. There is no declaration of types, either. A variable can change its type by assigning different things to it, as well, so be careful.
```
>>> x = 1
>>> 1+x
2
>>> x="abc"
>>> 1+x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```
> There is no difference between single quotes or double quotes!

To find the type of a variable, use `type()`:
```
>>> x="abc"
>>> type(x)
<class 'str'>
```

## Control structures
First important rule: indentations matter! Instead of surrounding blocks of code with brackets, Python uses indentation, which is very important with loops. Use 4 spaces, not tabs (some IDEs let you set tabs as a specified number of spaces, which is very handy). Use `:` at the end of the control statement:
```
>>> if 1 < 2:
...     print('one is less than two')
... 
one is less than two
```
Range is a sequence of numbers: `range(start, end, step)`. Often comes handy in loops, like so:
```
>>> for i in range(0, 6, 2):
...     print(i)
... 
0
2
4
```

## Functions
We define a function with `def(parameters)` and a `:` at the end of the line. If a function returns a value, we use `return`, otherwise it can be omitted.
```
>>> def mysum(a, b):
...    return a + b
...
>>> mysum(1, 2)
3
```

## Collections
Python has 4 different types of general-purpose containers: dictionaries, lists, sets, tuples.
container | description
--------- | ---------
dict | key-value mapping; unordered, mutable, indexed
list | used instead of arrays; ordered, mutable, indexed
set | unordered, mutable, unindexed
tup | ordered, immutable, indexed

Dictionary is used with `{key:value}` or `dict(key:value)`:
```
>>> d = {'first':100, 'second':42}
>>> d['second']
42
```
Lists are used with `[]` or `list()`. 
```
>>> l = ['first', 'second']
>>> l[1]
'second'
>>> for x in l:
...     print(x)
... 
first
second
>>> len(l)
2
```
Lists also have several methods with self-explanatory names: `in, append(), insert(), pop(), remove(), sort()`:
```
>>> l.append('abc')
>>> l
['first', 'second', 'abc']
>>> l.sort()
>>> l
['abc', 'first', 'second']
```
Sets are used with `{}` or `set()` and have methods for adding a single element with`add()` , multiple elements with `update()`, and removing items with `remove()` or `discard()`:
```
>>> s = {'first', 'second'}
>>> s
{'first', 'second'}
>>> s.add('abc')
>>> s
{'first', 'abc', 'second'}
```
Tuples are used with `()`. They are immutable, we can't change, add or remove elements from it. They are faster than lists (we can convert a list into a tuple).
```
>>> t = ('first', 'second', 42)
>>> t[0:2]
('first', 'second')
```

## Strings
Characters are strings. We can access characters as substrings by index or range:
```
>>> x = 'abcdef'
>>> x[0]
'a'
>>> x[1:3]
'bc'
```
Strings are immutable. Once you assign a string to a variable, it can't be changed. A great example is with concatenation. Let's see it in steps.
The simplest way to concatenate strings is with `+` operator:
```
>>> 'abc' + 'def'
'abcdef'
```
Memory address of a variable can be viewed with `id()` method, which we usually convert to hex in order to be more understandable.
Let's check that the variable's address really changes when we concatenate strings with `+`:
```
>>> x = 'abc'
>>> hex(id(x))
'0x7a66af490650'
>>> x = x + 'def'
>>> hex(id(x))
'0x7a66af4023e8'
```
A more powerful and efficient concatenation is using `string.split(separator, max_splits)`.
```
>>> x = 'abc;def;ghi'
>>> x.split(';')
['abc', 'def', 'ghi']
>>> x.split(';',2)
['abc', 'def', 'ghi']
>>> x.split(';',1)
['abc', 'def;ghi']
```
There is also a string formatting operator. Mind the `%s` for strings, `%d` for unsigned decimals ([there are others, too](https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting)), and the assignments after the final `%`:
```
>>> 'The answer to %s is %u.' % ('everything', 42)
'The answer to everything is 42.'
```

# Managing source files
The source code is saved in `.py` files. To execute it, specify the source file as an argument to the python executable.
Let's create a source file and add a simple `print("Hello, world.")` to it via vim:
```
> touch mypydemo.py
> vim mypydemo.py
//do our thing in vim ...
> python3 mypydemo.py 
Hello, world.
```

### \_\_main__
`__main__.py` serves as an entry point. If we have multiple Python files in a directory and also have `__main__.py`, then we can run our program simply by passing the directory name.
Note that if we run a program by specifying its name, then it will run as `__main__`.
Let's prepare an example to make this more clear ... We will create 2 files inside the `my_dir` directory.
In `my_prog.py` put:
```
def my_prog_main():
	print("Name of the Python file running:", __name__)

if __name__ == '__main__':
	my_prog_main()
```
In `__main__.py` put:
```
import my_prog
my_prog.my_prog_main()
```
Finally, let's run it in two different ways - observe the output:
```
> python3 my_dir/my_prog.py
Name of the Python file running: __main__
> python3 intro_python
Name of the Python file running: my_prog
```

## Packages
One Python file is called a module. A collection of modules can be organized into a package.

### \_\_init__
`__init__.py` files are used to mark directories as packages, used for easier importing. Without it, we would need to import with the name of the file we are importing explicitly defined AND that file would needed to be in the same directory. An empty `__init__.py` file would suffice to let Python treat subdirectories as packages.
The code in this file is executed once any of the modules in the current directory are loaded! It could come handy with setting up logging, sessions etc.



# Working with files
Opening files in different modes: 
r ... read
w ... write (will create it if it doesn't exist)
r+ ... read and write
x ... create
a ... append

Open in binary mode by adding `b`. Open in text mode by adding `t` (default).

## Read from a file
Let's open a file for reading, read the whole of it, close it (or return back to the initial position with `seek(0)`), open it again and then read only one line:
```
>>> f = open("dummy_text.txt", "r")
>>> f.read()
'First line.\nSecond line.'
>>> f.close()
>>> f = open("dummy_text.txt", "r")
>>> f.readline()
'First line.\n'
>>> f.close()
```
We could also make the closing implicit, which is much nicer:
```
>>> with open("dummy_text.txt", "r") as f:
...    f.read()
... 
'First line.\nSecond line.'
```

## Write to a file
Let's create another dummy file and put two lines into it:
```
>>> fw = open("dummy_text_write.txt", "w")
>>> fw.write("This is something.")
18
>>> fw.write("More.")
5
>>> fw.close()
```
Note that write will output the number of characters written. If we would want a new line, we would need the add `\n`.

# Accessing the Internet
Use `urllib` or `requests`.
```
>>> import requests
>>> web_results = requests.get("https://duckduckgo.com")
>>> web_results.status_code
200
>>> web_results.text
'<!DOCTYPE html><!--[if IEMobile 7 ]> <html ...
```

# Interacting with Bitcoin node

## Prepare the environment
Let's prepare the environment, using regtest network.
The configuration file bitcoin.conf should look like this:
```
# [debug]
regtest=1

# [rpc]
rpcuser=usr
rpcpassword=123
```
Run bitcoin daemon in the background:
```
> nohup bitcoind &
[1] 1882
```
Check that everything is ok:
```
> bitcoin-cli -regtest getblockchaininfo
{
  "chain": "regtest",
  "blocks": 0,
...
```
Mine some bitcoins (we mine 101 blocks because coinbase transactions are spendable only after 100 confirmations):
```
> bitcoin-cli -regtest generate 101
[
  "12869aa313fb324189af27a7f2e925b676300de2d109cda5afc97fa10eaf4db7",
...
```
Check that we have newly mined bitcoins available to spend:
```
> bitcoin-cli -regtest listunspent
[
  {
    "txid": "8b92aef16b6c3d7e9de145b3b5394690032566fb328678b6fb3126e8c6494074",
    "vout": 0,
    "address": "mq7TZKPo7jXubsZzBPgqvbESguDqqZFbSX",
    "scriptPubKey": "2103ce459d69b0d7e461f17e09fb5bf1f3e5f80d0e8e3f8288e761112b3162d3b597ac",
    "amount": 50.00000000,
    "confirmations": 101,
    "spendable": true,
    "solvable": true,
    "safe": true
  }
]
```

## Use the package python-bitcoinlib to check the balance
Check our balance with a simplest script using [python-bitcoinlib](https://github.com/petertodd/python-bitcoinlib) package. Name it `python_btc_balance.py`:
```
import bitcoin
from bitcoin.rpc import Proxy
from bitcoin.core import COIN

bitcoin.SelectParams('regtest')
bc = Proxy()
bal = bc.getbalance()
print ("Balance in satoshis: ", bal)
# 1 COIN = 100,000,000 satoshis
print ("Balance in bitcoins: {0}".format(bal/COIN))
```
Run it and observe the output:
```
> python3 python_btc_balance.py 
Balance in satoshis:  5000000000
Balance in bitcoins: 50.0
```

