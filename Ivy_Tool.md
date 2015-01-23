---
layout: master
title: Gant -- Ivy Tool
---

# Ivy Tool

[Ivy](http://ant.apache.org/ivy/) is a dependency management system for Ant.  As Gant uses Ant tasks, Gant
can make use of Ivy very straightforwardly.  Installing Gant installs the Ivy jar, so there are no missing
dependencies (Gant 1.2.0 installs Ivy 2.0.0-beta2). Gant provides an Ivy tool that is used to work with Ivy,
so you need the statement:

{%highlight groovy%}
includeTool << gant.tools.Ivy
{%endhighlight%}

to access the features.  This creates an object called Ivy which can then be used to invoke all the features
of the Ivy system, for example:

{%highlight groovy%}
ivy.cachepath(...)
ivy.cleancache()
{%endhighlight%}

The Ivy distribution has an example in it, that installs Ivy, ensures the local presence of the Commons-Lang
jar, compiles and runs a small test program. A Gantified version of part of this example is provided in the
Gant source. The Gantfile for this example shows how simple and straightforward using Ivy can be:

{%highlight groovy%}
buildDirectory = 'build'
sourceDirectory = 'source'

includeTargets << gant.targets.Clean
cleanDirectory << buildDirectory
cleanPattern << ['**/*~', '**/*.bak']

includeTool << gant.tools.Ivy

target(runTest: 'Run the Ivy "Hello" test.') {
  def classpathRef = 'libraryClasspath'
  ivy.cachepath(organisation: 'commons-lang', module: 'commons-lang', revision: '2.3',  pathid: classpathRef, inline: 'true')
  mkdir(dir: buildDirectory)
  javac(srcdir: sourceDirectory, destdir: buildDirectory, debug: 'true', classpathref: classpathRef)
  java(classname: 'example.Hello', classpathref: classpathRef) {
    classpath { pathelement location: buildDirectory }
  }
}

target(cleanCache: 'Clean the Ivy cache.') { ivy.cleancache() }

setDefaultTarget runTest
{%endhighlight%}

For full details of the Ivy features, it is best to check the Ivy manual pages for the version of Ivy
associated with your Gant installation.  To replicate the information here is probably just a bad idea, as
well as being a lot of work :-).
