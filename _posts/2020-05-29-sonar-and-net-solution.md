---
title: Sonar and .net solution - How to ensure your code
author: Chaves
date: 2020-05-29 22:00:00 +0000
categories: [Blogging, Tools, .NET]
tags: [.net, charp, msbuild, sonar, better code, mvc, code coverage, code quality, best pratices, visual studio, bat file, tfs]
---
![good code](/assets/img/posts/good_code.png)

Sonar Qube is a well-known code quality, security, and smell checker engine for custom development. On this post I'll leave some tips to set up for an asp.net project with the following requirements:

1. Code analysis;
1. Test analysis;
1. Code coverage;

There are some plugins that you can incorporate with our Visual Studio or Visual Studio Code, but here I'll cover only the most "manual" solution because we were integrating with a local CD/CI system. - also, there's integration with Azure DevOps.

First things first, you'll to install sonar or ask for a project set up on a remote server, and ask for your project key.

You can follow the instructions on sonar wiki page - they have great documentation - [https://docs.sonarqube.org/latest/setup/overview/](https://docs.sonarqube.org/latest/setup/overview/ )

We started to create a .bat file with the following instructions (you can find the final file at the end)

## First step
Create a folder to handle the test results and code coverage generated file, and the first step is to delete those files for each sonar run:

```bat
@echo SONAR-MyProject.services
 
del c:\@work\testresults\MyProject.services\ /F /Q /S
rmdir c:\@work\testresults\MyProject.services\ /Q /S
```

## Second step
Git pull to get the latest code version 😊 and the regular MSBuild clean (ensure that everything is reset) and the nugget restore command (remember that you must have the variables configured at you system environmental variables:

```bat
git pull
 
"C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\MSBuild.exe" /t:clean
 
nuget.exe restore
```
This project is using the 2017 version, but it's the same for more recent ones

## Third step
If you have installed Sonar on your machine, the default URL is the localhost:9000 where the webserver listens. So here we're starting the sonar MSBuild analyzer and passing some important and critical params, such as the unit test results and code coverage file (these ones are optional if you don't need.. but best practices are always preferred 😎 ), and the MSBuild rebuild command to initiate the building process.

```bat
SonarScanner.MSBuild.exe begin /k:"MyProject.services" /d:sonar.host.url="http://localhost:9000" /d:sonar.login="3a325d1f9fed13b744ac92e2cffc30acb59fd702" /d:sonar.cs.xunit.reportsPaths="C:\@Work\TestResults\MyProject.services\MyProject.services.unitresults.xml" /d:sonar.cs.vscoveragexml.reportsPaths="C:\@Work\TestResults\MyProject.services\MyProject.services.coverage.xml"
 
"C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\MSBuild\15.0\Bin\MSBuild.exe" /t:rebuild

```
## Forth step
After it is time to set up the steps for unit tests and code coverage. Here we're using the **XUnit framework**, but sonar supports other ones [https://docs.sonarqube.org/latest/analysis/coverage/](https://docs.sonarqube.org/latest/analysis/coverage/)

```bat
REM Steps to run tests and code coverage
 
c:\@Work\xunit\xunit.console.exe .\tst\MyProject.services.UnitTests\bin\Debug\MyProject.services.UnitTests.dll -xml C:\@Work\TestResults\MyProject.services\MyProject.services.unitresults.xml
```
## Fith step
Now that we have our unit tests results in an XML file, we're using the vstest.console to generate the code coverage file using a specific runsettings (check at the end for this file - it's simple) and use the unit adapter to run the test at the same time to evaluate the code coverage.

```bat
"C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe" .\tst\MyProject.services.UnitTests\bin\Debug\MyProject.services.UnitTests.dll /EnableCodeCoverage /Settings:"c:\@work\testresults\MyProject.services.runsettings" /TestAdapterPath:"c:\@Work\zoomra\MyProject.Services\packages\xunit.runner.visualstudio.2.4.1\build\_common\"
```

## Sixth step
At this stage, this batch will generate the both files that sonar analyzer is expecting, the unit tests results, and the code coverage, but because each time that vstest.console runs, it generates a guid-folder with the results, so we did a little ninja script to copy those files for a fixed path to makes things smother with the rest of the script.

```bat
rem Move file from coverage test folder to previous
@echo off
set "build_dir=C:\@Work\TestResults\MyProject.services"
set "source_filename=MyProject.services.coverage"
for /f "tokens=* delims=" %%i in ('dir /b /a:d /o:-d /t:w "%build_dir%"') do (
set "last_build_dir=%%i"
goto :break
)
:break
set "source_file=%build_dir%\%last_build_dir%\%source_filename%"
 
@echo %source_file%
 
xcopy %source_file% "C:\@Work\TestResults\MyProject.services" /b/v/y
```
So now files are at the C:\@work\testresults\myproject.services and ready to match the params that we set up at first step. deal? 

## Seventh step
Sonar is only compatible with code coverage files in XML format, so our vstest.console is generating the report in a different format, so the next step is to convert for XML one:

```bat
"C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Team Tools\Dynamic Code Coverage Tools\amd64\CodeCoverage.exe" analyze /output:"C:\@Work\TestResults\MyProject.services\MyProject.services.coverage.xml" "C:\@Work\TestResults\MyProject.services\MyProject.services.coverage"
 
 
SonarScanner.MSBuild.exe end /d:sonar.login="3a325d1f9fed13b744ac92e2cffc30acb59fd702"
```
the latest instruction is to inform the sonar analyzer that all our instructions are now finished and the engine should now process the results and update the project dashboard on Sonar's console:

![sonar1](/assets/img/posts/sonar_1.png)

Code coverage results per each code layer:

![sonar1](/assets/img/posts/sonar_2.png)

Voilá, you have your project configured. After we've done a windows task scheduler to call this bat every night, so even if our code doesn't have changes we're always granting that everything stays smooth with quality grades and code coverage percentage.

So this is one way to integrate the sonar step with a CD/CI system (TFS, team city, etc) and having a specific code analyzer for your projects to ensure quality and security hotspots.

Thank you for your reading, here you can find the samples:

[Sonar-Script-Run-Sample.txt (2.22 kb)](/assets/samples/myproject.runsettings)

[myproject.runsettings (787.00 bytes)](/assets/samples/Sonar-Script-Run-Sample.txt)