Morphux Server Infrastructure
=============================

## Overview

### Package Website

Website that list all the packages available on the distribution.
Languages: PHP / HTML / CSS

### Build system
System that build new packages / new versions of packages, on multiple
arch through VMs.

Languages: Python ?

### MPM Server
Server that respond to an user request to install a package.

Languages: C, Go, C++ ?


### Package Database
Server that stores informations about packages

Language: SQL

### Git Server
Server that stores maintainers and previous versions of a package.


## Life of a package in a nutshell

- Maintainer create a package
- Package is send to the build system
- VMS build the package, check if there's no dependencies problems
- Build system fill out the package automatic information:
	- Size of the installed package
	- Compilation time
	- Files installed by the package
	- Extract the binaries installed by the package
- The build system add the package to the Package database

## Step by step

### Creation of a package

- If the package is new to Morphux, the maintainer / admin have to create a new Git as follow
repository: gitpkg.morphux.org/packagename.git
- The master branch is the dev version of the package
- The git repository should contain the full package, sources included
- As soon as the maintainer / admin push the data into a branch like `version-XX.XX`,
the build system automatically test the package.

### Test a package
- The build system is mostly 'Vanilla' VMs of Morphux
- The build system test the package by cloning the maintainer repository, and execute
a test mpm binary.
- This binary will try to compile the package, with the informations provided by the maintainer in the `package.json`
- If the compilation works, the system will try to deduce some information:
	- Size of the installed package
	- Compilation time
	- Files installed and modified by the package
	- Extract the binaries installed by the package
- Then, the build system will add these informations into the package
- If all of those steps are sucessfull, the version is marked as good,
and the database server will take over

### Add a package to the database
- Some basic informations are extracted from the package, in order to respond quickly
on an install request:
	- Name of the package
	- Version of the package
	- Dependencies of the package
	- Compilation time of the package
	- Installed size of the package
	- Files installed by the package
- Once done, the package will be compressed, encrypted and moved to a
ftp server.
