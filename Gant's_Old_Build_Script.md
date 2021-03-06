---
layout: master
title: Gant -- Gant's old Gant build script
---

# Gant's Old Gant Build Script

As an example of a working Gant script this is the Gant script that was Gant's own Gant script as at
2009-08-19T11:12:30+01:00. This assumes Gant 1.7.0 or greater. If you find any errors or inefficiencies, do
say, either on the Gant developer mailing list or directly to me
[russel@winder.org.uk](mailto:russel@winder.org.uk). Note the use of the past tense in the first paragraph:
Gant is no longer using Gant as its build tool, it has switched to using Gradle. This script is therefore no
longer being actively evolved.

{%highlight groovy%}
//  Gant -- A Groovy way of scripting Ant tasks.
//
//  Copyright ?? 2006-9 Russel Winder
//
//  Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in
//  compliance with the License. You may obtain a copy of the License at
//
//    http://www.apache.org/licenses/LICENSE-2.0
//
//  Unless required by applicable law or agreed to in writing, software distributed under the License is
//  distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
//  implied. See the License for the specific language governing permissions and limitations under the
//  License.
//
//  Author : Russel Winder <russel@winder.org.uk>

final zipExtension = '.zip'
final tarExtension = '.tar'
final tgzExtension = '.tgz'

property ( file : 'local.build.properties' )
property ( file : 'build.properties' )

carryOn = true
[ 'gantVersion' ,
  'groovy15Version' , 'groovy16Version' , 'groovy17Version' ,
  'groovyStandaloneVersion' ,
  'junitVersion' , 'commonsCliVersion' , 'ivyVersion' , 'antVersion' , 'mavenVersion' ,
  'groovyAntTaskTestVersionPropertyFileName' ,
  ].each { name ->
           try { binding.getVariable ( name ) }
           catch ( MissingPropertyException mpe ) {
             ant.project.log ( "${name} is not defined." )
             carryOn = false
           }
}
if ( ! carryOn ) {
  ant.project.log ( 'A required property is not set, so terminating.' )
  System.exit ( 1 )
}

gantPrefix = 'gant-' + gantVersion

try { usingCobertura = Boolean.valueOf ( usingCobertura ) }
catch ( MissingPropertyException mpe ) { usingCobertura = false }

try { ignoreTestFailures = Boolean.valueOf ( ignoreTestFailures ) }
catch ( MissingPropertyException mpe ) { ignoreTestFailures = false }

try { skipTests = Boolean.valueOf ( skipTests ) }
catch ( MissingPropertyException mpe ) { skipTests = false }

try { testCase }
catch ( MissingPropertyException mpe ) { testCase = null }

try { testSpecialDistributionVersion}
catch ( MissingPropertyException mpe ) { testSpecialDistributionVersion = false }

//  The groovy.home property is only set if Gant is started from the command line.  When started
//  programmatically, and in particular via the Gant Ant task, it will have value null.  If the groovy.home
//  property is set then either it is a copy of the GROOVY_HOME environment variable if that is set or it is
//  deduced from the location of the groovyc command.  The GROOVY_HOME environment variable may or may not
//  be set.  If it is set use the installed Groovy, otherwise use the Groovy specified in the properties
//  file.  This only matters for non-distribution builds.

groovyHome = System.properties.'groovy.home'
groovyHomeEnvironment = System.getenv ( ).GROOVY_HOME
if ( groovyHomeEnvironment ) { assert groovyHome == groovyHomeEnvironment }
else { assert groovyHomeEnvironment == null }

//  The Gant source is structured according to the standard Maven 2 hierarchy.

final sourceDirectory = 'src/main/groovy'
final testsDirectory = 'src/test/groovy'
final jarfilesDirectory = 'jarfiles'
final scriptsDirectory = 'scripts'

final buildDirectory = 'target_gant'
final buildClassesDirectory = buildDirectory + '/classes'
final buildTestClassesDirectory = buildDirectory + '/test-classes'
final buildTestReportsDirectory = buildDirectory + '/test-reports'
final buildTestReportsHTMLDirectory = buildDirectory + '/test-reports/html'
final buildCoberturaClassesDirectory = buildDirectory + '/cobertura-classes'
final buildCoberturaReportsDirectory = buildDirectory + '/cobertura-reports'

final buildFilteredScriptsDirectory = buildDirectory + '/filteredScripts'
final buildInstallDirectory = buildDirectory + '/install'

final documentationDirectory = buildDirectory + '/documentation'
final apiDocumentationDirectory = documentationDirectory + '/api'

final buildMetadataDirectory = buildClassesDirectory + '/META-INF'

