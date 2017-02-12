---
layout: master
title: Gant – Man Page
---

# Man Page

{%highlight groovy%}
gant(1)                                                                     gant(1)



NAME
       gant - A build framework based on scripting Ant task using Groovy.


SYNOPSIS
       gant [options] [target [target2 [target3] ...]]


DESCRIPTION
       gant is a tool for working with Ant tasks but using Groovy as the scripting
       language rather than using XML as a specification language.

       By default it takes information from build.gant which describes the targets.


        -c,--usecache
              Whether  to  cache the generated class and perform modified checks on
              the file before re-compilation.

        -d,--debug
              Print debug levels of information.

        -f,--file <build-file>
              Use the named build file instead of the default, build.gant.

        -h,--help
              Print out this message.

        -l,--gantlib <library>
              A directory that contains classes to be used as extra Gant modules,

        -n,--dry-run
              Do not actually action any tasks.

        -p,--projecthelp
              Print out a list of the possible targets.

        -q,--quiet
              Do not print out much when executing.

        -s,--silent
              Print out nothing when executing.

        -v,--verbose
              Print lots of extra information.

        -C, --cachedir <cache-file>
              The directory where to cache generated classes to.

        -D <name>=<value>
              Define <name> to have value <value>.  Creates a variable named <name>
              for use in the scripts and a property named <name> for the Ant tasks.

        -L,--lib <path>
              Add a directory to search for jars and classes.

        -P,--classpath <path-list>
              Specify a path list to search for jars and classes.

        -T,--targets
              Print out a list of the possible targets.

        -V,--version
              Print the version number and exit.


AUTHOR
       Gant and this manpage written by Russel Winder <russel@winder.org.uk>.



Russel Winder                        2008-05-24                             gant(1)
{%endhighlight%}
