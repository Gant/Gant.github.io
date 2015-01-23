---
layout: master
title: Gant -- Pre-package Distributions
---

# Prepackaged Distributions

## Source and General Precompiled Distribution

The latest Gant distribution downloads are available:

---|---|---
1.9.11 Source | [Tarball](http://dist.codehaus.org/gant/distributions/gant_src-1.9.11.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant_src-1.9.11.zip)
1.9.11 Binary, compiled for use with Groovy 2.0.8 installation |[Tarball](http://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.0.8.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.0.8.zip)
1.9.11 Binary, compiled for use with Groovy 2.1.9 installation| [Tarball](http ://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.1.9.tgz)| [Zipfile](http://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.1.9.zip)
1.9.11 Binary, compiled for use with Groovy 2.2.2 installation| [Tarball](http://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.2.2.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.2.2.zip)
1.9.11 Binary, compiled for use with Groovy 2.3.0 installation | [Tarball](http://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.3.0.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant-1.9.11-_groovy-2.3.0.zip)
1.9.11 Binary, standalone installation | [Tarball](http://dist.codehaus.org/gant/distributions/gant-1.9.10.tgz) | [Zipfile](http://dist.codehaus.org/gant/distributions/gant-1.9.10.zip)

Installation of any of the binary distributions is simply a matter of extracting the distribution to the
desired location. A directory gant-1.9.11 will be created.

The jars have been uploaded to the Codehaus Maven repository (which then gets synced to the central Maven
repository), the groupId is org.codehaus.gant, the artifactId is one of gant_groovy2.0, gant_groovy2.1,
gant_groovy2.2, gant_groovy2.3 (indicating which Groovy version the artefact was compiled against), and the
version is 1.9.11.

Earlier distributions are available at
<http://dist.codehaus.org/gant/distributions>.

Snapshots of latest development versions, if any are available, are held on the
[Codehaus distribution snapshots repository](http://snapshots.dist.codehaus.org/gant/distributions) with
appropriate artefacts in the
[Codehaus artefacts snapshots repository](http://snapshots.repository.codehaus.org/).

## Mac OS X using MacPorts

**Mac OS X** users can get Gant, and Groovy, using MacPorts. As at 2015-01-23T10:18 this has Gant 1.9.5 and Groovy 2.0.6.


## Debian and Ubuntu Packages

**Debian** and **Ubuntu** users can install Gant using the usual package management system (apt-get and/or dpkg):

---|---|---
Squeeze (oldstable) | 1.9.1 | <http://packages.debian.org/squeeze/gant>
Wheezy (stable) | 1.9.7 | <http://packages.debian.org/wheezy/gant>
Jessie (testing)| 1.9.9| <http://packages.debian.org/jessie/gant>
Sid (unstable) | 1.9.9 | <http://packages.debian.org/sid/gant>

---|---|---
Lucid | 1.8.1 | <http://packages.ubuntu/com/lucid/gant>
Precise | 1.9.7 | <http://packages.ubuntu/com/precise/gant>
Trusty | 1.9.9 | <http://packages.ubuntu.com/quantal/gant>
Utopic | 1.9.9 | <http://packages.ubuntu.com/raring/gant>
Vivid | 1.9.9 | <http://packages.ubuntu.com/saucy/gant>

as at 2015-01-23T10:18. Gant always depends on the Groovy package, but this will get installed
automatically if not already installed.

If you want to stay more up to date but don't want to install from source, there is the following deb file
which creates a stand-alone installation of Gant with all the dependencies it needs, including Groovy 2.1.0:

---|---
Gant 1.9.11 standalone | <http://dist.codehaus.org/gant/debs/gant-standalone_1.9.11-1_all.deb>

## Fedora Packages

The page <https://admin.fedoraproject.org/pkgdb/package/gant/> details all the Gant packages for the
various Fedora versions. Sadly it seems they have retired the packaged for not being able to be
built. No-one from Fedora porject contacted anyone in the Groovy community about this :-(