//  This is the name used in in
//  gant.targets.tests.Maven_Tests.testCompileTargetInDirectoryOtherThanTheCurrentBuildDirectory.  It would
//  be good if these could be made dependent rather than independent as they are now.

final buildDirectoryForMavenTargetSet = 'target_forMavenTest'

final sourceCompatibility = '5'
final targetCompatibility = '5'

//  Labels for supporting use of the Maven Ant task.  NB Installing Gant installs the Maven and Ivy Ant
//  tasks jars into the Groovy library.

final mavenAntTaskJarName = 'maven-ant-tasks-' + mavenVersion + '.jar'

final antlibXMLns = 'antlib:org.apache.maven.artifact.ant'
final mavenPOMId = 'maven.pom'
final remoteRepositories = [
                            [ 'codehausSnapshot' , 'http://snapshots.repository.codehaus.org' ] ,
                            [ 'codehausMain' , 'http://repository.codehaus.org' ] ,
                            [ 'mavenCentral' , 'http://repo1.maven.org/maven2' ] ,
                            ]

//  List of files from the top-level directory to put into the distribution files.

final distributionTopLevelFiles = [
                             'build.gant' , 'buildMaven.gant' , 'build.properties' , 'build.xml' ,
                             '.classpath' , '.project' ,
                             'LICENCE.txt' ,
                             'README_Distribution.txt' , 'README_Install.txt' ,
                             'releaseNotes.txt'
                             ]

 //  List of directories to put in wholesale into the distribution files.

final distributionDirectories = [ 'documentation' , 'examples' , '.settings' , scriptsDirectory , jarfilesDirectory , sourceDirectory , testsDirectory ]

//  The versions of Gant in the Maven repository to build distribution version of Gant against.
//
//  Introducing the Gant Ant Task written in Java means we have to use the joint compiler to compile Gant
//  and this means we cannot build Gant against Groovy 1.0 since the joint compiler only arrived with 1.5.0.
//  Opinion on the user list indicated that it seemed reasonable to drop support for Groovy 1.0.
//
//  It is assumed that the groovyc fork mode works, which means we cannot sensibly build against any version
//  of Groovy prior to 1.5.2 -- since that is the version in which groovyc fork mode appeared.  Using a
//  forked Java task is not an option because there is no --classpath option in the 1.5.0 and 1.5.1 versions
//  of Groovyc and this is essential to get things to work correctly.  Gant can still be built and
//  distributed against Groovy 1.5.0 and 1.5.1 but it has to be done the hard (manual) way.
//
//  Allow a mechanism for testing individual distribution entries as well as producing all of them.

final distributionVersions = [ : ]
 if ( testSpecialDistributionVersion ) {
   //def theGroovyVersion = groovy15Version
   def theGroovyVersion = groovy16Version
   //def theGroovyVersion = groovy17Version
   distributionVersions.put ( 'groovy-' + theGroovyVersion , [ groupId : 'org.codehaus.groovy' , artifactId : 'groovy-all' , version : theGroovyVersion ] )
 }
 else {
   [ groovy15Version , groovy16Version , groovy17Version ].each { version ->
     distributionVersions.put ( 'groovy-' + version , [ groupId : 'org.codehaus.groovy' , artifactId : 'groovy-all' , version : version ] )
   }
   distributionVersions.put ( 'standalone' , [ groupId : 'org.codehaus.groovy' , artifactId : 'groovy-all' , version : groovyStandaloneVersion ] )
}

//  As J??rgen Hermann pointed out, the trailing / on the URL is very important.

final distributionURL = 'https://dav.codehaus.org/dist/gant/'

//  The hierarchy on the Codehaus server for the distributions is fixed and known.

final serverDistributionsDirectory = 'distributions'
final serverJarsDirectory = 'jars'

//  This is a map with key local source path and value server path.

final serverUploadProducts = [ : ]

//  This is the path to the Gant jar currently being processed.  It is set when the jar has been successfully created.

currentGantJarPath = null

//  --------------------------------------------------------------------------------------------------------
//
//  Various support Closures.
//
//  Currently these Closures do not have the GantMetaClass as their metaclass so there is no delegated
//  lookup on the ant object, it has to be done explicitly.
//
//  The various compilation and test actions are parameterized so they can be used with different
//  classpaths.
//
//  --------------------------------------------------------------------------------------------------------

final getDependency = { root , dependencies ->
  final pathId = root + 'PathId'
  final filesetId = root + 'FilesetId'
  binding.setVariable ( pathId , pathId )
  binding.setVariable ( filesetId , filesetId )
  "${antlibXMLns}:dependencies" ( pathId : pathId , filesetId : filesetId ) {
    if ( dependencies instanceof List ) { for ( item in dependencies ) { dependency ( item ) } }
    else { dependency ( dependencies ) }
    remoteRepositories.each { List item -> remoteRepository ( id : item[0] , url : item[1] ) }
  }
}

