Introduction
============

SonarQube is a Java application which performs analysis on source code,
to try and identify security issues, logical flaws, bad practices
and to detect deviation from local coding standards. 

It supports many languages/script, including C, C#, python, PHP, XML, CSS etc
via extensions which can be installed via the Admin Web site.
The basic version with standard plugins seems to be free, with more advanced
functionality being licensed and thus costing money.

SonarQube Server
----------------

SonarQube is available from https://www.sonarqube.org/

Being a Java application, it requires a decent machine (2GB RAM) and
scanning apparently can eat quite a lot of IOPS.
All results are stored in a database - postgreSQL is supported.

Clients
------

There are a number of clients which can be used to interact with SonarQube.
Typically, they are added to the 'make' tools, running before and after
the compilers which are used to build executable files from the source code.

### Sonarqube Scanner MSBuild ###

This is a Windows application (EXE file) which was developed in conjunction
with Microsoft.  Running it on a Windows platform is fairly straightforward.

If building a Dot.Net Core application on Linux, then the tool needs to be
invoked via mono (https://www.mono-project.com/)

Bamboo
------

Since we build code with Atlassian Bamboo, we want to enmesh Sonar with the
standard build tools.  There is a plugin for bamboo which will add a number
of Sonar tasks (https://marketplace.atlassian.com/plugins/ch.mibex.bamboo.sonar4bamboo/)
This has some complications, since typically, you will not want to run
Sonar on every single commit/build, and Bamboo doesn't really support
conditional branching in build tasks.


End of Document.
