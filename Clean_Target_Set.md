---
layout: master
title: Gant -- Clean Target Set
---

# Clean Target Set

The Clean target set provides the targets clean and clobber. It is included
using the statement:

{%highlight groovy%}
includeTargets << gant.targets.Clean
{%endhighlight%}

The idea of having clean and clobber is to provide two levels of tidying up.
Cleaning is getting rid of generated files and directories that are very
transitory. Clobbering is intended to be removing all generated files and
directories. The clobber target depends on the clean target so clobbering
implies cleaning, no replication is needed. Many people just put all the
toyding up into the clean target and ignore the clobber target.

The clean and clobber targets separate processing of patterns and directories.
Specifying a clean pattern is handled with the cleanPattern and clobberPattern
objects which can be give a string or a list of strings:

{%highlight groovy%}
cleanPattern << '**/*~'
cleanPattern << ['**/*.bak', '**/*.pyc', '**/*.class']
clobberPattern << '*.pdf'
{%endhighlight%}

This is the standard Ant pattern syntax being used. Specifying directories to
remove is handled with the cleanDirectory and clobberDirectory objects which
can also take a string or list of strings:

{%highlight groovy%}
cleanDirectory << 'target/classes'
cleanDirectory << ['target/test-classes', 'target/test-reports']
clobberDirectory << 'target'
{%endhighlight%}

Stating what is hopefully obvious: the examples here use string literals but
Groovy variables can be used.