final performCompile = { classpathId ->
  mkdir ( dir : buildClassesDirectory )
  taskdef ( name : 'groovyc' , classname : 'org.codehaus.groovy.ant.Groovyc' , classpathref : classpathId )
  groovyc ( srcdir : sourceDirectory , destdir : buildClassesDirectory , fork : 'true' , failonerror : 'true' , includeantruntime : 'false' ) {
    classpath { path ( refid : classpathId ) }
    javac ( source : sourceCompatibility , target : targetCompatibility , debug : 'on' )
  }
}

final makeManifest = { ->
  mkdir ( dir : buildMetadataDirectory )
  copy ( todir : buildMetadataDirectory ,  file : 'LICENCE.txt' )
  manifest ( file : buildMetadataDirectory + '/MANIFEST.MF' ) {
    attribute ( name : 'Built-By' , value : System.properties.'user.name' )
    attribute ( name : 'Extension-Name' , value : 'gant' )
    attribute ( name : 'Specification-Title' , value : 'Gant: scripting Ant tasks with Groovy.' )
    attribute ( name : 'Specification-Version' , value : gantVersion )
    attribute ( name : 'Specification-Vendor' , value : 'The Codehaus' )
    attribute ( name : 'Implementation-Title' , value : 'Gant: Scripting Ant tasks with Groovy.' )
    attribute ( name : 'Implementation-Version' , value : gantVersion )
    attribute ( name : 'Implementation-Vendor' , value : 'The Codehaus' )
  }
}

final performPackage =  { jarPath ->
  currentGantJarPath = null
  makeManifest ( )
  try {
    def tempfile = File.createTempFile ( 'gant_compile' , '.jar' )
    jar ( destfile : tempfile.path , basedir : buildClassesDirectory , manifest : buildMetadataDirectory + '/MANIFEST.MF' )
    taskdef ( resource : 'aQute/bnd/ant/taskdef.properties' , classpath : 'lib/bnd-0.0.337.jar' )
    bndwrap ( output : jarPath , jars : tempfile.path )
    currentGantJarPath = jarPath
  }
  finally {
    tempfile.delete ( )
  }
}

final performCompileTests = { classpathId ->
  if ( ! skipTests ) {
    mkdir ( dir : buildTestClassesDirectory )
    taskdef ( name : 'groovyc' , classname : 'org.codehaus.groovy.ant.Groovyc' , classpathref : classpathId )
    groovyc ( srcdir : testsDirectory , destdir : buildTestClassesDirectory , fork : 'true' , failonerror : 'true' , includeantruntime : 'false' ) {
      classpath {
        pathelement ( location : currentGantJarPath )
        path ( refid : classpathId )
      }
      javac ( source : sourceCompatibility , target : targetCompatibility , debug : 'on' )
    }
  }
}

