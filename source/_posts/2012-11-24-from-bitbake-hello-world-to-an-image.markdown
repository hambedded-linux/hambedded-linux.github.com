---
layout: post
title: "From Bitbake Hello World To an Image"
date: 2012-11-24 17:49
comments: true
categories: 
    - doc
    - bitbake
    - openembedded
---

[OpenEmbedded][openembedded] and [Yocto][yoctoproject] are based on
[bitbake][bitbake] build system. In order to understand the structure of OE, it
is beneficial to have an understanding of bitbake to some degree along with the
tasks and classes defined in OE. This document will, hopefully, present the "big
picture" of creating an image using OpenEmbedded and Yocto. It will start from a
very basic bitbake project, explain the important concepts, delve into OE
classes, and finally show the image creation process at its basics.

<!--more-->

**NOTE**: This document is draft and incomplete. OpenEmbedded part is still
being written.

Table Of Contents
=================
* Table of Contents
{:toc}

Bitbake
=======
BitBake is, at its simplest, a tool for executing tasks and managing metadata
[\[1\]][bitbake-doc]. What bitbake does, basically, is to handle the tasks
defined in bitbake metadata (.bbclass, .bb), resolve their task dependencies,
put them in right order and run them. You can think of a task as a function that
can be set to run before or after another function (or task). A task can also
have additional flag(s) assigned besides the dependency information. Flags are
basically variables assigned to variables or functions (tasks) that tell additional info
about them. All of these will be explained throughout this document. [Don't
panic!][hhgttg-dont-panic]

{% comment %}
Explain the following questions in tasks section
- what is a task?
- How tasks are added, dependency resolution happens?
- What is a flag, how flags are defined?
- How these flags are used? E.g: cleandirs, dirs flag used by bitbake. Give
  reference to the code executed (bb/build.py)
{% endcomment %}

Before we begin our hello world example, it is important to note that what
bitbake parses is a bitbake metadata. Hence, it has its own syntax, variable
assignments, function definitions, etc. Do not confuse it with a regular shell
scripts, although bitbake syntax resembles to them. Before proceeding, please
read [metadata section][bitbake-metadata] of [bitbake user
manual][bitbake-user-manual].

Obtaining Bitbake
-----------------
Let's obtain the bitbake source and extract it. As of this date, the latest
bitbake release is 1.16.0.

`````` bash
wget http://git.openembedded.org/bitbake/snapshot/bitbake-1.16.0.tar.bz2
tar xvf bitbake-1.16.0.tar.bz2
cd bitbake-1.16.0
``````

Instead of installing bitbake, we will run it in its build directory. Now, build
bitbake:

`````` bash
# FIXME: how to disable xml doc generation? It takes a little time for this doc.
python setup.py build
export PYTHONPATH=`pwd`/build/lib
``````

Now that we have built our bitbake, let's run it.

`````` bash
./bin/bitbake --version
BitBake Build Tool Core version 1.16.0, bitbake version 1.16.0
``````

If you are using vim, please copy the files in *contrib/vim* directory to ~/.vim
so that you have a decent syntax highlighting support for bitbake configuration
files, bbclass files, and bb recipes.

We are now ready to create our helloworld bitbake project.

Hello World Project
-------------------
Starting with the fundamentals, we will gradually evolve and see how bitbake
works by playing with it. In this part, we will cover what a task is, how it is
defined, how a task can be overwritten, what is a flag, and how a flag is used
by bitbake. With these informations in mind, we will also see how all of these
can come together to form a useful build system that creates embedded linux
images. After we have covered the concepts written above, we will delve into
OpenEmbedded code and see the ideas implemented.

*NOTE: This part is written with the help of the information in
[emails][bitbake-helloworld-email] sent to yocto project mailing list.*

Bitbake, when unpacked, includes a simple base.bbclass. You can find it in
*classes/base.bbclass*. It includes a few functions for printing infos and
warning, and defines 3 tasks: showdata, listtasks, build. 

