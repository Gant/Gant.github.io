---
layout: master
title: Gant -- Groovy dependency and Ant task programming
---

# Gant

*Gant 1.9.11 released, 2014-05-05T16:27:00+01:00*

# Gant is Groovy Ant Scripting

Gant is a tool for scripting Ant tasks using Groovy instead of XML to specify the logic. A Gant
specification is a Groovy script and so can bring all the power of Groovy to bear directly, something not
possible with Ant scripts.  Whilst it might be seen as a competitor to Ant, Gant uses Ant tasks for many of
the actions, so Gant is really an alternative way of doing things using Ant, but using a programming
language rather than XML to specify the rules.  Here is an example Gant script:

In this script there are two targets, stuff and otherStuff -- the default target for this build is
designated as stuff and is the target run when Gant is executed from the command line with no target as
parameter.

Targets are closures so they can be called as functions, in which case they are executed as you expect, or
they can be dependencies to other targets by being parameters to the depends function, in which case they
are executed if and only if they have not been executed already in this run. (There is a page with some more
information on [Targets](Targets.html).)

You may be wondering about the stuff at the beginning of the script. Gant has two ways of using pre-built
sub-scripts, either textual inclusion of another Gant script or the inclusion of a pre-compiled class. The
example here shows the latter -- the class gant.targets.Clean is a class that provides simple clean
capabilities.

The default name for the Gant script is _build.gant_, in the same way that
the default for an Ant script in _build.xml_.

Gant provides a way of finding what the documented targets are:

The messages on this output are exactly the strings associated with the target name in the introduction to
the target.

## Gant "can eat its own dog food"

Gant used to be used for building and installing itself and is being used for various build tasks including
building Groovy and Java programs, static websites, LaTeX documents. Until recently, Gant was an integral
part of [Grails](http://www.grails.org) and [Griffon](http://www.griffon.org). The ability to have arbitrary
Groovy methods within the scripts makes Gant so much easier to work with that the mix of XML and Groovy
scripts that using Ant necessitates. But then maybe this is an issue of individual perception.

## Gant isn't really a "build framework"

Gant is just a lightweight façade on Groovy's AntBuilder.  It just a way of scripting Ant tasks using
Groovy.  Gant can be used to do build tasks, but it doesn't have the integrated artefact dependency
management, project lifecycle management, and multi-module/sub-project support that a fully fledged build
framework should provide.  [Gradle](http://www.gradle.org) on the other hand is a complete build framework
based on Groovy and Ivy.  If you just want to do some Ant task scripting then Gant is probably the tool you
need, but for replacing Ant and Maven as build frameworks (so as to get rid of all the XML and use Groovy),
then you probably need to consider [Gradle](http://www.gradle.org).

And yes, Gant is managed by a [Gradle](http://www.gradle.org) build.

It is probably worth noting that [Gradle](http://www.gradle.org) grew out of work done on Gant.

## Getting Gant

[Prepackaged Distributions](Prepackaged_Distributions.html) are for those who want to "load and go".

[Developer Access](Developer_Access.html) is for those who want to be right on the cutting edge.

## Endnote

If you give Gant a go and have some feedback, do let us know on the Gant User mailing list or perhaps the
Groovy User mailing list.

BTW Gant is not a toy, it is used for task management in [Grails](http://www.grails.org)! In fact there is a
very nice [page on the Grails site](http://docs.codehaus.org/display/GRAILS/Command_Line_Scripting.html) showing
how Gant scripts can be used in Grails.

If you have any issues with Gant then please add a JIRA issue into the Gant JIRA at Codehaus. The URL is
[http://jira.codehaus.org/secure/CreateIssue!default.jspa](http://jira.codehaus.org/secure/CreateIssue!default.jspa)
