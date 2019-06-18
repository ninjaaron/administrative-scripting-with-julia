Administrative Scripting with Julia
===================================

Note that this tutorial is still in process. The intended
order, after the introduction is:

- files_
- CLI_
- filesystem_
- processes_
- regex_ (still being written)

However, each part should theoretically stand on its own to some extent.

Those parts which have been written should nonetheless be considered
drafts for the moment, but you may still find them useful.

.. _files: 1-files.ipynb
.. _CLI: 2-CLI.ipynb
.. _filesystem: 3-filesystem.ipynb
.. _processes: 4-processes.ipynb
.. _regex: 5-regex.ipynb

Introduction
------------
If you know anything about Julia_, you probably know it's an interpreted
language which is gaining popularity for numeric computing that competes
with the likes of R, Matlab, NumPy and others. You've probably also
heard that it compiles to LLVM bytecode at runtime and can often be
optimized to within a factor of two of C or Fortran.

Given these promises, it's not surprising that it's attracted some very
high-profile users_.

I don't do any of that kind of programming. I like Julia for other
reasons altogether. For me, the appeal is that it feels good to write.
It's like all the things I like from Python, Perl and Scheme all rolled
into one. The abstractions, both in the language and the standard
library, just feel like they are always hitting the right points.

Semantically and syntactically, it feels similar to Python and Ruby,
though it promotes functional design patterns and doesn't support
classical OO patterns in the same way. Instead, it relies on structs,
abstract types, and multiple dispatch for its type system. Julia favors
immutability, but it's not strict. Julia's metaprogramming story is
simple yet deep. It allows operator overloading and other kinds of magic
methods. If that isn't enough, it has Lips-like AST macros.

Finally, reading the standard library (which is implemented mostly in
very readable Julia), you see just how pragmatic it is. It is happy to
call into libc for the things libc is good at. It's equally happy to
shell out if that's the most practical thing. Check out the code for
the download_ function for an instructive example. Julia is very happy
to rely on PCRE_ for regular expressions. On the other hand, Julia is
fast enough that many of the bundled data structures and primitives
are implemented directly in Julia.

While keeping the syntax fairly clean and straightforward, the Julia
ethos is ultimately about getting things done and empowering the
programmer. If that means performance, you can optimize to your heart's
content. If it means downloading files with ``curl``, it will do that,
too!

This ethos fits very well with system automation. The classic languages
in this domain are Perl and Bash. Perl has the reputation of being
"write only," and Bash is much worse than that! However, both are
languages that emphasize pragmatism over purity, and that seems to be a
win for short scripts. Julia is more readable than either of these, but
it is not less pragmatic. [#]_

This tutorial follows roughly the approach of my `Python tutorial`_ on
administrative scripting and may refer to it at various points. Note
that I've been using Linux exclusively for more than a decade and I'm
not very knowledgable about Windows or OS X. However, if people wish to
contribute content necessary to make this tutorial more compatible with
those platforms, I would be very happy to learn.

.. _Julia: https://julialang.org/
.. _users: https://juliacomputing.com/case-studies/
.. _download:
  https://github.com/JuliaLang/julia/blob/e7d15d4a013a43442b75ba4e477382804fa4ac49/base/download.jl
.. _PCRE: https://pcre.org/
.. _Python tutorial:
  https://github.com/ninjaaron/replacing-bash-scripting-with-python

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
  only had it's 1.1 release this year (2019). Many Linux distros don't
  have a package available. Ubuntu 18.04 (the latest one as I write
  this) doesn't have a package in the repository, though it is available
  as a snap.
- Julia has a fat runtime and it has a human-perceptible load time on a
  slower system. For the kinds of intensive problems it's targeted at,
  this is nothing. On a constrained server or an embedded system, it's
  bad.
- Julia's highly optimizing JIT compiler also takes a little time to
  warm up. There are ways to precompile some things, but who wants to
  bother for little scripts? The speed of the compiler is impressive for
  how good it actually is, but it's not instant.

The above are reasonable arguments against using Julia on a certain
class of servers. However, none of this stuff really matters on a
PC/workstation running an OS with current packages. If your system can
run a modern web browser, Julia's runtime is a pittance.

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
very familiar, and I suspect the feeling will be similar for Ruby
programmers. For us, becoming productive in Julia should only take a few
hours, though there are rather major differences as one progresses in
the language.

For a quick introduction to the language, the `learning`_ page has some
good links. The `Intro to Julia`_ with Jane Herriman goes over
everything you'll need to know to understand this tutorial. If you
choose to follow this tutorial, you will be guided to log into
juliabox.com, but you don't need to unless you want to. You can
download and run the `Jupyter Notebooks`_ locally if you wish, and you
can also simply follow along in the Julia REPL in a terminal.

The `Fast Track to Julia`_ is a handy cheatsheet if you're learning
the language

.. _official docs: https://docs.julialang.org
.. _learning: https://julialang.org/learning/
.. _Intro to Julia: https://www.youtube.com/watch?v=8h8rQyEpiZA&t=
.. _Jupyter Notebooks: https://github.com/JuliaComputing/JuliaBoxTutorials
.. _Fast Track to Julia: https://juliadocs.github.io/Julia-Cheat-Sheet/

Anway, let's get straight on to files_.