final performTests =  { classpathId ->
  if ( ! skipTests ) {
    mkdir ( dir : buildTestReportsDirectory )
    mkdir ( dir : buildTestReportsHTMLDirectory )
    if ( usingCobertura ) {
      mkdir ( dir : buildCoberturaClassesDirectory )
      mkdir ( dir : buildCoberturaReportsDirectory )
      getDependency ( 'coberturaJar' , [ groupId : 'net.sourceforge.cobertura' , artifactId : 'cobertura' , version : '1.9' ] )
      taskdef ( resource : 'tasks.properties' , classpathref : 'coberturaJarPathId' )
      'cobertura-instrument' ( todir : buildCoberturaClassesDirectory ) { fileset ( dir : buildClassesDirectory ) }
    }
    getDependency ( 'ivyJar' , [ groupId : 'org.apache.ivy' , artifactId : 'ivy' , version : ivyVersion ] )
    def testClasspathId = 'testClasspathId'
    path ( id : testClasspathId ) {
      if ( usingCobertura ) {
        // MUST appear in classpath before the uninstrumented classes.
        pathelement ( location : buildCoberturaClassesDirectory )
        path ( refid : 'coberturaPathId' )
      }
      pathelement ( location : buildTestClassesDirectory )
      path ( refid : 'ivyJarPathId' )
      fileset ( dir : jarfilesDirectory , includes : mavenAntTaskJarName )
      path ( refid : classpathId )
    }
    // Forkmode should be once for speed but perTest for safety.
    if ( ! ( testCase ==~ 'org.codehaus.gant.ant.tests.*_Test' ) ) {
      junit ( printsummary : 'yes' , failureproperty : 'testsFailed' , fork : 'true' , forkmode : 'once' , includeantruntime : 'false' ) {
        // jvmarg ( line : '-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005' )
        formatter ( type : 'plain' )
        formatter ( type : 'xml' ) // Must have XML output for the continuous integration builds.
        if ( testCase != null) { test ( name : testCase , todir : buildTestReportsDirectory ) }
        else { batchtest ( todir : buildTestReportsDirectory ) { fileset ( dir : buildTestClassesDirectory , includes : '**/*_Test.class' , excludes : '**/ant/tests/*' ) } }
        classpath {
          path ( refid : testClasspathId )
          pathelement ( location : currentGantJarPath )
        }
      }
    }
    if ( ( testCase == null ) || ( testCase ==~ 'org.codehaus.gant.ant.tests.*_Test' ) ) {
      junit ( printsummary : 'yes' , failureproperty : 'testsFailed' , fork : 'true' , forkmode : 'once' , includeantruntime : 'false' ) {
        formatter ( type : 'plain' )
        formatter ( type : 'xml' ) // Must have XML output for the continuous integration builds.
        batchtest ( todir : buildTestReportsDirectory ) { fileset ( dir : buildTestClassesDirectory , includes : '**/ant/tests/*_Test.class' ) }
        classpath { path ( refid : testClasspathId ) }
      }
    }
    junitreport ( todir : buildTestReportsDirectory ) {
      fileset ( dir : buildTestReportsDirectory , includes : 'TEST-*.xml' )
      report ( todir : buildTestReportsHTMLDirectory )
    }
    if ( usingCobertura ) { 'cobertura-report' ( srcdir : sourceDirectory , destdir : buildCoberturaReportsDirectory ) }
  }
  //  Do not use the mapping of Ant properties to binding values here since that generates a
  //  MissingPropertyException in the case that the property doesn't exist whereas accessing it through the
  //  fully qualified name generates a null return.
  ant.project.properties.testsFailed == null
}

final setInstallationDirectory = { path ->
  if ( path == null ) {
    ant.project.log ( 'Property installationDirectory is not set, cannot undertake (un)install actions.' )
    System.exit ( 1 )
  }
  installationDirectory = path
}

final prepareInstallDirectory = { path , gantJarName , pathIdRoots , isStandalone , item ->
  setInstallationDirectory ( path )
  def installationBinDirectory = installationDirectory + '/bin'
  def groovyJarName = null
  if ( item != null ) { groovyJarName = ( isStandalone ? 'groovy-all-' : 'groovy-' ) + item.version + '.jar' }
  else {
    pathconvert ( property : 'groovyJarName' ) {
      path ( refid : groovyJarPathId )
      mapper ( type : 'flatten' )
    }
  }
  copy ( todir : installationBinDirectory ) {
    fileset ( dir : scriptsDirectory + ( isStandalone ? '/bin_standalone' : '/bin_requiresGroovy' ) )
    fileset ( dir : scriptsDirectory + '/bin' )
    filterset { filter ( token : 'GANT_VERSION' , value : gantVersion ) }
    filterset { filter ( token : 'GROOVYJAR' , value : groovyJarName ) }
  }
  chmod ( perm : 'a+x' ) { fileset ( dir : installationBinDirectory , includes : 'gant*' ) }
  copy ( todir : installationDirectory ) { fileset ( dir : scriptsDirectory , includes : 'conf/gant-starter.conf' ) }
  getDependency ( 'ivyJar' , [ groupId : 'org.apache.ivy' , artifactId : 'ivy' , version : ivyVersion ] )
  copy ( todir : installationDirectory + '/lib' ) {
    fileset ( dir : buildDirectory , includes : gantJarName )
    fileset ( dir : jarfilesDirectory , includes : mavenAntTaskJarName )
    fileset ( refid : ivyJarFilesetId )
    if ( isStandalone ) {
      if ( pathIdRoots instanceof List ) { for ( root in pathIdRoots ) { fileset ( refid : root + 'FilesetId' ) } }
      else { fileset ( refid : pathIdRoots + 'FilesetId' ) }
    }
    mapper ( type : 'flatten' )
  }
}

final prepareZipFromTarAndRegisterForUpload = { String root , Closure closure ->
  //  Have to put this in the binding :-(
  rootPath = buildDirectory + System.properties.'file.separator' + root
  closure ( )
  zip ( destfile : rootPath + zipExtension ) { tarfileset ( src : rootPath + tarExtension ) }
  gzip ( src : rootPath + tarExtension , destfile : rootPath + tgzExtension )
  serverUploadProducts [ rootPath + tgzExtension ] = serverDistributionsDirectory + '/' +  root + tgzExtension
  serverUploadProducts [ rootPath + zipExtension ] = serverDistributionsDirectory + '/' +  root + zipExtension
}

