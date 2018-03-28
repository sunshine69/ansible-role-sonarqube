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

There are many scanner suitable for many types. See
https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner for
more information.

Currently this ansible role supports the followings components. More can be
added into the role as role subcomponent.

* SonarQube Scanner for MSBuild

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

### Role Variables

* `sonar_host_url` - Required - Default: https://sonar.xvt.technology

   The sonarqube server that this agent scanner will talk to


* `sonar_scanner_msbuild_version` - Optional - Default:  4.1.1.1164

  The scanner version number.


* `sonar_scanner_package_url` - Required - Default:
   https://s3-ap-southeast-2.amazonaws.com/xvt-software/windows/sonar/sonar-scanner-msbuild-4.1.1.1164-net46.zip

   The url where you can download the scanner zip archive file to deploy.


* `dependency_packages` - Optional - Default: list of package 'mono-runtime' which required by the scanner.


* `sonar_login_token` - Required - Default: "ANSIBLE-ERROR. Variable sonar_login_token is NotSet"

   This variable will be used in templating the scanner property files to allow
   the clien to login into the sonar server. To generate one login to the sonar
   server, click your username icon next to the search box on the top right
   screen then select 'My Account'. Then click 'Security' and use the 'Generate
   New Token' section to generate new one. This variable needs to be in vault
   format in your inventory as it is of credential type.


* `sonar_scanner_install_dir` - Optional - Default: /opt/sonar-scanner-msbuild

   This will be the sonar scanner home directory. The bamboo agent will have
   capabilities (see examples) to point to this location and the bamboo
   server sonar built-in plugin will find these executable.

   Examples of bamboo agent cap:

   ```
     bamboo_agent_extra_capabilities: |
     # This is for the sonar HOME
     system.builder.sos.Sos=/opt/sonar-scanner-msbuild
     # This is for project dependencies checking
     SonarQubeScannerMSBuild=/opt/sonar-scanner-msbuild
     # For building the path to the executable
     system.builder.msbuild.SonarQubeScannerMSBuild=/opt/sonar-scanner-msbuild/linux

   ```

   Currently sonar scanner does not support the ability to bypass branches
   in the source code it scans thus we use the scanner command manually in
   bamboo script to skip branches. These caps only used if it exists rather
   than the value (path to find out executable)


* `sonar_scanner_extraction_dir` - Optional - Default: "{{ sonar_scanner_install_dir }}-{{ sonar_scanner_msbuild_version }}"

   This is the directory we unzip the archive. There will be a symlink
   `sonar_scanner_install_dir` => `sonar_scanner_extraction_dir`


* `sonar_scanner_property_file_name` - Optional - Default: sonar-scanner.properties

   We will use ansible find module to find out where the file name is and
   then template it using our template to configure the scanner.


End of Document.
