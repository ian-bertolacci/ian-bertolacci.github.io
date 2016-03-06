---
layout: post
title:  Writing Fibonacci in LLVM with llvmlite
date:   2016-03-05 18:42:06 -0700
categories: LLVM llvmlite python compilers programming
---
Whoa hey whats this? A blog? Man I should write one of those.

Today I'm going to show you something I _just_ got my hands on:
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">YES! I finally got my llvmlite install to work! Had to use the 0.10 dev version, but it works! <a href="https://t.co/6ZG5FwWWDh">https://t.co/6ZG5FwWWDh</a></p>&mdash; Ian Bertolacci (@IanBertolacci) <a href="https://twitter.com/IanBertolacci/status/706192233846874112">March 5, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Like all good coding blogs, the full code is [at the bottom](#full-code) (maybe I'll create a repo for it), but first lets get rant-y!

LLVM is cool (if difficult to build/install/use/link) and python is great!
So this is particularly exciting.

I've been interested in doing the LLVM [Kaleidoscope](http://llvm.org/docs/tutorial/LangImpl1.html) tutorial for ages, but dislike using C++, especially for personal projects.
Not to mention that LLVM itself can be quite unaccessable.
Have you ever built [LLVM from source](http://llvm.org/releases/download.html#3.7.1)?
If you want to hate yourself and your for a few hours try that sucker out.

Now, building llvmlite was not a walk in the park either (at least for me).
LLVM is a continuously developing project, and a moving target for end developers like the people at [Numba](http://numba.pydata.org/).
When things change in LLVM, it tends to break everyone's build process, which can be frustrating.

I myself am known as The Build Killer.
It seems that no-matter what I want to put together, the build or one of its dependencies is **always** broken, missing, the wrong version, or incorrectly installed.
A lot this probably stems from issues in the linux environment.
(Seriously could someone give a look at my [pip install problem?](http://superuser.com/questions/1019943/why-does-dnf-uninstall-python-pip-want-to-uninstall-fedora))
Start to mix this with the many myriad of ways to build and install and you're looking at a recipe for flying computers.

## Installing llvmlite
I recommend you try the [0.10 dev version](https://github.com/numba/llvmlite/releases/tag/v0.10.0.dev) (since thats what I'm using), but if you'd like, try the [0.9 release version](https://github.com/numba/llvmlite/releases/tag/v0.9.0) and see if it works for you.

I will give you a few pointers, but I'm not an expert in _my_ system, much less yours.
I do use Fedora 23 so keep that in mind.

I had to install these packages (some of which you'd _think_ were defaults) in order to get things to work.

1. llvm (llvm.x86_64)
2. libstdc++ (libstdc++.x86_64)
3. static-libstdc++ (libstdc++-static.x86_64)

I'm not sure what other dependencies (potentially with respect to python) that you might need.

Now in your downloaded/cloned llmvlite folder run  
`python ./setup.py build`  
and if/when that works  
`python ./setup.py install --user`

I use `--user` since I'm use to academic environments where you aren't root and can't install to /usr/lib and also because I've had problems with python setup/pip/easy_install/et al. not installing files with the correct permissions.

to ensure that it was installed, run  
`echo "import llvmlite" | python`

If nothing happened you're solid.  
If you got  
`Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named llvmlite`  
something went wrong.  

## LLVM and llvmlite

If you are not familiar with LLVM at all, I'd check out [this post](http://adriansampson.net/blog/llvm.html) by [Adrian Sampson](http://adriansampson.net/)

Before getting started I suggest a quick flyby of the [documentation](http://llvmlite.readthedocs.org/en/latest/), especially [the example](http://llvmlite.readthedocs.org/en/latest/ir/examples.html) where they construct a function from scratch and [then run it](http://llvmlite.readthedocs.org/en/latest/binding/examples.html), since I will be using quite a bit of all that code.

## Constructing the fibonacci function
Apart from hello_world, fibonacci is every programmer's go-to when briefly trying a new language/framework.  
So we are going to define the LLVM equivalent of
{% highlight python %}
def fibonacci( n ):
  if n <= 1:
    return 1;
  return fibonacci( n - 1 ) + fibonacci( n - 2)
{% endhighlight %}

Which will look something like:
{% highlight llvm %}
;; defines our fibonacci function as fn_fib
;; t1 = n
define i64 @"fn_fib"(i64 %".1")
{
fn_fib_entry:
  ;; t3 = (n <= 1)
  %".3" = icmp sle i64 %".1", 1
  ;; if t3 then goto fn_fib_entry.if else goto fn_fib_entry.endif
  br i1 %".3", label %"fn_fib_entry.if", label %"fn_fib_entry.endif"

;; Base case
fn_fib_entry.if:
  ;; return 1
  ret i64 1

;; Recursive case
fn_fib_entry.endif:
  ;; t6 = (n - 1)
  %".6" = sub i64 %".1", 1
  ;; t7 = (n - 2)
  %".7" = sub i64 %".1", 2
  ;; t8 = fn_fib( t6 ) ; ie fn_fib( n - 1 )
  %".8" = call i64 (i64) @"fn_fib"(i64 %".6")
  ;; t9 = fn_fib( t7 ) ; ie fn_fib( n - 2 )
  %".9" = call i64 (i64) @"fn_fib"(i64 %".7")
  ;; t10 = (t8 + t9)
  %".10" = add i64 %".8", %".9"
  ;; return t10
  ret i64 %".10"
}
{% endhighlight %}

Pretty simple.

And the LLVM construction reflects that.
Lets break down part by part whats going on (full code embedded below).

This is the only relevant import.  
The [ir layer module](http://llvmlite.readthedocs.org/en/latest/ir/index.html) defines all the relevant types and operations for creating the LLVM IR tree.

{% highlight python %}
from llvmlite import ir
{% endhighlight %}

First we need to define two types:

1. The 64 bit wide int type (int(64)).
2. The function :: int(64) -> int(64) type.

{% highlight python %}
# Create a 64bit wide int type
int_type = ir.IntType(64);
# Create a int -> int function
# first argument is return type, second argument is a list (or iterable thing)
# of the argument types, in order.
fn_int_to_int_type = ir.FunctionType( int_type, [int_type] )
{% endhighlight %}

Next we define the module that this code will exist in.
{% highlight python %}
module = ir.Module( name="m_fibonacci_example" )
{% endhighlight %}

Then we declare the function and create a basic block where its code will be placed into.

{% highlight python %}
# Create the Fibonacci function and block
fn_fib = ir.Function( module, fn_int_to_int_type, name="fn_fib" )
fn_fib_block = fn_fib.append_basic_block( name="fn_fib_entry" )
{% endhighlight %}

Now we can start generating code, but first we need to get the LLVM builder object.

{% highlight python %}
# Create the builder for the fibonacci code block
builder = ir.IRBuilder( fn_fib_block )
{% endhighlight %}

First step in fibonacci is to check if `n <= 1`.  
To do that, we need to first get the argument `n` from the function

{% highlight python %}
# Access the function argument
fn_fib_n, = fn_fib.args # trailing , to unwrap tuple
{% endhighlight %}

and create the constant int value '1' (lets also go ahead and make the int value '2').

{% highlight python %}
# Const values for int(1) and int(2)
const_1 = ir.Constant(int_type,1);
const_2 = ir.Constant(int_type,2);
{% endhighlight %}

Now we can do our comparison.
`builder.icmp_signed` will create the comparison instruction.
This may seem/be obvious to some, but the instruction implicitly will store to an auto-generated temporary value.
This is a big reason for using LLVM (at the most basic level of compiler development), since we don't have to do our own register allocation.

{% highlight python %}
# Create inequality comparison instruction
fn_fib_n_lteq_1 = builder.icmp_signed(cmpop="<=", lhs=fn_fib_n, rhs=const_1 )
{% endhighlight %}

We then build our conditional jump using `builder.if_then`, passing our predicate (the result of `fn_fib_n_lteq_1`) to it.
Importantly, this doesn't just create our branch instruction, it also creates the 'then' and 'endif' code-blocks.

Now, `builder.if_then` gives us a Generator object which we use by the `with` statement.
When we are inside that `with` block, we are appending instructions/blocks/etc to the 'then' block.
We simply return the constant value '1':
{% highlight python %}
# Create the base case
# Using the if_then helper to create the branch instruction and 'then' block if
# the predicate (fn_fib_n_lteq_1) is true ( ie if n <= 1 then ... )
with builder.if_then( fn_fib_n_lteq_1 ):
  # Simply return 1 if n <= 1
  builder.ret( const_1 )
{% endhighlight %}

Now we are appending instructions to the 'endif' block
First we add instructions to subtract 1 and 2 from n:
{% highlight python %}
# This is where the recursive case is created
# _temp1= n - 1
fn_fib_n_minus_1 = builder.sub( fn_fib_n, const_1 )
# _temp2 = n - 2
fn_fib_n_minus_2 = builder.sub( fn_fib_n, const_2 )
{% endhighlight %}

Then we call our fibonacci function on the results:

{% highlight python %}
# Call fibonacci( n - 1 )
# arguments in a list, in positional order
call_fn_fib_n_minus_1 = builder.call( fn_fib, [fn_fib_n_minus_1] );
# Call fibonacci( n - 2 )
call_fn_fib_n_minus_2 = builder.call( fn_fib, [fn_fib_n_minus_2] );
{% endhighlight %}

Then we add the results, and return it.

{% highlight python %}
# Add the resulting call values
fn_fib_rec_res =  builder.add( call_fn_fib_n_minus_1, call_fn_fib_n_minus_2 )

# Return the result of the addition
builder.ret( fn_fib_rec_res )
{% endhighlight %}

And that's it!  
That's how you build code in LLVM.

The `ir.Module` can actually be printed and we can see our results

{% highlight python %}
# Print the generated LLVM asm code
print( module )
{% endhighlight %}

{% highlight llvm %}
; ModuleID = "m_fibonacci_example"
target triple = "unknown-unknown-unknown"
target datalayout = ""

define i64 @"fn_fib"(i64 %".1")
{
fn_fib_entry:
  %".3" = icmp sle i64 %".1", 1
  br i1 %".3", label %"fn_fib_entry.if", label %"fn_fib_entry.endif"
fn_fib_entry.if:
  ret i64 1
fn_fib_entry.endif:
  %".6" = sub i64 %".1", 1
  %".7" = sub i64 %".1", 2
  %".8" = call i64 (i64) @"fn_fib"(i64 %".6")
  %".9" = call i64 (i64) @"fn_fib"(i64 %".7")
  %".10" = add i64 %".8", %".9"
  ret i64 %".10"
}
{% endhighlight %}

What's more, with llvmlite, we can actually run this!

First we need to import a few things.

{% highlight python %}
import llvmlite.binding as llvm
from ctypes import CFUNCTYPE, c_int
{% endhighlight %}

The [`llvmlite.bindings`](http://llvmlite.readthedocs.org/en/latest/binding/index.html) module contains all the LLVM virtual machine bindings (surprised?).
Not exactly sure what the importance of ctypes is, but we will be using it to actually run the function.

Now we initialize all the llvm virtual machine.

{% highlight python %}
# initialize the LLVM machine
# These are all required (apparently)
llvm.initialize()
llvm.initialize_native_target()
llvm.initialize_native_asmprinter()
{% endhighlight %}

And create the execution engine.

{% highlight python %}
# Create engine and attach the generated module
# Create a target machine representing the host
target = llvm.Target.from_default_triple()
target_machine = target.create_target_machine()
# And an execution engine with an empty backing module
backing_mod = llvm.parse_assembly("")
engine = llvm.create_mcjit_compiler(backing_mod, target_machine)
{% endhighlight %}

Now we parse our module.
Important note: module cannot be directly parsed by llvm.
`llvm.parse_*` takes either a llvm bytecode object, or a string of llvm ir code.
`ir.module` is neither, but casting it as string will give us the underlying llvm ir code.

{% highlight python %}
# Parse our generated module
mod = llvm.parse_assembly( str( module ) )
mod.verify()
# Now add the module and make sure it is ready for execution
engine.add_module(mod)
engine.finalize_object()
{% endhighlight %}

Now we get a function pointer, and (it appears) cast that function pointer using ctype.

{% highlight python %}
# Look up the function pointer (a Python int)
func_ptr = engine.get_function_address("fn_fib")

# Run the function via ctypes
c_fn_fib = CFUNCTYPE(c_int, c_int)(func_ptr)
{% endhighlight %}

`c_fn_fib` is now a callable function!

{% highlight python %}
# Test our function for n in 0..40
for n in xrange(0,40+1):
  result = c_fn_fib(n)
  print( "c_fn_fib({0}) = {1}".format(n, result) )
{% endhighlight %}

{% highlight bash %}
c_fn_fib(0) = 1
c_fn_fib(1) = 1
c_fn_fib(2) = 2
c_fn_fib(3) = 3
c_fn_fib(4) = 5
c_fn_fib(5) = 8
c_fn_fib(6) = 13
c_fn_fib(7) = 21
c_fn_fib(8) = 34
c_fn_fib(9) = 55
c_fn_fib(10) = 89
c_fn_fib(11) = 144
c_fn_fib(12) = 233
c_fn_fib(13) = 377
c_fn_fib(14) = 610
c_fn_fib(15) = 987
c_fn_fib(16) = 1597
c_fn_fib(17) = 2584
c_fn_fib(18) = 4181
c_fn_fib(19) = 6765
c_fn_fib(20) = 10946
c_fn_fib(21) = 17711
c_fn_fib(22) = 28657
c_fn_fib(23) = 46368
c_fn_fib(24) = 75025
c_fn_fib(25) = 121393
c_fn_fib(26) = 196418
c_fn_fib(27) = 317811
c_fn_fib(28) = 514229
c_fn_fib(29) = 832040
c_fn_fib(30) = 1346269
c_fn_fib(31) = 2178309
c_fn_fib(32) = 3524578
c_fn_fib(33) = 5702887
c_fn_fib(34) = 9227465
c_fn_fib(35) = 14930352
c_fn_fib(36) = 24157817
c_fn_fib(37) = 39088169
c_fn_fib(38) = 63245986
c_fn_fib(39) = 102334155
c_fn_fib(40) = 165580141
{% endhighlight %}


Awesome!  
Lets take it a bit further!
{% highlight bash %}
c_fn_fib(40) = 165580141
c_fn_fib(41) = 267914296
c_fn_fib(42) = 433494437
c_fn_fib(43) = 701408733
c_fn_fib(44) = 1134903170
c_fn_fib(45) = 1836311903
c_fn_fib(46) = -1323752223
c_fn_fib(47) = 512559680
c_fn_fib(48) = -811192543
c_fn_fib(49) = -298632863
c_fn_fib(50) = -1109825406
{% endhighlight %}

Whoops we integer overflowed!
64 bits is not big enough.

Lets change the integer type
{% highlight python %}
int_type = ir.IntType(128);
{% endhighlight %}

And try that.

{% highlight bash %}
; ModuleID = "m_fibonacci_example"
target triple = "unknown-unknown-unknown"
target datalayout = ""

define i128 @"fn_fib"(i128 %".1")
{
fn_fib_entry:
  %".3" = icmp sle i128 %".1", 1
  br i1 %".3", label %"fn_fib_entry.if", label %"fn_fib_entry.endif"
fn_fib_entry.if:
  ret i128 1
fn_fib_entry.endif:
  %".6" = sub i128 %".1", 1
  %".7" = sub i128 %".1", 2
  %".8" = call i128 (i128) @"fn_fib"(i128 %".6")
  %".9" = call i128 (i128) @"fn_fib"(i128 %".7")
  %".10" = add i128 %".8", %".9"
  ret i128 %".10"
}

Segmentation fault (core dumped)
{% endhighlight %}

Damn.
In particular, it faults on `result = c_fn_fib(n)`
I wonder who's issue that is...

### Conclusion
That was fun!
This provides a nice way to interact and learn LLVM.
If you're a compilers professor, maybe think about integrating this into your compilers course.

Though I did have some trouble.

The whole `builder.if_then` and `builder.if_else` producing contextlib.GeneratorContextManager things is confusing.
One would expect them to produce block objects that can be referenced and played with (say as entry positions to a phi node).
This is doubly confusing when you look at the [source code](https://github.com/numba/llvmlite/blob/master/llvmlite/ir/builder.py#L210), and it even looks like a `Block` object is being returned.

Maybe that's something that can be worked out.

However its a great start and I'll continue to use it.

Thanks to [Numba](http://numba.pydata.org/) [developers](https://github.com/numba) for their work and to you for reading!  
-Ian J. Bertolacci

### Full Code

{% highlight python %}
from __future__ import print_function
from llvmlite import ir
import llvmlite.binding as llvm
from ctypes import CFUNCTYPE, c_int

"""
Generate fibonacci function:
f(n) = 1 if n <= 1 else f(n-1) + f(n-2)
"""
# Create a 64bit wide int type
int_type = ir.IntType(64);
# Create a int -> int function
fn_int_to_int_type = ir.FunctionType( int_type, [int_type] )

module = ir.Module( name="m_fibonacci_example" )

# Create the Fibonacci function and block
fn_fib = ir.Function( module, fn_int_to_int_type, name="fn_fib" )
fn_fib_block = fn_fib.append_basic_block( name="fn_fib_entry" )

# Create the builder for the fibonacci code block
builder = ir.IRBuilder( fn_fib_block )

# Access the function argument
fn_fib_n, = fn_fib.args
# Const values for int(1) and int(2)
const_1 = ir.Constant(int_type,1);
const_2 = ir.Constant(int_type,2);

# Create inequality comparison instruction
fn_fib_n_lteq_1 = builder.icmp_signed(cmpop="<=", lhs=fn_fib_n, rhs=const_1 )


# Create the base case
# Using the if_then helper to create the branch instruction and 'then' block if
# the predicate (fn_fib_n_lteq_1) is true ( ie if n <= 1 then ... )
with builder.if_then( fn_fib_n_lteq_1 ):
  # Simply return 1 if n <= 1
  builder.ret( const_1 )


# This is where the recursive case is created
# _temp1= n - 1
fn_fib_n_minus_1 = builder.sub( fn_fib_n, const_1 )
# _temp2 = n - 2
fn_fib_n_minus_2 = builder.sub( fn_fib_n, const_2 )

# Call fibonacci( n - 1 )\
# arguments in a list, in positional order
call_fn_fib_n_minus_1 = builder.call( fn_fib, [fn_fib_n_minus_1] );
# Call fibonacci( n - 2 )
call_fn_fib_n_minus_2 = builder.call( fn_fib, [fn_fib_n_minus_2] );

# Add the resulting call values
fn_fib_rec_res =  builder.add( call_fn_fib_n_minus_1, call_fn_fib_n_minus_2 )

# Return the result of the addition
builder.ret( fn_fib_rec_res )

# Print the generated LLVM asm code
print( module )

"""
Execute generated code.
"""
# initialize the LLVM machine
# These are all required (apparently)
llvm.initialize()
llvm.initialize_native_target()
llvm.initialize_native_asmprinter()

# Create engine and attach the generated module
# Create a target machine representing the host
target = llvm.Target.from_default_triple()
target_machine = target.create_target_machine()
# And an execution engine with an empty backing module
backing_mod = llvm.parse_assembly("")
engine = llvm.create_mcjit_compiler(backing_mod, target_machine)

# Parse our generated module
mod = llvm.parse_assembly( str( module ) )
mod.verify()
# Now add the module and make sure it is ready for execution
engine.add_module(mod)
engine.finalize_object()

# Look up the function pointer (a Python int)
func_ptr = engine.get_function_address("fn_fib")

# Run the function via ctypes
c_fn_fib = CFUNCTYPE(c_int, c_int)(func_ptr)

# Test our function for n in 0..40
for n in xrange(0,40+1):
  result = c_fn_fib(n)
  print( "c_fn_fib({0}) = {1}".format(n, result) )
{% endhighlight %}