final preparePom = { String version ->
  //  Return the reference to an edited copy of the POM template , replacing the version numbers and other
  //  tokens with the values for this JAR.
  def classifier = 'groovy' + version.split ( "[\\.-]" ) [ 0..1 ].join ( '.' )
  def targetPOM = classifier + '.pom'
  copy ( tofile : targetPOM , file : 'pom.xml.in' ) {
    filterset {
      filter ( token : 'artifactId' , value : 'gant_' + classifier )
      filter ( token : 'gantVersion' , value : gantVersion )
      filter ( token : 'groovyVersion' , value : version )
      filter ( token : 'commonsCliVersion' , value : commonsCliVersion )
      filter ( token : 'antVersion' , value : antVersion )
      filter ( token : 'mavenVersion' , value : mavenVersion )
      filter ( token : 'ivyVersion' , value : ivyVersion )
    }
  }
  "${antlibXMLns}:pom" ( file : targetPOM , id : targetPOM )
  return targetPOM
}

final doWithAllGroovyVersions = { Closure c ->
  distributionVersions.each { label , item ->
    def groovyAntTaskTestVersionPropertyFile = new File ( groovyAntTaskTestVersionPropertyFileName )
    groovyAntTaskTestVersionPropertyFile.write ( 'groovyAntTaskTestVersion = ' + item.version )
    def pathIdRoot = 'classpath' + item.version
    def pathId = pathIdRoot + 'PathId'
    getDependency ( pathIdRoot , [
                                  [ groupId : item.groupId , artifactId : item.artifactId , version : item.version ] ,
                                  [ groupId : 'org.apache.ant' , artifactId : 'ant' , version : antVersion  ] ,
                                  [ groupId : 'org.apache.ant' , artifactId : 'ant-junit' , version : antVersion ] ,
                                  [ groupId : 'commons-cli' , artifactId : 'commons-cli' , version : commonsCliVersion ] ,
                                  [ groupId : 'junit' , artifactId : 'junit' , version : junitVersion ]
                                  ] )
    //  Remove the compilation products of the last compilation so as not to affect this one!
    delete ( dir : buildClassesDirectory )
    delete ( dir : buildTestClassesDirectory )
    delete ( dir : buildFilteredScriptsDirectory )
    delete ( dir : buildInstallDirectory )
    performCompile ( pathId )
    def jarPath = buildDirectory + '/' + gantPrefix + '_groovy-' + item.version + '.jar'
    performPackage ( jarPath )
    performCompileTests ( pathId )
    if ( ! skipTests && ! performTests ( pathId ) ) { someTestsFailed = true }
    c.call ( label , item , jarPath , pathIdRoot )
    groovyAntTaskTestVersionPropertyFile.delete ( )
  }
}

final doDocumentation = { ->
  taskdef ( name : 'groovydoc' , classname : 'org.codehaus.groovy.ant.Groovydoc' )
  def javaApiDocumentationDirectory = apiDocumentationDirectory + '/java'
  def groovyApiDocumentationDirectory = apiDocumentationDirectory + '/groovy'
  def classpathIdRoot = 'javaDoc'
  def classpathId = classpathIdRoot + 'PathId'
  getDependency ( classpathIdRoot , [
                                     [ groupId : 'org.codehaus.groovy' , artifactId : 'groovy-all' , version : groovyStandaloneVersion ] ,
                                     [ groupId : 'org.apache.ant' , artifactId : 'ant' , version : antVersion  ] ,
                                     [ groupId : 'commons-cli' , artifactId : 'commons-cli' , version : commonsCliVersion ]
                                     ] )
  mkdir ( dir : javaApiDocumentationDirectory )
  mkdir ( dir : groovyApiDocumentationDirectory )
  def packageTitle = "Gant (${gantVersion})"
  def packageIncludeSpec = 'gant.*,org.codehaus.gant.*'
  //  Don't assume that &copy; and &ndash; are understood by the browser -- even though they are in the HTML
  //  4.0 specification.  Copyright symbol is &copy; or &#169;.  N-dash symbol is &ndash; or &#8211;
  def copyrightString = 'Copyright &#169; 2006&#8211;9 The Codehaus.  All rights reserved.'
  javadoc (
               sourcepath : sourceDirectory ,
               destdir : javaApiDocumentationDirectory ,
               packagenames : packageIncludeSpec,
               overview : 'overview.html' ,
               private : 'true' ,
               encoding : 'UTF-8' ,
               use : 'true' ,
               author : 'true' ,
               version : 'true' ,
               windowtitle : packageTitle ,
               doctitle : packageTitle ,
               header : packageTitle ,
               footer : copyrightString ,
               source : sourceCompatibility
               ) {
    classpath {
      pathelement ( location : currentGantJarPath )
      path ( refid : classpathId )
    }
  }
  groovydoc (
             sourcepath : sourceDirectory ,
             destdir : groovyApiDocumentationDirectory ,
             packagenames : packageIncludeSpec ,
             overview : 'overview.html' ,
             private : 'true' ,
             //encoding : 'UTF-8' ,
             use : 'true' ,
             //author : 'true' ,
             //version : 'true' ,
             windowtitle : packageTitle ,
             doctitle : packageTitle ,
             header : packageTitle ,
             footer : copyrightString ,
             //source : sourceCompatibility
             )
}

