Administrative Scripting with Julia
===================================

Note that this tutorial is still in process. Eventually the parts that
are only in notebooks will be moved into this file with a script, but
for the moment, you can look at the notebooks directly. The intended
order, after the introduction is:

- files
- CLI
- filesystem
- processes
- regex (still being written)

However, each part should theoretically stand on its own to some extent.

Those parts which have been written should nonetheless be considered
drafts for the moment, but you may still find them useful.

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


Reading and Writing Files
=========================

I like to start going over administrative scripting with the topic of
files because files are fundamental to the way a Unix system thinks
about data. If the filesystem were relational database, files would be
the tables, and each line would be like a record. This is obviously not
true of every file, but it is a pervasive pattern. To the system, files
are not only data stored on disk. They can be anything that can do IO
streaming. Devices attached to the computer show up as files, sockets
can show up as files and many other things as well.

Opening Files
-------------

.. code:: julia

    # write some text into a file
    io = open("foo.txt", "w")
    println(io, "Some text concerning foo.")
    close(io)
    
    # read the text from a file
    io = open("foo.txt")
    read(io, String)




.. parsed-literal::

    "Some text concerning foo.\n"



Opening files is similar to Python3. There is an ``open`` method which
takes then name of the file as a string, and a number of mode arguments
after, and returns an ``IO`` object. The modes you'll most often be
using are ``"r"``, ``"w"`` and ``"a"``, for *read*, *write* and
*append*. These correspond to ``<``, ``>`` and ``>>`` in the shell.
``"r"`` is the default. There are more mode arguments, and you can read
about them in the `documentation for
``open`` <https://docs.julialang.org/en/v1/base/io-network/#Base.open>`__.
There is a ``write`` function for writing to files, but ``print`` and
``println`` work just as well, and they will convert any non-string
arguments to a string representation before sending it to the file. The
``write`` function, however, can also take an array of bytes (``UInt8``,
in Julia parlance) and send those to the specified stream as well.

Likewise, ``read`` can also return an array of bytes. In fact, this is
the default behavior. This is why, in our first example, the second
argument, ``String`` is used. Here is the result if it is omitted:

.. code:: julia

    # return to beginning of file
    seek(io, 0)
    
    show(read(io))
    close(io)


.. parsed-literal::

    UInt8[0x53, 0x6f, 0x6d, 0x65, 0x20, 0x74, 0x65, 0x78, 0x74, 0x20, 0x63, 0x6f, 0x6e, 0x63, 0x65, 0x72, 0x6e, 0x69, 0x6e, 0x67, 0x20, 0x66, 0x6f, 0x6f, 0x2e, 0x0a]

We've also seen the ``close`` function so far. This cleans up the file
descriptor for the system and flushes and data remaining in buffers.
However, you normally won't call it yourself. For one, if you want to be
lazy, the file descriptor will be cleaned up when the IO object is
garbage-collected, so you *can* ignore it, espeically if you're not
opening many files. However, if you are opening a lot of files and you
aren't sure when the garbage collector runs (like me), There are other
ways to do it. The first one is functionally similar to a context
manager in Python, but it looks a little different.

In Python, you'd write:

.. code:: python

    with open("foo.txt") as io:
        print(io.read())

In Julia, it's a `do
block <https://docs.julialang.org/en/v1/manual/functions/#Do-Block-Syntax-for-Function-Arguments-1>`__:

.. code:: julia

    open("foo.txt") do io
        print(read(io, String))
    end


.. parsed-literal::

    Some text concerning foo.


Do blocks are useful because, like Python context manager, they still do
the cleanup step even if an exception is thrown inside the block.
However, Julia has better shortcuts than that. Many functions that would
take a readable ``IO`` instance as their argument can take the name of
the file directly instead.

.. code:: julia

    read("foo.txt", String)




.. parsed-literal::

    "Some text concerning foo.\n"



The do-block version is always the safest if you're doing anything
inside the block besides just calling the "read" function, but it
doesn't make a big difference if you're not planning on using up all
your file descriptors. Now let's get rid of that file and get to the
next section.

