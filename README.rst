Administrative Scripting with Julia
===================================

Introduction
------------
If you know anything about Julia_, you probably know it's an interpreted
technical/scientific computing language that competes with the likes of
R, Matlab, NumPy and others. You've probably also heard that it compiles
to LLVM bytecode at runtime and can often be optimized to within a
factor of two of C/C++.

Given these promises, it's not surprising that it's attracted some very
high-profile users_.

I don't do any of that kind of programming. I like Julia for other
reasons altogether. It just feels good to write. The language features
scratch all my itches and the standard library has all the right
abstractions.

Semantically and syntactically, it feels similar to Python and Ruby
though it promotes functional design patterns and doesn't support
classic OO patterns in the same way. Instead, it relies on structs,
abstract types, and multiple dispatch as tools for working with types.
Julia favors immutability, but it's not strict. Julia's metaprogramming
story is simple yet deep. It allows operator overloading and other kinds
of magic methods. If that isn't enough, it has Lips-like AST macros. It
tends toward orthogonal abstractions, but not slavishly so.

Finally, reading the standard library (which is implemented mostly in
very readable Julia), you see just how pragmatic it is. It is happy to
call into libc for the things libc is good at. It's equally happy to
shell out if that's the most practical thing. Check out the code for the
download_ function for an instructive example. Julia is very happy to
rely on PCRE_ for regular expressions. On the other hand, Julia is fast
enough that basically all of the bundled data structures and primitives
are implemented directly in Julia (with the exception of the fundamental
array types).

While keeping the syntax clean and straightforward, the Julia ethos is
ultimately about getting things done and empowering the programmer. If
that means performance, you can do manual memory management (though you
never have to). If it means downloading file with ``curl``, it will do
that, too!

