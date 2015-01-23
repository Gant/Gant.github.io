---
layout: master
title: Gant -- Default Target
---

# Default Target

The default target, i.e. the target executed when no target is specified on the command line is specified by
defining the target 'default'. So for example:

{%highlight groovy%}
target('default', 'The default target.') { aTarget() }
{%endhighlight%}

This assumes aTarget exists!

Rather than requiring that the above mechanism is used, Gant provides a shortcut:

{%highlight groovy%}
setDefaultTarget aTarget
{%endhighlight%}

What this actually does is:

{%highlight groovy%}
target('default': 'aTarget') { aTarget() }
{%endhighlight%}

The convention of using the target name as the description is for creating the string at the end of the
project help output (-p, --projecthelp, -T, --targets options to Gant).

The parameter to the setDefaultTarget call can be either the name of a target or a string that evaluates to
the name of a target.  So for the above example we could have:

{%highlight groovy%}
setDefaultTarget 'aTarget'
{%endhighlight%}

This allows great flexibility, possibly needless :-), in coding up defaults. For example:

{%highlight groovy%}
defaultTargetName = 'aTarget'
setDefaultTarget defaultTargetName
{%endhighlight%}