//  --------------------------------------------------------------------------------------------------------
//
//  The targets for doing a local build, test, install and uninstall.
//
//  install and uninstall require the setting of the property installDirectory -- usually done by creating a
//  file local.build.properties which is not in version control.
//
//  --------------------------------------------------------------------------------------------------------

includeTargets << gant.targets.Clean
cleanPattern <<  [ '**/*~' , 'cobertura.ser' , 'texput.log' ]
cleanDirectory << [ buildDirectory , buildDirectoryForMavenTargetSet ]

includeTool << gant.tools.Execute

target ( reallyClean : 'Ensure all clean operations for the entire tree get actioned.' ) {
  clobber ( )
  execute.shell ( "cd packaging/debian && gant clobber" )
}

target ( initializeNonDistribution : '' ) {
  def groovyJarPathIdRoot = 'groovyJar'
  groovyJarPathId = groovyJarPathIdRoot + 'PathId'
  compileJarSetPathId = 'compileJarSetPathId'
  testJarSetPathId = 'testJarSetPathId'
  //  If GROOVY_HOME is set then create a build of Gant dependent on that Groovy, otherwise pull down all
  //  the required bits and pieces to make a standalone build.  NB System.properties.'groovy.home' is not a
  //  good discriminator!
  if ( groovyHomeEnvironment ) {
    ant.project.log ( "Building with the Groovy installation at GROOVY_HOME=${groovyHomeEnvironment}." )
    //----------------------------------------------------------------------------------------------
    //  TODO : Need to find out which version is actually being used -- this is currently a bug.
    //----------------------------------------------------------------------------------------------
    groovyVersion = groovy17Version
    path ( id : groovyJarPathId ) { fileset ( dir : groovyHome + '/lib' , includes : 'groovy-*.jar' ) }
    path ( id : compileJarSetPathId ) {
      path ( refid : groovyJarPathId )
      fileset ( dir : groovyHome + '/lib' , includes : 'commons-cli*.jar' )
      //  The ASM and Antlr jars are just transitive dependencies of the Groovy jar, the Ant and Commons CLI
      //  jars are needed directly by the Gant code.
      fileset ( dir : groovyHome + '/lib' , includes : 'asm*.jar' )
      fileset ( dir : groovyHome + '/lib' , includes : 'ant*.jar' )  // Intentionally includes Ant and Antlr jars.
    }
    path ( id : testJarSetPathId ) {
      path ( refid : compileJarSetPathId )
      fileset ( dir : groovyHome + '/lib' , includes : 'junit*.jar' )
    }
  }
  else {
    groovyVersion = groovyStandaloneVersion
    ant.project.log ( "Building without a Groovy installation. Using Groovy ${groovyVersion}, Commons CLI ${commonsCliVersion}, Ant ${antVersion} from Maven repository." )
    getDependency ( groovyJarPathIdRoot , [ groupId : 'org.codehaus.groovy' , artifactId : 'groovy-all' , version : groovyVersion ] )
    def compileJarsNotGroovyPathIdRoot = 'compileJarsNotGroovy'
    compileJarsNotGroovyPathId = compileJarsNotGroovyPathIdRoot + 'PathId'
    getDependency ( compileJarsNotGroovyPathIdRoot , [
                                                      [ groupId : 'commons-cli' , artifactId : 'commons-cli' , version : commonsCliVersion ] ,
                                                      ] )
    path ( id : compileJarSetPathId ) {
      path ( refid : groovyJarPathId )
      path ( refid : compileJarsNotGroovyPathId )
    }
    def testAntJarSetPathIdRoot = 'testAntJarSet'
    testAntJarSetPathId = testAntJarSetPathIdRoot + 'PathId'
    getDependency ( testAntJarSetPathIdRoot , [
                                               [ groupId : 'org.apache.ant' , artifactId : 'ant-junit' , version : antVersion ] ,
                                               ] )
    path ( id : testJarSetPathId ) {
      path ( refid : compileJarSetPathId )
      path ( refid : testAntJarSetPathId )
    }
  }
}

