
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
First to run the code on our own server we change "runs-on
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUwMDEwNDM1OCwyODkxMzQzNjVdfQ==
-->