.. code:: julia

    rm("foo.txt")
    # yes, that's really how you remove a file in Julia.

Iterating on Files
------------------

.. code:: julia

    # setup a dummy file for this section
    open("dummy.txt", "w") do io
        print(io,
            """
            The first line
            Another line
            The last line
            """)
    end
    print(read("dummy.txt", String))
    
    # Note that Julia truncates lines in triple-quote strings so you can still
    # use pretty indentation.


.. parsed-literal::

    The first line
    Another line
    The last line


Reading a file as a chunck of text is fine, but Unix tools really need
to be able to break files into lines and deal with them one line at a
time. In Julia, there are a couple ways to do this. The first is using
``readlines`` to read the lines in the file into an array. Like
``read``, ``readlines`` can take an IO object or a filename as the first
argument.

.. code:: julia

    show(readlines("dummy.txt"))


.. parsed-literal::

    ["The first line", "Another line", "The last line"]

Notice that Julia has very shell-like instincts about this. Trailing
newlines are skipped automatically, whereas this takes an extra step in
any other language, including Perl, whose syntax is largely based on the
shell. If you want to ``keep`` the trailing newlines, that's also
possible, just not default.

.. code:: julia

    show(readlines("dummy.txt", keep=true))


.. parsed-literal::

    ["The first line\n", "Another line\n", "The last line\n"]

``readline`` will be fine for most files, but it's not good if you have
to read a large file that can't actually fit in memory. A more robust
way to deal with lines is lazily. That's what ``eachline`` is for. It
takes the same kind arguments as ``readlines``, but doesn't load
everything into memory at once. You just loop over it and get your
lines.

.. code:: julia

    for line in eachline("dummy.txt")
        println(repr(line))
    end


.. parsed-literal::

    "The first line"
    "Another line"
    "The last line"


``eachline`` will close the file when it reaches the end, but not if
iteration is interupted. Therefore, if the loop could be broken and
you're worried about running out of file descriptors, it's safer to use
a do block.

.. code:: julia

    open("dummy.txt") do io
        for line in eachline(io)
            # do something
        end
    end

There are many more functions you can use with ``IO`` objects, but this
covers the common case for administrative scripting. You can read the
`documentation <https://docs.julialang.org/en/v1/base/io-network/>`__ if
you want more info. We're moving on to `command-line
interfaces <foo>`__.

Command-Line Interfaces
=======================

In order to write flexible, reusable scripts, one must get information
from the user and also send it back to them. Hard codeing a bunch of
global constants is no way to live!

``stdin``, ``stdout`` and ``stderr``
------------------------------------

These are your standard streams. The reason I started with a section on
files was so I could get to these babies. They are ``IO`` objects that
every script starts with open, and they automatically close at the end.
They aren't "real" files, but they give the same interfaces as files
(besides ``seek``). ``stdin`` is open for reading and both ``stdout``
and ``stderr`` are open for writing.

As you probably know, you can send data to the ``stdin`` of a program by
piping the output of another program to it.

.. code:: bash

    $ ls / | grep "b"
    bin
    boot
    lib
    lib64
    sbin

You can also do by using file redirection.

.. code:: bash

    $ grep "b" < some_file
    ...

From inside the script, this looks like any other IO object, and you can
do whatever you need with the lines.

.. code:: julia

    for line in eachline(stdin)
        # do something
    end

However, the creators of Julia know that this is such a common case that
both ``readlines`` and ``eachline`` default to using stdin.
``eachline()`` is identical to ``eachline(stdin)``

``stdout`` is the easy one. You already know how to write to it: the
``print`` and ``println`` functions. You can also use ``write``, of
course, if you need to write binary data.

``stderr`` is exactly the same as stdout, but you'd explicitely state
that you wanted things to go there:

.. code:: julia

    println(stderr, "things are real bad in this script")


.. parsed-literal::

    things are real bad in this script


Normally, you want to send data to stdout that is suitable to be used by
the stdin of another program (maybe ``grep`` or ``sed``?), and
``stderr`` is for messages for the user about what's happening in the
script (error messages, logging, debugging info). For more advanced
logging, Julia provides a `Logging
module <https://docs.julialang.org/en/v1/stdlib/Logging/>`__ in the
standard library.