target ( compile : 'Compile everything needed for a Groovy installation dependent build of Gant.' ) {
  depends ( initializeNonDistribution )
  performCompile ( compileJarSetPathId )
}

target ( 'package' : 'Create the jar file for a Groovy installation dependent build of Gant.' ) {
  depends ( compile )
  performPackage ( buildDirectory + "/gant-${gantVersion}.jar" )
}

target ( compileTests : 'Compile all the tests for a Groovy installation dependent build of Gant.' ) {
  depends ( 'package' )
  if ( ! skipTests ) { performCompileTests ( testJarSetPathId ) }
}

target ( test : 'Test a build of Gant.' ) {
  depends ( compileTests )
  if ( ! skipTests ) {
    if ( ! groovyHomeEnvironment ) {
      groovyAntTaskTestVersionPropertyFile = ( new File ( groovyAntTaskTestVersionPropertyFileName ) )
      groovyAntTaskTestVersionPropertyFile.write ( 'groovyAntTaskTestVersion = ' + groovyVersion )
    }
    def returnCode = performTests ( testJarSetPathId )
    ant.project.log ( 'Tests ' + ( returnCode ? 'suceeded' : 'failed' ) + '.' )
    if ( ! groovyHomeEnvironment ) { groovyAntTaskTestVersionPropertyFile.delete ( ) }
    return returnCode ? 0 : 1
  }
  return 0
}

target ( coberturaTest : 'Test a Groovy installation dependent build of Gant using Cobertura to get a coverage report.' ) {
  usingCobertura = true
  depends ( test )
}

target ( documentation : 'Create the API documentation of Gant.' ) {
  depends ( compile )
  doDocumentation ( )
}

target ( install : "Install a Groovy installation dependent build of Gant." ) {
  depends ( test )
  prepareInstallDirectory ( installDirectory , 'gant-' + gantVersion + '.jar' , ( groovyHomeEnvironment ? 'compileJarSet' : 'compileJars' ) , groovyHomeEnvironment == null , null )
}

target ( uninstall : "Uninstall Gant." ) {
  setInstallationDirectory ( installDirectory )
  delete ( dir : installationDirectory )
}

//  --------------------------------------------------------------------------------------------------------
//
//  The targets for handling the builds for the Maven repository.
//
//  --------------------------------------------------------------------------------------------------------

target ( mavenInstall : "Install Gant artifacts into local Maven cache." ) {
  doWithAllGroovyVersions { label , item , jarPath , pathIdRoot ->
    // Skip the standalone artifact since Maven artefacts always depend on the artifacts rather than packaging them.
    if ( label != 'standalone' ) {
      def targetPOM = preparePom ( item.version )
      "${antlibXMLns}:install" ( file : jarPath ) { pom ( refid : targetPOM ) }
      delete ( file : targetPOM )
    }
  }
}

target ( mavenDeploy : "Deploys the Gant artifacts to the appropriate Maven repository." ) {
  "${antlibXMLns}:install-provider" ( artifactId : 'wagon-webdav' , version : '1.0-beta-2' )
  doWithAllGroovyVersions { label , item , jarPath , pathIdRoot ->
    // Skip the standalone artifact since Maven artefacts always depend on the artifacts rather than packaging them.
    if ( label != 'standalone' ) {
      def targetPOM = preparePom ( item.version )
      "${antlibXMLns}:deploy" ( file : jarPath ) { pom ( refid : targetPOM ) }
      delete ( file : targetPOM )
    }
  }
}

//  --------------------------------------------------------------------------------------------------------
//
//  The targets for handling the distribution builds.
//
//  --------------------------------------------------------------------------------------------------------

