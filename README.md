Introduction
============

SonarQube is a Java application which performs analysis on source code,
to try and identify security issues, logical flaws, bad practices
and to detect deviation from local coding standards.

It supports many languages/script, including C, C#, python, PHP, XML, CSS etc
via extensions which can be installed via the Admin Web site.
The basic version with standard plugins seems to be free, with more advanced
functionality being licensed and thus costing money.

Bamboo
======

Since we build code with Atlassian Bamboo, we want to enmesh Sonar with the
standard build tools.  There is a plugin for bamboo which will add a number
of Sonar tasks (https://marketplace.atlassian.com/plugins/ch.mibex.bamboo.sonar4bamboo/)
This has some complications, since typically, you will not want to run
Sonar on every single commit/build, and Bamboo doesn't really support
conditional branching in build tasks.


SonarQube Server
================

SonarQube is available from https://www.sonarqube.org/

Being a Java application, it requires a decent machine (2GB RAM) and
scanning apparently can eat quite a lot of IOPS.
All results are stored in a database - postgreSQL is supported.

Sonar Clients
=============

There are a number of clients which can be used to interact with SonarQube.
Typically, they are added to the 'make' tools, running before and after
the compilers which are used to build executable files from the source code.

Sonarqube Scanner MSBuild
-------------------------

Sonarqube Scanner MSBuild is a Windows application (EXE file) which was
developed in conjunction with Microsoft.

There are a couple of config files - it seems that the best way is to
pre-configure them with the location of the SonarQube server and possibly
also the account which will be used to authenicate with the server.

Running it on a Windows platform seems to be fairly straightforward.

If building on Linux (eg. for a Dot.Net Core application), then since the
EXE file is not directly executable, the scanner will need to be invoked
via mono (https://www.mono-project.com/).

### Bamboo ###

Note that the Bamboo tasks hardcode the final directory name ('bin')
and the name of the EXE file itself, so on Linux, a shim script will need
to be created which matches this.

The bamboo Remote agent should be configured with two capabilities:
    * SonarQubeScannerMSBuild - which is  normal key-value capability.
      It specifies the directory where the client has been unpacked/configured.
      Suggested location:  /opt/sonar-scanner-msbuild
    * system.builder.msbuild.SonarQubeScannerMSBuild - an Executable capabiity of type MSBuild.
      It also needs to point to the client dir, and contain a file "bin/MSBuild.SonarQube.Runner.exe"
      which is the hardcoded name that the plugin will append.
      Suggested location:  /opt/sonar-scanner-msbuild/linux

Ansible
=======

This role contains the following tasks:
* install the Sonarqube Scanner MSBuild client.
  This requires a variable 'sonarqube.logintoken', which should come from a vault or something.


End of Document.