CLI Arguments
-------------

Another important way to get information from your users is through
command line arguments. As in most languages, you get an array of
strings. Unlike many languages, the first item in this array is *not*
the name of the program. That's in a global variable called
``PROGRAM_FILE``. That can also be useful, but we're just talking about
the ``ARGS`` array for now.

Here is a simple clone of ``cp``:

.. code:: julia

    # cp.jl

    function main()
        dest = ARGS[end]
        srcfiles = ARGS[1:end-1]
        
        if isdir(dest)
            dest = joinpath.(dest, basename.(srcfiles))
        end

        cp.(srcfiles, dest)
    end

    main()

Which you would use like this :

.. code:: bash

    $ julia cp.jl afile otherfile targetdir

We don't really need the main function here, it's just best practice to
put everything besides constants inside of a function in Julia for
performance reasons (globals are slow unless they are constants), and
because it leads to more modular, reusable code.

For more sophisticated argument parsing, two popular third-party modules
are `ArgParse.jl <https://juliaobserver.com/packages/ArgParse>`__ and
`DocOpt.jl <https://juliaobserver.com/packages/DocOpt>`__, which provide
similar interfaces to the Python modules of the same names.

    *Note on vectorized functions*:

    If you're new to Julia, you might have trouble understanding a
    couple of lines:

    ::

        dest = joinpath.(dest, basename.(srcfiles))

    and

    ::

        cp.(srcfiles, dest)

    These lines make use of the Julia's `dot syntax for vectorizing
    functions <https://docs.julialang.org/en/v1/manual/functions/#man-vectorized-1>`__
    as an alternative to loops. In the first case, ``srcfiles`` is a
    vector of strings. ``basename.(srcfiles)`` returns an array of the
    basename of each path in ``srcfiles``. It's the same as
    ``[basename(s) for s in srcfiles]``. Each element in this array is
    then joined with the original ``dest`` directory for the full path.
    Because this operation contains nested dot operations, they are all
    *fused* into a single loop for greater efficiency.

    Because ``dest`` can now either be a vector or a string,
    ``cp.(srcfiles, dest)`` can mean two different things: If ``dest``
    is still a string, something like this happens:

    .. code:: julia

        for file in srcfiles
            cp(file, dest)
        end

    If dest has become a vector, however, it means this:

    .. code:: julia

        for (file, target) in zip(srcfiles, dest)
            cp(file, target)
        end

    This is handy for our case because, no matter which type ``dest``
    has in the end, the vectorized version will do the right thing!

    For more on the nuances of vectorizing functions, check out the
    documentation on
    `broadcasting <https://docs.julialang.org/en/v1/manual/arrays/#Broadcasting-1>`__

Environment Variables and Config Files
--------------------------------------

Another way to get info from your user is from configuration settings.
Though it is not the approach I prefer, one popular way to do this is
using environment variables store settings, which are exported in
``~/.profile`` or some other shell configuration file. In Julia,
environment variables are stored in the ``ENV`` dictionary.

.. code:: julia

    for var in ("SHELL", "EDITOR", "USER")
        @show var ENV[var]
    end


.. parsed-literal::

    var = "SHELL"
    ENV[var] = "/usr/bin/zsh"
    var = "EDITOR"
    ENV[var] = "nvim"
    var = "USER"
    ENV[var] = "ninjaaron"


I personally prefer to use config files.
`TOML <https://github.com/toml-lang/toml>`__ seems to be what all the
cool kids are using these days, and it's also used by Julia's built-in
package manager, so that's probably not a bad choice. There is a
"secret" TOML module in the standard library which is vendor by ``Pkg``.

You can get at it this way:

.. code:: julia

    import Pkg: TOML
    @doc TOML.parse




.. math::

    Executes the parser, parsing the string contained within.
    
    This function will return the \texttt{Table} instance if parsing is successful, or it will return \texttt{nothing} if any parse error or invalid TOML error occurs.
    
    If an error occurs, the \texttt{errors} field of this parser can be consulted to determine the cause of the parse failure.
    
    Parse IO input and return result as dictionary.
    
    Parse string
    




