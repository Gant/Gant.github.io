---
layout: master
title: Gant -- Pre-package Distributions
---

# Prepackaged Distributions

## Source and General Precompiled Distribution

_Due to a management oversight, the 1.9.11 distribution files were not retrieved from the Codehaus
repository before the repository went away. For various reasons, it is not possible to recreate the
files. A new release 1.10.0 will shortly be made._

_NB 1.10.0 is in fact a bug release over 1.9.11, but because of the change of build, supported Groovy
versions, and base Java version, a new minor version is released._

<!--
The latest Gant distribution downloads are available:

---|---|---
1.10.0 Source | [Tarball](http://dist.codehaus.org/gant/distributions/gant_src-1.10.0.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant_src-1.10.0.zip)
1.10.0 Binary, compiled for use with Groovy 2.4.8 installation | [Tarball](http://dist.codehaus.org/gant/distributions/gant-1.10.0-_groovy-2.3.0.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant-1.10.0-_groovy-2.3.0.zip)
1.10.0 Binary, standalone installation | [Tarball](http://dist.codehaus.org/gant/distributions/gant-1.10.0.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant-1.10.0.zip)

Installation of any of the binary distributions is simply a matter of extracting the distribution to the
desired location. A directory gant-1.10.0 will be created.
-->

All the artefacts for Gant 1.9.11 and many earlier versions remain on Maven Central. The groupId is
org.codehaus.gant, the artifactId is one of gant\_groovy2.0, gant\_groovy2.1, gant\_groovy2.2, gant\_groovy2.3
(indicating which Groovy version the artefact was compiled against), and the version is 1.9.11.

The artefacts for Groovy 2.3.x should work with Groovy 2.4.x, pending release of Gant 1.10.0 which will be
compiled for Groovy 2.4.8 but will work with any 2.4.x, and 2.5.x versions of Groovy.

## Mac OS X using MacPorts

**Mac OS X** users can get Gant, and Groovy, using MacPorts. As at 2017-02-12T11:59+00:00 this has Gant 1.9.10 and Groovy 2.4.8.


## Debian and Ubuntu Packages

**Debian** and **Ubuntu** users can install Gant using the usual package management system (apt-get and/or dpkg):

---|---|---
Wheezy (oldstable) | 1.9.7 | <http://packages.debian.org/wheezy/gant>
Jessie (stable)| 1.9.9 | <http://packages.debian.org/jessie/gant>
Stretch (testing)| 1.9.11 | <http://packages.debian.org/stretch/gant>
Sid (unstable) | 1.9.11 | <http://packages.debian.org/sid/gant>

---|---|---
Precise | 1.9.7 | <http://packages.ubuntu/com/precise/gant>
Trusty | 1.9.9 | <http://packages.ubuntu.com/trusty/gant>
Xenial | 1.9.11 | <http://packages.ubuntu/com/xenial/gant>
Yakkety | 1.9.11 | <http://packages.ubuntu.com/yakkety/gant>
Zesty | 1.9.11 | <http://packages.ubuntu.com/zesty/gant>

as at 2017-02-12T11:59+00:00. Gant always depends on the Groovy package, but this will get installed
automatically if not already installed.

## Fedora Packages

The page <https://admin.fedoraproject.org/pkgdb/package/gant/> details all the Gant packages for the
various Fedora versions. Sadly it seems they have retired the packaged for not being able to be
built. No-one from Fedora project contacted anyone in the Groovy community about this :-(
