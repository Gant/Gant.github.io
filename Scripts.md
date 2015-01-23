---
layout: master
title: Gant -- Scripts
---

# Scripts

A Gant script is a Groovy script that contains target definitions, calls to a pre-defined AntBuilder, and
operations on some other predefined objects.  The two main pre-defined objects are called includeTargets and
includeTool.  includeTargets is for including targets defined in other Gant scripts or appropriately
constructed classes.  includeTool is for including classes that provide services for use in a Gant file.
The following script shows some examples:

{%highlight groovy%}
ant.property environment: 'environment'
ant.taskdef name: 'groovyc', classname: 'org.codehaus.groovy.ant.Groovyc'
includeTargets << gant.targets.Clean
cleanPattern << '**/*~'
includeTool << gant.targets.Ivy
{%endhighlight%}

In this case we include targets from a pre-compiled class called Clean. This defined an object cleanPattern
that we can add new patterns to. As well as classes we can use files:

{%highlight groovy%}
includeTargets << new File('source/org/codehaus/groovy/gant/targets/clean.gant')
{%endhighlight%}

It is also possible to use string literals for the situation where target definitions will be constructed
programmatically.  As an example, if we have a directory with a source code sub-directory containing Java
and Groovy source that needs compiling then we might have the Gant script:

{%highlight groovy%}
sourceDirectory = 'source'
buildDirectory = 'build'
includeTargets << gant.targets.Clean
cleanPattern << '**/*~'
cleanDirectory << buildDirectory
ant.taskdef name: 'groovyc', classname: 'org.codehaus.groovy.ant.Groovyc'
target(compile: 'Compile source to build directory.') {
   javac srcdir: sourceDirectory, destdir: buildDirectory, debug: 'on'
   groovyc srcdir: sourceDirectory, destdir: buildDirectory
}
{%endhighlight%}

Now from the command line we can issue the command "gant compile" or "gant clean". If we want a default
target then we specify what it is:

{%highlight groovy%}
setDefaultTarget compile
{%endhighlight%}