Because it's vendored, it's probably considered an implementation detail
and subject to disappaer without notice. I don't know what the deal is.
Anyway, the library they vendor can be found
`here <https://github.com/wildart/TOML.jl>`__. There are a couple other
TOML libraries on juliaobserver.com. There are also a semi-official
looking packages under the JuliaIO org on github called
`ConfigParser.jl <https://github.com/JuliaIO/ConfParser.jl>`__ That can
deal with ini files a few other types. There is also a
`JSON.jl <https://github.com/JuliaIO/JSON.jl>`__. I'm pretty against
using JSON for config files, but there it is.

Filesystem Stuff
================

Paths
-----

Julia provides a lot of built-ins for working with paths in a
cross-platformy way.

.. code:: julia

    currdir = pwd()
    @show basename(currdir)
    @show dirname(currdir)
    readme = joinpath(currdir, "README.rst")


.. parsed-literal::

    basename(currdir) = "administrative-scripting-with-julia"
    dirname(currdir) = "/home/ninjaaron/doc"




.. parsed-literal::

    "/home/ninjaaron/doc/administrative-scripting-with-julia/README.rst"



``joinpath`` can join an arbitrary number of path elements. I found it
very strange that there was no ``splitpath`` method to return an array
of all path elements. There has only been a ``splitdir`` function, which
returns a tuple.

.. code:: julia

    splitdir(currdir)




.. parsed-literal::

    ("/home/ninjaaron/doc", "administrative-scripting-with-julia")



However, I'm happy to say that a ``splitpath`` method is included in the
1.1 release of Julia, for which a release candidate has just been
released (on 2019-1-1), so you should be able to do

.. code:: julia

    julia> splitpath(currdir)
    ["/", "home", "ninjaaron", "doc", "administrative-scripting-with-julia"]

... or something like that.

.. code:: julia

    @show splitext("README.rst")
    @show isdir(readme)
    @show isfile(readme)
    st = stat(readme)


.. parsed-literal::

    splitext("README.rst") = ("README", ".rst")
    isdir(readme) = false
    isfile(readme) = true




.. parsed-literal::

    StatStruct(mode=0o100644, size=7528)



``StatStruct`` instances have a lot more attributes than this, of
course. They have `all these
attributes <https://docs.julialang.org/en/v1/base/file/#Base.stat>`__ as
well. A couple of these attributes, like ``mtime`` and ``ctime`` are in
Unix time, so it might be good mention that you can convert them to a
human readable representation with the Dates module, which is in the
standard library. It will be covered more in a later section. (Note that
this pretty-printed date is just the way it prints. It is a data
structure.)

.. code:: julia

    import Dates
    Dates.unix2datetime(st.mtime)




.. parsed-literal::

    2019-01-02T12:58:42.201



There are many other methods available in Base which have names you
should already recognize, which I won't demonstrate now. Names include:
``cd``, ``rm``, ``mkdir``, ``mkpath`` (like ``mkdir -p`` in the shell),
``symlink``, ``chown``, ``chmod`` (careful to make sure youre mode
argument is in octal, ``0o644`` or whatever), ``cp``, ``mv``, ``touch``,
as well as a lot of tests like ``isfile``, ``isdir``, ``islink``,
``isfifo``, etc. You know what they do, and you can [read the docs] if
you need more. The one thing that's missing is ``ls``. That's called
``readdir``.

.. code:: julia

    readdir()




.. parsed-literal::

    7-element Array{String,1}:
     ".git"              
     ".gitignore"        
     ".ipynb_checkpoints"
     "CLI.ipynb"         
     "README.rst"        
     "files.ipynb"       
     "filesystem.ipynb"  



There's also a
```walkdir`` <https://docs.julialang.org/en/v1/base/file/#Base.Filesystem.walkdir>`__
which recursively walks the directory and returns tuples of
``(rootpath, dirs, files)`` which is rather handy.

There are a few things Julia still lacks in the filesystem department.
It doesn't support any kind of file globbing, but that's easy enough to
handle with regex or plain substring matching.

