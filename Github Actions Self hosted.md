
---
title: GitHub Actions
---

## Introduction

Most tutorial I see online for Github actions are for build and testing and if at all the CD part is included its for installing the application in Azure environment. In my case I want to deploy on my own virtual machine (could be on Azure, AWS, GCP, or any other provider). This document is create as I follow tutorials around the web and compile it into one that will work for me.
```
name: .NET

on:
	push:
		branches: [ main ]
	pull_request:
		branches: [ main ]
	jobs:
		build:
			runs-on: ubuntu-latest
	steps:
	- uses: actions/checkout@v2
	- name: Setup .NET
	uses: actions/setup-dotnet@v1
	with:
		dotnet-version: 5.0.x
	- name: Restore dependencies
		run: dotnet restore
	- name: Build
		run: dotnet build --no-restore
```
First to run the code on our own server we change "runs-on" value.
~~runs-on: ubuntu-latest~~
runs-on: self-hosted


## Setting Up
On the main menu select Settings for the project and click on “Actions”. Make sure “Allow all actions” is selected.

When you clicked on “Actions” you should have noted that additional actions opened up and we are actually in the “General” menu. Click on “Runners” and click on “Add runner”. Select appropriate operating system and architecture. You
should have a real valid reason not to select Linux and Architecture X64.

## On the terminal
We now have some copy and paste work to do. From the download section of the
“Runners” page select each command that will

-   Create a folder
-   Download the latest runner package
-   Extract the installer
Then follow the instructions under Configure; *we have already attended to “Using your self-hosted runner”.*

## Changes-2

The build command “dotnet build” generates a debug enabled code and in production, unless the system is for internal use, it is better we run the application in release mode. So change the command:

`run: dotnet build --no-restore`
`to`
run: dotnet publish -r linux-x64 --self-contained false --configuration Release

When the commands run successfully your executables will be generated several
folders deep in a bin/……./Publish folder. I built a script in the server that
will copy the executables from publish folder to the folder where we want to
execute our executables. To do this I added another section as follows:

| \- name: Install |
|------------------|
|                  |

run: /home/srvean/bin/installApps.sh

## Final note:

When we followed the steps under “On the terminal” =\> Configure we started the
jobs that will execute the yaml instructions with the command “run.sh”. This
needs you log into server and start run.sh manually. We can convert this action
into a service by running the commands

cd \~/actions-runner

sudo svc.sh install

sudo svc.sh start

sudo svc.sh status
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAzMTQwNzM4MywtNzI3NDQyNDAyLDI4OT
EzNDM2NV19
-->