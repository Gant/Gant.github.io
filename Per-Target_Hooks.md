---
layout: master
title: Gant -- Per-target Hooks
---

# Per-Target Hooks

From version 1.8.0 onwards, Gant has per-target hooks that can be programmed.  Hooks are just closures that
get executed. There are entry hooks and exit hooks for each target.  There are actually two different types
of entry hook and two different types of exit hook.  There are hooks that are defined by the target itself
which are for that target only and there are global hooks that are defined outside any particular target and
are the same for all targets.

By default the global hooks are empty lists, and each target has an entry and an exit list which contain a
closure for putting out a log entry.  The enter target log entry and the exit target log entry are handled
by closures in the per-target local entry and exit hooks.  (This means that a target can change its log
behaviour individually.)

The global hooks are just entries in the binding:

    globalPreHook
    globalPostHook

The variables are mutable ones with no contraints on assignment (so you can do anything with them).  The
intention is that the refer to a closure or list of closures.  If they refer to anything else then
exceptions will be thrown.

The local per-target hooks are expected to be set in the parameter list to the target.  The keys prehook and
posthook refer to the respective hooks.  Setting them overwrites the default -- which is a list of closures.
So, for example:

    target(name: 'blah', description: 'An example', prehook: [], posthook: []) {
    . . .
    }

This example replaces the default lists with empty ones.  There are no constraints on what can and cannot be
used as the values to associate with the keys, but if the item is not a closure or list of closures,
exceptions will result.

Most of the time people will want to add to the default list rather than replace the default list.  For this
there are the addprehook and addposthook keys:

    target(name: 'blah', description: 'An example', addprehook: [], addposthook: []) {
    . . .
    }

The values for the keys can be closures or lists of closures, and they will be appended to or concatenated
to respectively to the existing list of closures.  If the item is not a closure or list of closures,
exceptions will result.

Where the item is a list of closures, the closures will be executed in sequence order.

If it wasn't already obvious, the prehooks are executed before the target body, and then the posthooks are
executed.  Global prehooks are executed before the local prehooks, and global posthooks are executed after
the local posthooks.