.. code:: julia

    [path for path in readdir() if occursin("ipynb", path)]




.. parsed-literal::

    4-element Array{String,1}:
     ".ipynb_checkpoints"
     "CLI.ipynb"         
     "files.ipynb"       
     "filesystem.ipynb"  



.. code:: julia

    # or
    filter!(p -> !startswith(p, "."), readdir())




.. parsed-literal::

    4-element Array{String,1}:
     "CLI.ipynb"       
     "README.rst"      
     "files.ipynb"     
     "filesystem.ipynb"



It also weirdly lacks a function for making hard links. Bah. I guess
that's what the `C
interface <https://docs.julialang.org/en/v1/manual/calling-c-and-fortran-code/>`__
is for. (I'm both thumping my chest and groaning inside as I say that,
but at least it is crazy easy to call C from Julia and is as efficient
as native calls)

.. code:: julia

    function hardlink(oldpath, newpath)
        # calling:  int link(char *oldpath, char *newpath)
        ret_code = ccall(:link, Int32, (Cstring, Cstring), oldpath, newpath)
        ret_code == 0 ? newpath : error("couldn't link link: $oldpath -> $newpath")
    end
    
    hardlink("README.rst", "foo.txt")
    @show stat("foo.txt").nlink
    rm("foo.txt")


.. parsed-literal::

    (stat("foo.txt")).nlink = 2


Course, using ``ccall`` sort of depends on, you know, knowing enough C
to read and understand C function declarations for simple things, and it
involves pointers and memory allocation crap if you want to do something
more serious. It's C. What did you expect?

Julia also lacks Python's easy, built-in support for compression and
archive formats, though third-party packages do exist for GZip and Zip
archives. Maybe I should work on an archiving library. Hm.

Anyhow, there's more than one way to skin that cat. One distinctive
feature of Julia is that is very clear after you use it a little, but
it's hard to point to any one thing, is that it wants to make it easy to
bootstrap whatever functionality you need into the language. The
``ccall`` API is part of that. It is used liberally in the
implementation of OS interfaces, as well as some of the mathematical
libraries (``ccall`` also works on Fortran). Though they aren't shipped
with Julia, the community also maintain PyCall.jl and RCall.jl, which
allow "native" calls into those runtimes for wrapping their libraries.
Macros are different example of the same thing. Language missing a
feature? Alter the semantics with a macro. Yet another example of this
"bootstrap-ability" of Julia is the ease with which it allows the
programmer to orchestrate the use of external processes.

To take the example of the above ``hardlink`` function, If programming
in C ain't your bag, Julia has really great support for running external
processes, so it is also possible (but rather slower) to simply do:

.. code:: julia

    hardlink(oldpath, newpath) = run(`link $oldpath $newpath`)

Running Processes
=================

.. code:: julia

    command = `link README.rst foo.txt`




.. parsed-literal::

    `[4mlink[24m [4mREADME.rst[24m [4mfoo.txt[24m`



What is it? It's glorious!

.. code:: julia

    typeof(command)




.. parsed-literal::

    Cmd



It's Julia's ``Cmd`` literal, and it's a thing of beauty. What has it
done? Nothing.

Command literals, though they look the same, are not like process
substitution in Perl, Ruby or Bash in that they execute a command and
return the output as a string. They are something so much better. The
create a ``Cmd`` instance which contains the arguments and some other
information, and that object can be sent to various different functions
to be executed in different ways. `The
documentation <https://docs.julialang.org/en/v1/manual/running-external-programs/>`__
gives a good description of how to use these little marvels, so I'll
just cover a few simple cases here and explain what makes these so
great.

The simplest thing you can do, and the thing you need most often, is
simply to run the command.

.. code:: julia

    filename = "foo.txt"
    run(`link README.rst $filename`)




.. parsed-literal::

    Process(`[4mlink[24m [4mREADME.rst[24m [4mfoo.txt[24m`, ProcessExited(0))



.. code:: julia

    run(`ls -lh $filename`)


