---
layout: master
title: Gant -- Ant Task
---

# Ant Task

Some builds have to have Ant as the driver, but people still want to use Gant (rather than just Groovy). To
support this there is the Gant Ant task. The Gant jar contains the Gant Ant task:
org.codehaus.gant.ant.Gant.  With the Gant jar in the class path, we can create an instance of the Gant Ant
task by:

{%highlight xml%}
<taskdef name="gant" classname="org.codehaus.gant.ant.Gant"/>
{%endhighlight%}

Use the classpath or classpathref attribute or nested tags as needed in the
usual way.

## Attributes

The Gant Ant task supports the following attributes:

Attribute | Optional | Default value |
---|---|--- | ---
file | yes | build.gant | The path to the Gant file to use.
target | yes | default | The target to achieve.

As an example, explicitly stating both the file and the target a Gant build
can be initiated from an Ant script by:

{%highlight xml%}
<gant file="build.gant" target="test"/>
{%endhighlight%}

Because of the defaults this is equivalent to:

{%highlight xml%}
<gant target="test"/>
{%endhighlight%}

If the Gant file specifies the test target as the default target then this is
equivalent to:

{%highlight xml%}
<gant/>
{%endhighlight%}

## Nested Tags

### Targets

The target attribute specifies a single target, if multiple targets are to be
used then they must be provided using nested target tags.  A target tag must
have a value attribute.  So for example

{%highlight xml%}
<gantTarget value="test"/>
{%endhighlight%}

### Definitions

Definitions, equivalent of -D... options on the Gant command line when using
that, are provided to the Gant executed via the gant tag, by nested definition
tags.  Each definition tag has a name and a value attribute.  So, for example:

{%highlight xml%}
<definition name="skipTests" value="true"/>
{%endhighlight%}
