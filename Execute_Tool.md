---
layout: master
title: Gant -- Execute Tool
---

# Execute Tool

Executing programs and shell scripts can be problematic. It generally involves
multiple processes and pipes, and pipes need to be emptied as well as filled
or everything can block. The techniques for dealing with this are well known
and indeed idiomatic, effectively just boilerplate code. So to avoid everyone
having to write the same thing many times, this tool provides the boilerplate
code.

To get access to the tool:

{%highlight groovy%}
includeTool << gant.tools.Execute
{%endhighlight%}

Two methods are then available, executable and shell. The former is used to
invoke an executable, whilst the latter invokes a shell that parses and
executes the command passed. So for example:

{%highlight groovy%}
execute.shell 'echo "Hello" && echo "World!"'
{%endhighlight%}

By default output from the executing program is passed through, the programs
standard out to the curtrent standard out, and the programms standard error to
the current standard error. If this is not the desired behaviour then keyword
parameters can be used to alter this. The keywords errProcessing and
outProcessing can be assigned closures which are executed in place of the
defaults.

{%highlight groovy%}
execute.shell 'echo "Hello" && echo "World!"', outProcessing: {}
{%endhighlight%}

causes all output from the shell on the standard output to be ignored.

Currently there are two overloads of executable, one taking a single string as
the command, teh other taking a list of strings as the command.