.. parsed-literal::

    -rw-r--r-- 2 ninjaaron ninjaaron 7,7K Jan 26 14:50 foo.txt




.. parsed-literal::

    Process(`[4mls[24m [4m-lh[24m [4mfoo.txt[24m`, ProcessExited(0))



What actually happened here? Obviously we ran the ``link`` executable
and the ``ls`` executable on the local system, but maybe not in the way
you'd expect in other languages, the default methods for running
commands *generally* create a subshell and execute your input there. In
Julia, commands never get a shell. As far as I know, the only way to
give a command a shell would be to do so explicitely, something like
``bash -c echo "my injection vulnerability"``, but you really don't need
a shell, so that's fine. What Julia's command literals do is pass the
string to parser for a shell-like mini-language, which converts the
command into a vector of strings which will ultimately be handed to one
of the OS's ``exec`` familiy of functions--on \*nix. I don't know how
these things happen on Windows.

The result is that running commands in Julia is safe and secure by
default because the shell never has the chance to do horrible things
with user input.

What's more, while Julia's shell mini-language resembles POSIX syntax on
a surface level, it is actually much saner and safer. It's very easy to
convert a working Bash script to Julia, but the result will usually be
safer in the end, which you can't say in most languages! For example, in
a Bash script, you should not really do this:

.. code:: bash

    link README.rst $filename

You should always put double quotes around the variable, because
otherwise it will be expanded into multiple arguments on whitespace.
However, in Julia, interpolated strings are never expanded in this way.
Some things are expanded, however: iterables

.. code:: julia

    `echo $(1:10)`




.. parsed-literal::

    `[4mecho[24m [4m1[24m [4m2[24m [4m3[24m [4m4[24m [4m5[24m [4m6[24m [4m7[24m [4m8[24m [4m9[24m [4m10[24m`



As you can see, this is expanded by Julia before the command is even
run. These can also combine with other elements to make Cartesian
products in a way similar to how brace expansion works in the shell:

.. code:: julia

    `./file$(1:10)`




.. parsed-literal::

    `[4m./file1[24m [4m./file2[24m [4m./file3[24m [4m./file4[24m [4m./file5[24m [4m./file6[24m [4m./file7[24m [4m./file8[24m [4m./file9[24m [4m./file10[24m`



.. code:: julia

    words = ["foo", "bar", "baz"]
    numbers = 1:3
    `$words$numbers`




.. parsed-literal::

    `[4mfoo1[24m [4mfoo2[24m [4mfoo3[24m [4mbar1[24m [4mbar2[24m [4mbar3[24m [4mbaz1[24m [4mbaz2[24m [4mbaz3[24m`



As seen in some of these examples, using a ``$()`` inside of a command
doesn't do process substitution as in the shell, it does, uh, "Julia
substitution," as it would in a Julia string--aside from the expansion
of iterables.

Julia has some other nice, logical features around commands. For
example, when a process exits with a non-zero exit code in Bash, the
script just tries to keep going and do who know's what. Same goes for
starting processes in most other languages. That's just silly, and Julia
knows it.

.. code:: julia

    run(`link README.rst $filename`)


.. parsed-literal::

    link: cannot create link 'foo.txt' to 'README.rst': File exists


::


    failed process: Process(`link README.rst foo.txt`, ProcessExited(1)) [1]

    

    Stacktrace:

     [1] error(::String, ::Base.Process, ::String, ::Int64, ::String) at ./error.jl:42

     [2] pipeline_error at ./process.jl:785 [inlined]

     [3] #run#515(::Bool, ::Function, ::Cmd) at ./process.jl:726

     [4] run(::Cmd) at ./process.jl:724

     [5] top-level scope at In[8]:1


That's right: Finished processes raise an error when there is a non-zero
exit status in the general case. Why doesn't every other language do
this by default? No idea. There are cases where you don't want this,
like if you're using ``grep``, for example. ``grep`` exits 1 if no
matches were found, which isn't exactly an error.

You can avoid it by passing additional arguments to the ``Cmd``
constructor.

.. code:: julia

    run(Cmd(`link README.rst $filename`, ignorestatus=true))


