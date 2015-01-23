---
layout: master
title: Gant -- Maven Target Set
---

# Maven Target Set

Maven brought the idea of standard project directory tree structures and standardized project lifecycles to
the world of Java project builds. With Ant you have to specify everything about the build, with Maven
everything is standardized. Ant has the benefit of being completely general, Maven has the benefit of
convention. Gant sits with Ant, it is a gneral mechanism for scripting Ant tasks using Groovy rather than
XML. Using a dynamic programming language means that it is possible to write complete target sets that can
be included. So we don't have to use a completely different framework to harness the power of
convention. Examples of prepackaged target sets are gant.targets.Clean and gant.targets.Maven. These are, in
effect, mixins that can be used to create build specifications.

An example:

{%highlight groovy%}
includeTargets << gant.targets.Maven

maven.groupId = 'org.codehaus.gant'
maven.artifactId = 'gant'
maven.version = '1.2.1-SNAPSHOT'
maven.javaCompileProperties = [source: '1.5', target: '1.5', debug: 'true']
maven.deployURL = 'https://dav.codehaus.org/repository/gant'
maven.deploySnapshotURL = 'https://dav.codehaus.org/snapshots.repository/gant'

setDefaultTarget install
{%endhighlight%}

The Maven target set imports the targets: initialize, compile, test-compile,
test, package, install, and deploy. We can add any other targets just as we
might in any Gant file.

An alternative way of writing this is:

{%highlight groovy%}
includeTargets ** gant.targets.Maven * [
    groupId: 'org.codehaus.gant',
    artifactId: 'gant',
    version: '1.2.1-SNAPSHOT',
    javaCompileProperties: [source: '1.5', target: '1.5', debug: 'true'],
    deployURL: 'https://dav.codehaus.org/repository/gant',
    deploySnapshotURL: 'https://dav.codehaus.org/snapshots.repository/gant'
    ]

setDefaultTarget install
{%endhighlight%}

With either of the above, we can see what targets are available using the -p
(or -T) option:

{%highlight groovy%}
|> gant -p

 clean         Action the cleaning.
 clobber       Action the clobbering.  Do the cleaning first.
 compile       Compile the source code in src/main to target/classes.
 deploy        Deploy the artefact: copy the artefact to the remote repository https://dav.codehaus.org/snapshots.repository/gant.
 initialize    Ensure all the dependencies can be met and set classpaths accordingly.
 install       Install the artefact into the local repository.
 package       Package the artefact as a jar in target/classes.
 site          Create the website.
 test          Run the tests using the junit unit testing framework.
 test-compile  Compile the test source code in src/test to target/test-classes.

Default target is install.
{%endhighlight%}

NB The site target does not work as yet. Note that the comments explain as
much as possible about the action inclusing sources and targets where known.
The above example is for building Gant itself -- the 1.0.1-SNAPSHOT version.

For those people who prefer to use Test'N'Groove rather than JUnit for unit
tests, simply specify the testFramework property to be 'testng'. So for
example, the ADS Package build file is:

{%highlight groovy%}
includeTargets ** gant.targets.Maven * [
    groupId: 'org.devjavasoft',
    artifactId: 'ads',
    version: '3.1.0',
    javaCompileProperties: [source: '1.6', target: '1.6', debug: 'true'],
    testFramework: 'testng'
]

cleanPattern << '**/*~'

setDefaultTarget test
{%endhighlight%}

NB The Maven target set automatically pulls in the Clean target so we can use
the standard Gant clean support features. Checking the available targets for
the above Gant file:

{%highlight groovy%}
|> gant -p

 clean         Action the cleaning.
 clobber       Action the clobbering.  Do the cleaning first.
 compile       Compile the source code in src/main to target/classes.
 deploy        Deploy the artefact: copy the artefact to the remote repository .
 initialize    Ensure all the dependencies can be met and set classpaths accordingly.
 install       Install the artefact into the local repository.
 package       Package the artefact as a jar in target/classes.
 site          Create the website.
 test          Run the tests using the testng unit testing framework.
 test-compile  Compile the test source code in src/test to target/test-classes.

Default target is test.
{%endhighlight%}

Note that the test target tells us which unit test framework is being used.
Groovy's GString in action.

What about dependencies? Simply add to the compileDependencies or
testDependencies property, for example:

{%highlight groovy%}
maven.testDependencies << [groupId: 'org.testng', artifactId: 'testng', version: '5.8', scope: 'test', classifier: 'jdk15']
{%endhighlight%}

A dependency is a hash with keys for all the attributes required by the Maven
dependency resolution system. During completion of the initialize target all
specified dependencies will be checked for presence in the standard Maven
local repository or downloaded if not present. Although we have presented the
example of downloading TestNG here, you don't need it for using Test'N'Groove
since this dependency is automatically added when testFramework is set to
'testng'.

Currently the list of amendable properties is:

{%highlight groovy%}
groupId: ''
artifactId: ''
version: ''
sourcePath: 'src'
mainSourcePath: '${properties.sourcePath}${System.properties.'file.separator'}main'
testSourcePath: '${properties.sourcePath}${System.properties.'file.separator'}test'
targetPath: 'target' ,
mainCompilePath: '${properties.targetPath}${System.properties.'file.separator'}classes'
testCompilePath: '${properties.targetPath}${System.properties.'file.separator'}test-classes'
testReportPath: '${properties.targetPath}${System.properties.'file.separator'}test-reports'
metadataPath: '${properties.mainCompilePath}${System.properties.'file.separator'}META-INF'
javaCompileProperties: [source: '1.3', target: '1.3', debug: 'false']
groovyCompileProperties: [:]
compileClasspath: []
testClasspath: []
compileDependencies: []
testDependencies: []
testFramework: 'junit'
testFrameworkVersion: '3.8.2'
testFrameworkClassifier: 'jdk15'
packaging: 'jar'
deployURL: ''
deploySnapshotURL: ''
deployId: 'dav.codehaus.org' ,
manifest: [:] ,
manifestIncludes:  [] ,
{%endhighlight%}

Options for testFramework are currently 'junit' and 'testng'. Options for
packaging are currently 'jar' and 'war' (but the war packaging has not been
tested).

_NB The Maven target set is a work in progress. Currently it seems to work for
simple Maven structured projects, but it does not yet have all the plugin
control facilities of Maven itself. If you find anything that is missing or
doesn't work, please create a JIRA issue._
