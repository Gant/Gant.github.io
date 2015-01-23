---
layout: master
title: Gant -- Gant and the Ant Optional Tasks
---

# Gant and the Ant Optional Tasks

A Groovy installation includes Ant (Groovy 1.5.0 ships with Ant 1.7.0), and
JUnit (Groovy 1.5.0 ships with JUnit 3.8.2). This means that Gant scripts can
use all the Ant core tasks and the junit optional task without any extra
installations. However, all the other optional tasks require extra action to
be used from Gant. There are three mechanisms available.

  1. The option "--classpath <path>" or "-P <path>" can be used to specify a path to add to the classpath.
  1. The jar can be put into ${user.home}/.ant/lib which is automatically searched.
  1. The environment variable ANT_HOME can be set, in which case all jars in ANT_HOME/lib are included in the classpath.