.. parsed-literal::

    link: cannot create link 'foo.txt' to 'README.rst': File exists




.. parsed-literal::

    Process(`[4mlink[24m [4mREADME.rst[24m [4mfoo.txt[24m`, ProcessExited(1))



So the error message still goes to stderr, because it's from the process
itself, but it prevents a non-zero exit status from throwing an error.

Another nice feature that show that the Julia developers "get it" when
it comes to processes, is that basically any function that can be
applied to a file can be applied to a command literal.

.. code:: julia

    readlines(`ls`)




.. parsed-literal::

    6-element Array{String,1}:
     "CLI.ipynb"       
     "files.ipynb"     
     "filesystem.ipynb"
     "foo.txt"         
     "processes.ipynb" 
     "README.rst"      



.. code:: julia

    open(`tr a-z A-Z`, "w", stdout) do io
        println(io, "foo")
    end


.. parsed-literal::

    FOO


Julia also supports pipelines, of course, but not with the pipe
operator, ``|``. Instead, one uses the ``pipeline`` function, which is
also useful if you want to do more complex IO things. Rather than cover
all this here, I will once again direct the reader to the
`documentation <https://docs.julialang.org/en/v1/manual/running-external-programs/#Pipelines-1>`__,
where it is all laied out very clearly.

Word of warning to the reader: while it's wonderful that it's so easy
and safe to work with processes in Julia, keep in mind that starting a
process is very expensive for the OS relative t executing code in the
current process. Particularly inside of hot loops, You should look for a
way to do what you need directly in Julia first, and only resort to
calling process when there is no apparent way to do the needful
natively. It is so much slower.

One place where someone with a background writing shell scripts in other
languages, but not as much experience in other languages might be
tempted to use for string filtering utilities in coreutils--sed, grep,
awk, etc. This would usually be a no-no, so the next section will
provide a quick introduction about how to do the kinds of things you
frequently do with those tools using Julia's regular expressions.

.. code:: julia

    rm("foo.txt")

String Filtering and Manipulation (with regex and otherwise)
============================================================

This section is primarily for those used to writing shell scripts who
want to do similar kinds of string jobs as one does with coreutils. If
you're used to string manipulation in other programming languages, Julia
will not be dramatically different, but you may still want to read a
little just to see how the basics look.

Note on regex dialects that I originally wrote for the the `Python
tutorial <https://github.com/ninjaaron/replacing-bash-scripting-with-python>`__:

    One thing to be aware of is that Python's regex is more like PCRE
    (Perl-style -- also similar to Ruby, JavaScript, etc.) than BRE or
    ERE that most shell utilities support. If you mostly do sed or grep
    without the -E option, you may want to look at the rules for Python
    regex (BRE is the regex dialect you know). If you're used to writing
    regex for awk or egrep (ERE), Python regex is more or less a
    superset of what you know. You still may want to look at the
    documentation for some of the more advanced things you can do. If
    you know regex from either vi/Vim or Emacs, they both use their own
    dialect of regex, but they are supersets of BRE, and Python's regex
    will have some major differences.

This is also true for Julia, except that Julia's regex isn't "like"
PCRE, it uses the actual PCRE library. The canonical resource on this
dialect of regex is the `Perl regex
manpage <http://perldoc.perl.org/perlre.html>`__, but note that, while
Perl generally places regexes between slashes (``/a regex/``), Julia
regex literals look like this: ``r"a regex"``. Also be aware that julia
doesn't have the same kinds of operators for dealing with regexes, like
=~, s, m, etc. Instead, normal functions are used with regex literals,
as in JavaScript and Ruby.

how to ``grep``
---------------

If you want to check if a substring occurs in a string, julia has a
function called ``occursin`` for that.

.. code:: julia

    occursin("substring", "string containing substring")




.. parsed-literal::

    true



As with most functions dealing with substrings in Julia, ``occursin``
can also be used with regular expressions.

.. code:: julia

    occursin(r"\w the pattern", "string containing the pattern")




.. parsed-literal::

    true



So let's get a long array of strings to grep.

.. code:: julia

    filenames = split(read(`find -print0`, String), '\0')