When bitbake is run to build a recipe, this *base.bbclass* file gets inherited
by any recipe by default.  This is an ideal place for us to define our package
building logic. If you are familiar with building programs for UNIX-like
operating systems, you know that building a package includes fetching the
source, unpacking, patching, configuring, compiling (making), and installing.
Let's now add a useful base for our package building logic through defining tasks.


### Defining The Tasks ###
Defining a task is achieved through *addtask* keyword followed by the name of
the task as well as its dependency relation (before/after keywords). As you see
in the example bbclass, the name of the task and the actual definition of the
task is different. The code defining the task is prepended with *do_*. Tasks can
be defined as python executables or shell. Lets define our package building
logic. Edit *base.bbclass* and copy&paste the contents below:

`````` bash

bbplain() {
	echo "$*"
}

bbnote() {
	echo "NOTE: $*"
}

bbwarn() {
	echo "WARNING: $*"
}

bberror() {
	echo "ERROR: $*"
}

bbfatal() {
	echo "ERROR: $*"
	exit 1
}

addtask fetch
python base_do_fetch() {
    bb.note("BASE_DO_FETCH")
}

addtask unpack after do_fetch
python base_do_unpack() {
    bb.note("BASE_DO_UNPACK")
}

addtask patch after do_unpack
base_do_patch() {
    bbnote "BASE_DO_PATCH"
}

addtask configure after do_patch
base_do_configure() {
    bbnote "BASE_DO_CONFIGURE"
}

addtask make after do_configure
base_do_make() {
    bbnote "BASE_DO_MAKE"
}

addtask install after do_make
base_do_install() {
    bbnote "BASE_DO_INSTALL"
}

addtask build after do_install
base_do_build() {
    bbnote "DO_BUILD"
}

EXPORT_FUNCTIONS do_fetch do_unpack do_patch do_configure do_make do_install do_build

``````

As you see, I've defined the first functions as python, and the others as
shells to be an example. I also printed only the function names
instead of actually doing useful stuff for building a package. In that way, it's
easier to read the program logic from logs.