target ( distribution : 'Create the distributions of Gant.' ) {
  def distributionClasspathId = 'classpath'
  clean ( )
  someTestsFailed = false
  doWithAllGroovyVersions { label , item , jarPath , pathIdRoot ->
    def isStandalone = label == 'standalone'
    def discriminator = isStandalone ? gantPrefix : gantPrefix + '_groovy-' + item.version
    def jarName = new File ( jarPath ).name
    serverUploadProducts [ jarPath ] = serverJarsDirectory + '/' + jarName
    prepareInstallDirectory ( buildInstallDirectory , jarName , pathIdRoot , isStandalone , item )
    prepareZipFromTarAndRegisterForUpload ( discriminator ) {
      //  NB Rely on Closures not being closures:  rootPath is only resolved when the Closure is executed.
      tar ( destfile :  rootPath + tarExtension ) {
        tarfileset ( dir : buildInstallDirectory , mode : '755' , prefix : gantPrefix ) { include ( name : 'bin/*' ) }
        tarfileset ( dir : buildInstallDirectory , prefix : gantPrefix ) {
          include ( name : '**' )
          exclude ( name : 'bin/*' )
        }
        tarfileset ( dir : '.' , includes : 'README*' , prefix : gantPrefix )
      }
    }
  }
  //  Create the source distribution.
  prepareZipFromTarAndRegisterForUpload ( 'gant_src-' + gantVersion ) {
    //  NB Rely on Closures not being closures:  rootPath is only resolved when the Closure is executed.
    tar ( destfile :  rootPath + tarExtension ) {
      tarfileset ( dir : '.' , includes : distributionTopLevelFiles.join ( ',' ) , prefix : gantPrefix )
      distributionDirectories.each { directory -> tarfileset ( dir : directory , prefix : gantPrefix + System.properties.'file.separator' + directory ) }
    }
  }
  // Create the API documentation distribution.
  doDocumentation ( )
  prepareZipFromTarAndRegisterForUpload (  'gant_doc-' + gantVersion ) {
    //  NB Rely on Closures not being closures:  rootPath is only resolved when the Closure is executed.
    tar ( destfile :  rootPath + tarExtension ) {
      tarfileset ( dir : documentationDirectory , prefix : gantPrefix + System.properties.'file.separator' + 'doc' )
    }
  }
  //  Return the result of the test executions so that uploadDistributions can choose not to upload a failed
  //  compilation.
  ! someTestsFailed
}

target ( uploadDistribution : 'Upload a complete distribution.' ) {
  if (  ( ! distribution ( ) ) && ( ! ignoreTestFailures ) ) {
    ant.project.log ( 'Not uploading as some tests failed.' )
    return
  }
  //  Slide has been retired by the Apache Foundation in favour of Jackrabbit, but we use Slide anyway.
  "${antlibXMLns}:dependencies" ( pathId :  'slidePathId' ) {
    //dependency ( groupId : 'org.apache.jackrabbit' , artifactId : 'jackrabbit-webdav' , version : '1.3.3' )
    dependency ( groupId : 'slide' , artifactId : 'slide-webdavlib' , version : '2.1' )
  }
  //  Is there a way of extracting the username and password information from the Maven Ant Task?  For the
  //  moment simply read the ~/.m2/settings.xml file.
  def loader = getClass ( ).getClassLoader ( )
  ( path ( refid : 'slidePathId' ).list ( ) as List ).each { location -> loader.addURL ( new URL ( 'file://' + location ) ) }
  def settings =  ( new XmlSlurper ( ) ).parse ( System.properties.'user.home' + '/.m2/settings.xml' ).servers.server.find { item -> item.id == 'codehaus.org' }
  def credentials =  loader.loadClass ( 'org.apache.commons.httpclient.UsernamePasswordCredentials' ).getConstructor ( String , String ).newInstance ( settings.username.toString ( ) , settings.password.toString ( ) )
  //  Have to put resource in the binding so that the Closures can access it.
  resource = loader.loadClass ( 'org.apache.webdav.lib.WebdavResource' ).getConstructor ( String ,  loader.loadClass ( 'org.apache.commons.httpclient.Credentials' ) , boolean ).newInstance ( distributionURL , credentials , true )
  if ( resource.statusCode != 200 ) { ant.project.log ( 'Failed to open ' + distributionURL ) }
  else {
    serverUploadProducts.each { source , destination ->
      def serverPath = resource.path + '/' + destination
      ant.project.log ( 'Uploading ' + source + ' -> ' + destination + ' : ' )
      def result = 'Failed.'
      if ( resource.putMethod ( serverPath , new File ( source ) ) ) { result = 'OK.' }
      ant.project.log ( result + ' Status : ' + resource.statusMessage )
    }
  }
  if ( resource != null ) { resource.close ( ) }
}

//  --------------------------------------------------------------------------------------------------------
//
//  A target needed in support of building a deb. See packaging/debian/build.gant.
//
//  --------------------------------------------------------------------------------------------------------

target ( debDistribution : '' ) {
  distributionVersions = [
                          ( 'groovy-' + groovyDebianVersion ) :  [ groupId : 'org.codehaus.groovy' , artifactId : 'groovy-all' , version : groovyDebianVersion ] ,
                          standalone : [ groupId : 'org.codehaus.groovy' , artifactId : 'groovy-all' , version : groovyStandaloneVersion ]
                          ]
  distribution ( )
}

//  --------------------------------------------------------------------------------------------------------
//
//  Set the default target.
//
//  --------------------------------------------------------------------------------------------------------

setDefaultTarget ( test )
{%endhighlight%}
