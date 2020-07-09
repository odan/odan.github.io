---
title: Reset windows folder permissions
layout: post
comments: true
published: false
description: 
keywords: php, vfsStream
---


```bat
@echo off

rem Adjust the path here (without backslash at the end)
set VERZ=c:\xampp
echo %VERZ%

rem Transfer permissions to the current user for all files in a directory
rem takeown /F "%VERZ%\*.*" /R /D J
rem This command only works on english computers
takeown /F "%VERZ%\*.*" /R /D Y

rem Reset permissions to default
icacls "%VERZ%" /reset /t

rem Full rights for current user
icacls "%VERZ%" /grant %COMPUTERNAME%\%USERNAME%:(OI)(CI)f /T /c

rem From now on we have full access
echo done

pause
```