.. parsed-literal::

    180-element Array{SubString{String},1}:
     "."                                                       
     "./.gitignore"                                            
     "./.ipynb_checkpoints"                                    
     "./.ipynb_checkpoints/processes-checkpoint.ipynb"         
     "./.ipynb_checkpoints/Regex-checkpoint.ipynb"             
     "./.ipynb_checkpoints/files-checkpoint.ipynb"             
     "./.ipynb_checkpoints/CLI-checkpoint.ipynb"               
     "./.git"                                                  
     "./.git/info"                                             
     "./.git/info/exclude"                                     
     "./.git/COMMIT_EDITMSG"                                   
     "./.git/hooks"                                            
     "./.git/hooks/pre-commit.sample"                          
     ⋮                                                         
     "./.git/objects/81"                                       
     "./.git/objects/81/60dad2f73025ce91ab7ad9fb75e501f1bf15e2"
     "./.git/objects/4c"                                       
     "./.git/objects/4c/c736cc4785b2050e9c86f18714175f97d3c239"
     "./.git/config"                                           
     "./CLI.ipynb"                                             
     "./README.rst"                                            
     "./processes.ipynb"                                       
     "./Regex.ipynb"                                           
     "./files.ipynb"                                           
     "./filesystem.ipynb"                                      
     ""                                                        



    Note 1: You wouldn't normally use ``find`` in a Julia script. You'd
    be more likely to use the ``walkdir`` function, documented
    `here <https://docs.julialang.org/en/v1/base/file/#Base.Filesystem.walkdir>`__.

    Note 2: the reason this is isn't just ``readlines(`find`)`` is that
    POSIX filenames can contain newlines. Isn't that horrible?
    ``-print0`` uses the null byte to separate characters, rather than a
    newline to avoid exactly this problem, since it's the only byte that
    is forbidden in a filename.

So, let's try to match some git hashes that have four adjecent letters.

.. code:: julia

    filter(s->occursin(r".git/objects/.*[abcde]{4}", s), filenames)




.. parsed-literal::

    7-element Array{SubString{String},1}:
     "./.git/objects/d0/0db2ebda0b296f6f08e54ad06f3102e7abdec6"
     "./.git/objects/9c/f63bd3bbeea6c067d1e08f762acce5ac8adfe0"
     "./.git/objects/33/c9b993c55a75a2424acae6f1bcc5dcbf1f1ef7"
     "./.git/objects/1c/3c450edb480db60f6c949adf0b5dccdaebfc64"
     "./.git/objects/68/0c692e7095ecab805f649885ccc0e32c63ae1b"
     "./.git/objects/92/1cab47e3aafe6adab84ffdd9b06a16c34fa2e0"
     "./.git/objects/b8/2403b2c7d4f507c4debdb47b46fb3754a3085c"



.. code:: julia

    # this can also be done with comprehension syntax, of course
    
    [fn for fn in filenames if occursin(r".git/objects/.*[abcde]{4}", fn)]




.. parsed-literal::

    7-element Array{SubString{String},1}:
     "./.git/objects/d0/0db2ebda0b296f6f08e54ad06f3102e7abdec6"
     "./.git/objects/9c/f63bd3bbeea6c067d1e08f762acce5ac8adfe0"
     "./.git/objects/33/c9b993c55a75a2424acae6f1bcc5dcbf1f1ef7"
     "./.git/objects/1c/3c450edb480db60f6c949adf0b5dccdaebfc64"
     "./.git/objects/68/0c692e7095ecab805f649885ccc0e32c63ae1b"
     "./.git/objects/92/1cab47e3aafe6adab84ffdd9b06a16c34fa2e0"
     "./.git/objects/b8/2403b2c7d4f507c4debdb47b46fb3754a3085c"



Notes about performance:

these examples are given for the sake of sympicity and nice print-outs,
but, in cases where you don't know the size of the input data in
advance, you will want to use generators rather than arrays. Generators
expressions look like list comprehensions, but are in parentheses rather
than brackets. For a streaming version of the filter function, use
``Iterators.filter``.
