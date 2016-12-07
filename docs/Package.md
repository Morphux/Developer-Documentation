Morphux Package Skeleton
========================

## Basic Skeleton
```
.:
package.json  scripts  srcs  patches

./scripts:
after.sh  before.sh

./srcs:
main.c
Makefile

./patches:
name.patch
```

## Basic template
```json
{
	"package": {
		"name": "package-name",
		"version": "version",
		"description": "Short description of the package"
	},
	"compilation": {
		"patches": ["name.patch"],
		"configure": [
			{"option-with-argument": "value"},
			"option-without-argument",
		],
		"make": "",
		"test": "",
		"install": "install",
	},
	"dependencies": {
		"needed": [
			"package-version"
		],
		"recommended": [],
		"optionnal": []
	}
}
```

### Package Section
```json
	"package": {
		"name": "package-name",
		"version": "version",
		"description": "Short description of the package"
	}
```
- `name` is the package name. Like `openssh`
- `version` is the package version. Like `7.0-p1`
- `description` is a short description about the package.

### Compilation Section
```json
	"compilation": {
		"patches": ["name.patch"],
		"configure": [
			{"option-with-argument": "value"},
			"option-without-argument",
		],
		"make": "",
		"test": "",
		"install": ""
	}
```
- `patches` is an array of patches file to apply _before_ the compilation.
All the patches must be stored in the patches/ directory.
- `configure` is an array of ./configure options. In this example the output
command will be: `./configure --option-with-argument=value --option-without-argument`

Note that the options with argument are defined through an object.
- `make` is the default make command. If blank, the command `make` is executed.
- `test` describe the test command, through a makefile. For example, if the value is
`test-all`, the command executed will be `make test-all`
- `install` is the default install command. If blank, the command `make install`
is executed.

### Dependencies section
```json
"dependencies": {
		"needed": [
			"package-version"
		],
		"recommended": [],
		"optionnal": []
	}
```
- `needed` is a list of packages that _must_ be present and installed in order
to compile / install the package. The format is the following: `package-version`.
For example `openssl-7.0`.
- `recommended` is a list of packages that improve the stability of the package.
- `optionnal` is a list of packages totally optionnals. For example, documentation.

## Before and After script
In the directory `script/` we can found the two differents scripts:
- `before.sh`
- `after.sh`

These are simple shell scripts executed before and after the install.

**Note**: Since we don't know what shell is installed user-side, all install scripts
_must_ be executed with /bin/sh.

## Optionnal
All optionnals sections must be added in the template file, after the required
sections.

### Group Management
If a package needs to add a group to the system, the following must be used:
```json
"group": [
	{
		"group_name": "group_login",
		"guid": 50
	}
]
```
Note that groups will always be added **before** the users

### User Management
If a package needs to add a user to the system, the following must be used:
```json
"user": [
	{
		"username": "login",
		"comment": "comment about the user",
		"home_dir": "/default/home/directory",
		"group": "default_group",
		"shell": "/bin/false",
		"uid": "50"
	}
]
```

### Kernel Configuration
If a package need a particular kernel configuration, the following must be used:
```json
"kernel": [
	"KERNEL_CONFIG_NAME_1",
	"KERNEL_CONFIG_NAME_2"
]
```

### SysV Configuration
If a package need a init.d script, the following must be used:
```json
"init": [
	// Not decided yet
]
```