Having logging code and building logic in one file is not modular. Wouldn't it
be good to separate logging code from base? Let's create *logging.bbclass* in
*classes/* directory. Logging.bbclass looks like this: 

`````` 
bbplain() {
	echo "$*"
}

bbnote() {
	echo "NOTE: $*"
}

bbwarn() {
	echo "WARNING: $*"
}

bberror() {
	echo "ERROR: $*"
}

bbfatal() {
	echo "ERROR: $*"
	exit 1
}

``````

We will just put *inherit logging* at the top of base.bbclass. When base.bbclass is
parsed, logging.bbclass is now automatically inherited. Inheritence is one of
the many ways of modularity in bitbake.

`````` bash

inherit logging

addtask fetch
python base_do_fetch() {
    bb.note("BASE_DO_FETCH")
}

...

`````` 


### Adding an Example Layer ###
Having the program building logic and logging written, we need to add a layer to
start experimenting with building. First, create a directory named *meta-test*,
which will be our layer directory.  

`````` bash

mkdir meta-test

``````

We, then, need to inform bitbake that we have a layer. We will create
*conf/bblayers.conf* with following content:

`````` bash

BBPATH := "${TOPDIR}"
BBFILES ?= ""
BBLAYERS = " \
  ${TOPDIR}/meta-test \
    "

``````

${TOPDIR} is not defined anywhere in our \*.conf files. This variable, when
undefined, will be defined by bitbake itself in
*lib/bb/parse/parse_py/ConfHandler.py:36*. 

`````` python

(...)

def init(data):
    topdir = data.getVar('TOPDIR')
    if not topdir:
        data.setVar('TOPDIR', os.getcwd())

(...)

``````

As you see. TOPDIR is defined to be the directory in which we run bitbake. The
use of BBPATH and BBLAYERS is explained in [user
manual][bitbake-manual-parsing]:

    Bitbake will first search the current working directory for an optional
    "conf/bblayers.conf" configuration file. This file is expected to contain a
    BBLAYERS variable which is a space delimited list of 'layer' directories. For
    each directory in this list a "conf/layer.conf" file will be searched for and
    parsed with the LAYERDIR variable being set to the directory where the layer was
    found. The idea is these files will setup BBPATH and other variables correctly
    for a given build directory automatically for the user.

    Bitbake will then expect to find 'conf/bitbake.conf' somewhere in the user
    specified BBPATH. That configuration file generally has include directives
    to pull in any other metadata (generally files specific to architecture,
    machine, local and so on.

Additionally, *BBPATH* is analogous to PATH variable. Configuration files can be
included in bitbake.conf or other \*.conf files. When relative path is given,
the file is searched in the directories set in BBPATH, and the first hit will be
included.

Continuing with our layer, let's add our layer configuration. Create *conf/*
directory first:

`````` bash

mkdir meta-test/conf

``````

And add *conf/layer.conf* with the following contents:

`````` bash

BBPATH .= ":${LAYERDIR}"

BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
            ${LAYERDIR}/recipes-*/*/*.bbappend"

BBFILE_COLLECTIONS += "test"
BBFILE_PATTERN_test := "^${LAYERDIR}/"
BBFILE_PRIORITY_test = "5"

``````

We append our layer directory to *BBPATH* so that our layer directory is also
searched when bitbake looks for a configuration file which is relatively
included.

*BBFILES* variable is appended with our recipes. We tell bitbake that we have
recipes in our layer directory. All the recipes we create in
recipes-XXX/YYY/ directories within our LAYERDIR will get appended to BBFILES.

**FIXME:** Add explanations for COLLECTIONS, PATTERNS, and PRIORITY

Lets create our example recipe within the directory structure we have just
defined above.  This recipe is named *firstrecipe*.

`````` bash

mkdir -p meta-test/recipes-example/firstrecipe
vim meta-test/recipes-example/firstrecipe/firstrecipe_0.0.bb

``````

Using your favorite editor, create firstrecipe_0.0.bb. The version of a recipe
is contained with its filename seperated by \_. Since we do not have version for
our recipe, we make it 0.0. Our recipe will only contain a description.

`````` bash

DESCRIPTION = "Our first recipe for bitbake hello world"

``````

Finally, our conf, classes and meta-test directories are organized as below:

`````` bash
.
|-- classes
|   |-- base.bbclass
|   `-- logging.bbclass
|-- conf
|   |-- bblayers.conf
|   `-- bitbake.conf
|-- meta-test
|   |-- conf
|   |   `-- layer.conf
|   `-- recipes-example
|       `-- firstrecipe
|           `-- firstrecipe_0.0.bb
\-- 

``````


We are now ready to test our logic in base. Cd into the bitbake directory and
run bitbake with debug on.

`````` bash

./bin/bitbake firstrecipe -vDD

``````

Congratulations! Your package building logic is in action! You can check the
additional log information in *tmp/work/firstrecipe-0.0-r0/temp/*. Morover, the
code run in each task is written to *temp* directory with name "run_TASK.PID".
Check these files to see what is run in each task. Bitbake includes the logs of
all tasks in temp/ directory. Additionally, please look at *log.task_order*
file, which includes our task order defined in base.

It is now up to you to add required code for tasks defined in base to get a
functional build system (actual fetcher, patcher, extractor code etc). Don't
invent the wheel again, OpenEmbedded already built one :)

### Overriding Tasks ###
Defining the package building logic is not enough to create a useful build
system. There are different ways of configuration, making, and installing
provided by such tools as autotools, cmake, scons, etc. Although the routines
for getting the source, unpacking, and patching it can be applied to all
recipes, we need to override *do_configure, do_make, do_install* tasks for
different recpies.

Assume that our firstrecipe uses autotools which means that it has *configure*
script, we can build it using *make*, and install it with *make install*.
Instead of providing an actual code, we will again just print a line in
autotools package. Now, create *autotools.bbclass* in *classes/* directory with
the following contents:

`````` bash
autotools_do_configure() {
    bbnote "AUTOTOOLS_CONFIGURE"
}

autotools_do_make() {
    bbnote "AUTOTOOLS_MAKE"
}

autotools_do_install() {
    bbnote "AUTOTOOLS_INSTALL"
}

EXPORT_FUNCTIONS do_configure do_make do_install

``````

Inherit the autotools package in our firstrecipe.

`````` bash

DESCRIPTION = "Our first recipe for bitbake hello world"

inherit autotools

``````

We are now ready to build our firstrecipe using different
do_{configure,make,install} tasks.

`````` bash

./bin/bitbake firstrecipe -vDD

``````

Now, please go to *temp* directory and see which code is run in configure, make,
and install tasks. Without *inherit* keyword in our recipe, bitbake runs the
following code on do_configure task:

`````` bash

#!/bin/sh -e
export HOME="/Users/eren"
export SHELL="/bin/zsh"
export LOGNAME="eren"
export USER="eren"
export TERM="screen-256color"
export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin:/Users/eren/.zsh_scripts:/sbin:/usr/sbin:/Users/eren/.zsh_scripts"
do_configure() {
    base_do_configure

}

base_do_configure() {
    bbnote "BASE_DO_CONFIGURE"

}

bbnote() {
	echo "NOTE: $*"

}

set -x
cd /Users/eren/sourcebox/bitbake-doc/tmp/work/firstrecipe-0.0-r0/firstrecipe-0.0
do_configure

``````


However, when autotools is inherited, the following code is run:

`````` bash

#!/bin/sh -e
export HOME="/Users/eren"
export SHELL="/bin/zsh"
export LOGNAME="eren"
export USER="eren"
export TERM="screen-256color"
export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin:/Users/eren/.zsh_scripts:/sbin:/usr/sbin:/Users/eren/.zsh_scripts"
do_configure() {
    autotools_do_configure

}

autotools_do_configure() {
    bbnote "AUTOTOOLS_CONFIGURE"

}

bbnote() {
	echo "NOTE: $*"

}

set -x
cd /Users/eren/sourcebox/bitbake-doc/tmp/work/firstrecipe-0.0-r0/firstrecipe-0.0
do_configure

``````

Can you spot the difference? When autotools is inherited, do_configure() calls
autotools_do_configure(). This is achieved by EXPORT_FUNCTIONS keyword.
EXPORT_FUNCTIONS maps each task in its argument to the functions in the bbclass
prefixed with the name of that bbclass.  So, when EXPORT_FUNCTIONS is used in
autotools.bbclass, do_configure is mapped to autotools_do_configure. We did the
same in *base.bbclass* file. We used EXPORT_FUNCTIONS, it mapped each task to
base_do_TASKNAME. This type of abstraction has an advantage that when we
override a task, we do not lose the original task. For example, in
autotools_do_configure, we can still call base_do_configure.

**FIXME:** I need clarification on how export_functions works and how these
tasks gets mapped.

**FIXME:** task flags have not been mentioned, dirs, cleandirs etc. Mention how
bitbake interprets it, what's the need, etc

Summary
-------
We have covered all the necessary information about bitbake. We know how tasks
are defined, how layers can be added, how bitbake parses configuration and
metadata files, how tasks can be overwritten. With these information, we are now
ready to delve into openembedded code.


Understanding OpenEmbedded
==========================
OpenEmbedded provides a complete build system and a number of recipes. It has a
lot of code for fetching the source, patching, building, installing, producing
ipk, deb, rpm packages, and making an image etc. OpenEmbedded also provides
other features such as caching but it's out of the scope of this document.

**FIXME**: Mention the distinction/common things with yocto and openembedded

Getting The Source
------------------
As of this date, the latest stable version of Yocto is *danny*. Cd into your
working directory and get the source.

`````` bash

wget http://downloads.yoctoproject.org/releases/yocto/yocto-1.3/poky-danny-8.0.tar.bz2
tar xvf poky-danny-8.0.tar.bz2
cd poky-danny-8.0

``````

Let's move on to explaining the configuration files.


Configuration
-------------
As we have seen, bitbake.conf is parsed when bitbake is run. This file is among
the important files in OE. The file is in *meta/conf/* directory and it includes a
number of configuration variables that are used in metadatas and bb recipes.
Bitbake.conf is not standalone and it includes other configuration files within
*conf* directory. Please navigate to the line no 677 in bitbake.conf and see
that there are other configuration files appended into bitbake.conf for
modularity.

Please do not miss the lines that make use of variables. This is what allows you
to build an image for different architectures by only changing a few variables.

`````` bash

include conf/build/${BUILD_SYS}.conf
include conf/target/${TARGET_SYS}.conf
include conf/machine/${MACHINE}.conf
include conf/machine-sdk/${SDKMACHINE}.conf
include conf/distro/${DISTRO}.conf

``````

Note that these inclusions are done with relative path to the configuration files.
These configuration files will be searched in the directories defined in BBPATH
variable. It means that when bitbake is run, these configuration files will be
searched in all layers you defined, and the first hit will be used. Remember that
the path of all the layers are appended to BBBPATH variable.

Tasks
-----
In order to understand OE architecture, it's important to know which tasks are
defined where. We have seen that base.bbclass is an ideal place to define our
logic and put common code. All bbclass files are in *meta/classes/* directory.
Open base.bbclass file and search for "addtask" keyword. In base.bbclass, the
following tasks are defined:

`````` bash

...
addtask fetch

...
addtask unpack after do_fetch

...
addtask configure after do_patch

...
addtask compile after do_configure

...
addtask install after do_compile

...
addtask build after do_populate_sysroot

...
addtask cleansstate after do_clean

...
addtask cleanall after do_cleansstate

``````

However, these are not the only tasks defined. Take attention to the "inherit"
keywords on the top. There are tasks defined in {patch,staging}.bbclass files.
Let's see which tasks are defined.


`````` bash

// patch.bbclass
addtask patch after do_unpack

// staging.bbclass
addtask populate_sysroot after do_install
addtask do_populate_sysroot_setscene

``````

These tasks are also not enough. In bitbake.conf,
*conf/distro/defaultsetup.conf* is included. It defines the following packages
that should be inherited automatically: *package_ipk*, *insane*, *debian
devshell sstate license*, among which some of them define tasks. Here is a code
portion that defines inheritence:

`````` bash

USER_CLASSES ?= ""
PACKAGE_CLASSES ?= "package_ipk"
INHERIT_INSANE ?= "insane"
INHERIT_DISTRO ?= "debian devshell sstate license"
INHERIT += "${PACKAGE_CLASSES} ${USER_CLASSES} ${INHERIT_INSANE} ${INHERIT_DISTRO}"

``````

Additional to the tasks that we have covered so far, some of the classes that
are automatically inherited also define tasks.

[openembedded]: http://localhost/
[yoctoproject]: http://localhost/
[hhgttg-dont-panic]: https://en.wikipedia.org/wiki/Phrases_from_The_Hitchhiker%27s_Guide_to_the_Galaxy#Don.27t_Panic
[bitbake]: http://localhost/
[bitbake-doc]: http://localhost/
[bitbake-metadata]: http://docs.openembedded.org/bitbake/html/ch02.html
[bitbake-user-manual]: http://docs.openembedded.org/bitbake/html/
[bitbake-manual-parsing]: http://docs.openembedded.org/bitbake/html/ch02s03.html
[bitbake-helloworld-email]: http://www.mail-archive.com/yocto@yoctoproject.org/msg09379.html

{% comment %}
- after explaining hello world bitbake and before delving into OE code, give the
  build diagram in getting started. Say it's not enough to understand the code,
  and start explaining.

{% endcomment %}