This ethos fits very well with system automation. The classic languages
in this domain are Perl and Bash. Perl has the reputation of being
"write only," and Bash is much worse than that! [#]_ However, both are
languages that emphasize pragmatism over purity, and that seems to be a
win for one-off scripting. Julia is more readable than either of these,
but it is not less pragmatic. [#]_

This tutorial follows roughly the approach of my `Python tutorial`_ on
administrative scripting and will refer to it at various points.

.. _Julia: https://julialang.org/
.. _users: https://juliacomputing.com/case-studies/
.. _download:
  https://github.com/JuliaLang/julia/blob/e7d15d4a013a43442b75ba4e477382804fa4ac49/base/download.jl#L62-L70
.. _PCRE: https://pcre.org/
.. _Python tutorial:
  https://github.com/ninjaaron/replacing-bash-scripting-with-python

.. [#] Anyone who fully undersands the semantics of the Bash they write
       even at the time they write it is something of a domain expert.
       Simple, straightforward Bash is vulnerable to injection more
       often than not. A completely correct Bash script is either
       delegating all control flow to other processes, or it is using
       non-obvious, ugly language features to ensure safety.

.. [#] This is not to fault the creators of Perl or Bourne Shell. They
       are much older langauges, and all interpreted languages,
       including Julia, have followed in their footsteps. Later
       languages learned from their problems, but they also learned from
       what they did right, which was a lot!

.. contents:: 

Why You Shouldn't Use Julia for Administrative Scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
It's just a bad idea!

- Julia is not the most portable. It's a relatively new language and has
  only had it's 1.0 release this year (2018). Many Linux distros don't
  have a package available. Ubuntu 18.04 (the latest one as I write
  this) doesn't have a package in the repository, though it is available
  as a snap. Even if you can compile from source, it has that LLVM
  dependency.
- Julia has a fat runtime. It's more than 100KB and it has a
  human-perceptible load time on a slower system. For the kinds of
  intensive problems it's targeted at, this is nothing. On a
  constrained server or an embedded system, it's bad.
- Julia's highly optimizing JIT compiler also takes a little time to
  warm up. There are ways to recompile some things, but who wants to
  bother for little scripts? The speed of the compiler is impressive for
  how good it actually is, but it's not instant.

The above are reasonable arguments against using Julia on a certain
class of servers. However, none of this stuff really matters on a PC
running an OS with current packages. If your system can run a modern web
browser, Julia's ~140K runtime is a pittance. (By contrast, the Python
3.7 runtime is about 8.6K, Perl 5.28 is about 3.3K, just doing a quick
check on my system.)

If you already want to learn Julia, which there are many good reasons to
do, writing small automation scripts is a gentle way to become
acquainted with the basic features of the language.

The other reason you might want to try administrative scripting in Julia
is because the abstractions it provides are surprisingly well suited to
the task. Translating a Bash script to Julia is very easy but will
automatically make your script safer and easier to debug.

One final reason to use Julia for administrative is that it means you're
not using Bash! I've made a `case against Bash`_ for anything but
running and connecting other processes in Bash in my Python tutorial. In
short, Bash is great for interactive use, but it's difficult to do
things in a safe and correct way in scripts, and dealing with data is an
exercise in suffering. Handle data and complexity in programs in other
languages.

.. _case against bash:
  https://github.com/ninjaaron/replacing-bash-scripting-with-python#if-the-shell-is-so-great-what-s-the-problem


Learning Julia
~~~~~~~~~~~~~~
This tutorial isn't going to show you how to do control flow in Julia
itself, and it certainly isn't going to cover all the ways of dealing
with the rich data structures that Julia provides. To be honest, I'm
still in the process of learning Julia myself, and I'm relying heavily
on the `official docs`_ for that, especially the "Manual" section. As an
experienced Python programmer, the interfaces provided by Julia feel
very familiar, and I suspect the feeling will be even stronger for Ruby
programmers. For programmers with this background, becoming productive
in Julia should only take a few hours, though there are rather major
differences as one progresses in the language.

.. _official docs: https://docs.julialang.org


Julia in Five Minutes
+++++++++++++++++++++

Here are some code samples running in the Julia repl to get you
started.

.. code:: Julia

  julia> # control-flow structures end with "end"
  
  julia> for i in 1:3
             println(i)
         end
  1
  2
  3

  julia> function foo()
             "foo"
         end
  foo (generic function with 1 method)

  julia> foo()
  "foo"

  julia> # there is a "return" keyword, but functions will also return
         # the value of the last expression if it's not used.

  julia> # One-liner functions are also there:

  julia> f(r) = pi * r ^ 2     # ^ for exponent
  f (generic function with 1 method)

  juila> # short lambda:

  julia> (a, b) -> a - b
  #3 (generic function with 1 method)

  julia> # longer lambda

  julia> function(a, b)
             a * b
         end

Strings are a little different than Python or Ruby. Strings are always
in double quotes in Julia, like in C. There are also triple-quote
strings which act like Python programmers would expect. Single quotes
are for ``Char`` literals, again like C.

However, a ``Char`` in Julia is a unicode code-point. A String is not
made of Chars internally. It is a string of bytes in UTF8, but this only
comes into play when indexing (something best avoided). Iterating on a
string will give Chars.


.. code:: Julia

  julia> s = "gemütlig"    # "ü" is two bytes in UTF-8
  "gemütlig"

  julia> a = [c for c in s]
  8-element Array{Char,1}:
   'g'
   'e'
   'm'
   'ü'
   't'
   'l'
   'i'
   'g'

  julia> String(a)
  "gemütlig"

  julia> # string concatenation

  julia> string("Sehr ", s)
  "Sehr gemütlig"

  julia> # and Bash-like interpolation

  julia> "Sehr $s"
  "Sehr gemütlig"

  julia> # expressions can also be interpolated

  julia> "ten and ten is $(10 + 10)"
  "ten and ten is 20"


Arrays, while not really much at all like Python lists internally, look
and act similarly on a superficial level. One thing to know, though, is
that Julia is a 1-index language. Isn't that ridiculous? Yes, yes it is.

.. code:: Julia

  julia> a = []
  0-element Array{Any,1}

  julia> for c in "string"
             # by convention, names of functions with side-effects end with an "!"
             push!(a, c)
         end

  julia> a
  6-element Array{Any,1}:
   's'
   't'
   'r'
   'i'
   'n'
   'g'

  julia> a[0]
  ERROR: BoundsError: attempt to access 6-element Array{Any,1} at index [0]
  Stacktrace:
   [1] top-level scope at none:0

  julia> a[1]
  's': ASCII/Unicode U+0073 (category Ll: Letter, lowercase)

Because we know that ``a`` is going to be an array of ``Char`` s, it's
actually much more efficient to declare the type. Of course, type
declarations are never strictly necessary in Julia unless you're
leveraging multiple dispatch.

.. code:: julia

  julia> ca = Char[]
  0-element Array{Char,1}

  julia> append!(ca, "string")
  6-element Array{Char,1}:
   's'
   't'
   'r'
   'i'
   'n'
   'g'

  julia> # append!(Array, iterable) in Julia is like list.extend(iterable) in Python.

  julia> # Julia can also infer type in a comprhension

  julia> [c for c in "another string"]
  14-element Array{Char,1}:
   'a'
   'n'
   'o'
   't'
   'h'
   'e'
   'r'
   ' '
   's'
   't'
   'r'
   'i'
   'n'
   'g'

Julia also has literal syntax for matrices, and, though I don't
believe there is literal syntax, it supports arrays of arbitrary
dimensions. As someone who's never taken college math, I must admit
that I find it a little unnerving!

To find the end of a sequence, Julia uses the ``end`` keyword.

.. code:: Julia

  julia> s = "string"
  "string"

  julia> s[end]
  'g': ASCII/Unicode U+0067 (category Ll: Letter, lowercase)

  julia> s[1:end]
  "string"

  julia> s[2:end]
  "tring"

  julia> s[2:end-1]
  "trin"

Dictionaries in Julia don't have literal syntax, but they still aren't
too bad. It does have literal syntax for a "Pair", which uses the ``=>``
operator, which can be used to build up dictionares.

.. code:: Julia

  julia> d = Dict(
             "one" => 1,
             "two" => 2,
             "three" => 3
         )
  Dict{String,Int64} with 3 entries:
    "two"   => 2
    "one"   => 1
    "three" => 3

  julia> d["two"]
  2

  julia> a = ["foo", "bar", "baz"]
  3-element Array{String,1}:
  "foo"
  "bar"
  "baz"

  julia> d = Dict(v => i for (i, v) in enumerate(a))
  Dict{String,Int64} with 3 entries:
  "bar" => 2
  "baz" => 3
  "foo" => 1

If you're building lots of small dictionaries with the same keys, you
might consider a ``NamedTuple`` instead.

.. code:: Julia

  julia> nt = (foo=1, bar=2, baz=3)
  (foo = 1, bar = 2, baz = 3)

  julia> nt.foo
  1

  julia> typeof(nt)
  NamedTuple{(:foo, :bar, :baz),Tuple{Int64,Int64,Int64}}

Course, it's pretty easy to create arbitrary structs also, which some
might prefer.

.. code:: Julia

  julia> struct MyStruct
             foo
             bar
             baz
         end

  julia> ms = MyStruct(1,2,3)
  MyStruct(1, 2, 3)

  julia> ms.baz
  3

This would be the right time to introduce Julia's type system, but this
has already gone on long enough.

I guess the last thing to say is that, in Julia, you say ``catch``
instead of ``except`` and ``throw`` instead of ``raise``. That part
actually works more like JavaScript (as you might infer from the
naming conventions).

.. code:: Julia

  julia> try
             a[0]
         catch err
             if isa(err, BoundsError)
                 println("there was a bounds error!")
             else
                 rethrow(err)
             end
         end
  there was a bounds error!

There is also a ``finally`` clause which will be run whether an
exception is caught or not.


Reading and Writing Files
-------------------------
foo
