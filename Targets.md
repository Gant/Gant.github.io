---
layout: master
title:
---

# Targets

A Gant target is a binding of a closure to a name:

**target(name:** _target-name_**)** _target-closure_

the parameter of the call of the **target** function is a map that must have
the key **name**, and may have other keys. Perhaps the most important other
key is **description**:

**target(name:** _target-name_ **, description:** _target-description_**)** _target-closure_

Giving a target a name and a description is such a common form that there is a
short-cut for this:

**target(**_target-name_**:** _target-description_**)** _target-closure_

So for example:

{%highlight groovy%}
target(name: 'flob', description: 'Some message or other.') {
  print('Some message.')
}
{%endhighlight%}

can be written:

{%highlight groovy%}
target(flob: 'Some message or other.') {
  print('Some message.')
}
{%endhighlight%}

In this form, any legal Groovy string can be used as the _target-name_ but if
it a Groovy keyword then it must be explicitly a string (automatic
stringification doesn't work for Groovy keywords).

The _target-description_ string is used as the string to output when executing
"gant -p" (or "gant -T") to discover the list of allowed command-line targets.
If _target-description_ is missing or is the empty string then the target will
not appear in the target list - using the short-cut form of target, the
descritpion string must always be present but it can be the empty string. Only
targets with non-empty descriptions are treated as callable targets.

The _target-closure_ can contain any legal Groovy code. An AntBuilder is pre-
defined and is called **ant**, so calls can be made to any Ant task:

{%highlight groovy%}
target(flob: 'Some message or other.') {
  ant.echo message: 'Some message.'
}
{%endhighlight%}

Calling Ant tasks in a Gant closure is so common that Gant automatically tries
the **ant** object during function lookup. Hence the above can be written:

{%highlight groovy%}
target(flob: 'Some message or other.') {
  echo message: 'Some message.'
}
{%endhighlight%}

The usual rules associated with working with an AntBuilder in a Groovy script
apply.

As each target labels a closure, targets can be called from other targets:

{%highlight groovy%}
target(adob: 'A target called adob.') {
  flob()
}
{%endhighlight%}

Target **flob** is called as part of target **adob**. This allows a lot of
dependency information to be expressed. However, the calls are always made,
there is no implicit checking whether a target has already been executed.
Instead there is a method **depends** that can be used to handle the situation
where a target should only be called if it has not been executed already in
this run.

{%highlight groovy%}
target(adob: 'A target called adob.') {
  depends flob
}
{%endhighlight%}

The **depends** method is an executable method so it can be called at any time
in a target closure. This allows for a much more flexible expression of
dependency than specifying dependencies as a property of the target (as is
done in Ant for example).

In the above examples, the target name was always a string literal - Groovy
'stringyfies' the key of a map by default and that is what is happening here.
Often, particularly when synthesizing targets programmatically (the huge
benefit of Gant over Ant!), we want the name of the target to be constructed.
No problem here, we can do things like:

{%highlight groovy%}
aTargetName = 'something'
target((aTargetName): 'A target called' + aTargetName + '.' ) {
  println 'Executing ' + aTargetName
}
{%endhighlight%}

Putting parentheses around the bit before the colon means that Groovy
evaluates the expression and uses the result as the target name.

You might contemplate using GStrings in target names, and indeed descriptions,
for example:

{%highlight groovy%}
target("X${something}": '')
{%endhighlight%}

However, this is, in general, **_not_** a good thing to do. The reason is that
GStrings are evaluated only on actual use and this means that if variable
bindings have changed, the target name you thought you had is not the target
name you actually have. There can be too much dynamism some times. So it is
always wise to ensure that target names are fully evaluated at the point of
target definition. There is no block to target synthesis, it just means don't
use GStrings as target names.
