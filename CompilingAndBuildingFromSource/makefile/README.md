# Makefile

#### Source: ftp://ftp.gnu.org/old-gnu/Manuals/make-3.79.1/html_chapter/make_4.html

--------------------------------------------------------------------------------
--------------------------------------------------------------------------------

The makefile tells make how to compile and link a program.

A simple makefile consists of "rules" with the following shape:

```
target ... : prerequisites ...
        command
        ...
        ...
```

A target is usually the name of a file that is generated by a program; examples of targets are executable or object files. A target can also be the name of an action to carry out, such as `clean`.
A prerequisite is a file that is used as input to create the target. A target often depends on several files.
A command is an action that make carries out. A rule may have more than one command, each on its own line. Please note: you need to put a tab character at the beginning of every command line! This is an obscurity that catches the unwary.
A rule explains how and when to remake certain files which are the targets of the particular rule. make carries out the commands on the prerequisites to create or update the target. A rule can also explain how and when to carry out an action.


## Examples:

In this example, all the C files include `defs.h`, but only those defining editing commands include `command.h`, and only low level files that change the editor buffer include `buffer.h`.

```
edit : main.o kbd.o command.o display.o \
       insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
                   insert.o search.o files.o utils.o

main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit main.o kbd.o command.o display.o \
           insert.o search.o files.o utils.o
```

To use this makefile to create the executable file called `edit`, type:`make`.

To use this makefile to delete the executable file and all the object files from the directory, type: `make clean`.


## How make Processes a Makefile:

By default, make starts with the first target (not targets whose names start with `.`). This is called the default goal.

`make` reads the `makefile` in the current directory and begins by processing the first rule. In the example, this rule is for relinking `edit`; but before make can fully process this rule, it must process the rules for the files that `edit` depends on, which in this case are the object files. Each of these files is processed according to its own rule. These rules say to update each `.o` file by compiling its source file. The recompilation must be done if the source file, or any of the header files named as prerequisites, is more recent than the object file, or if the object file does not exist.
We can use variable with makefiles.
It is not necessary to spell out the commands for compiling the individual C source files, because make can figure them out: it has an implicit rule for updating a `.o` file from a correspondingly named `.c` file using a `cc -c` command. For example, it will use the command `cc -c main.c -o main.o` to compile `main.c` into `main.o`. We can therefore omit the commands from the rules for the object files.

When a `.c` file is used automatically in this way, it is also automatically added to the list of prerequisites. We can therefore omit the `.c` files from the prerequisites, provided we omit the commands.

Here is the entire example, with both of these changes, and a variable objects as suggested above:

```
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
        -rm edit $(objects)
```
This is how we would write the makefile in actual practice.

When the objects of a makefile are created only by implicit rules, an alternative style of makefile is possible. In this style of makefile, you group entries by their prerequisites instead of by their targets. Here is what one looks like:

```
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h
```

Makefiles contain five kinds of things: explicit rules, implicit rules, variable definitions, directives, and comments:
- An explicit rule says when and how to remake one or more files, called the rule's targets. It lists the other files that the targets depend on, call the prerequisites of the target, and may also give commands to use to create or update the targets.
- An implicit rule says when and how to remake a class of files based on their names. It describes how a target may depend on a file with a name similar to the target and gives commands to create or update such a target.
- A variable definition is a line that specifies a text string value for a variable that can be substituted into the text later.
- A directive is a command for make to do something special while reading the makefile. These include:
    * Reading another makefile.
    * Deciding (based on the values of variables) whether to use or ignore a part of the makefile
    * Defining a variable from a verbatim string containing multiple lines.
- `#` in a line of a makefile starts a comment. It and the rest of the line are ignored, except that a trailing backslash not escaped by another backslash will continue the comment across multiple lines.

By default, when make looks for the makefile, it tries the following names, in order: `GNUmakefile`, `makefile` and `Makefile`. If you do not specify any makefiles to be read with `-f` or `--file` options, make will try the default makefile names.

The include directive tells make to suspend reading the current makefile and read one or more other makefiles before continuing. The directive is a line in the makefile that looks like this: `include filenames...`.

Filenames can contain shell file name patterns.

If you want make to simply ignore a makefile which does not exist and cannot be remade, with no error message, use the `-include` directive instead of `include`, like this:
```
-include filenames...
sinclude is another name for -include.
```

GNU make does its work in two distinct phases. During the first phase it reads all the makefiles, included makefiles, etc. and internalizes all the variables and their values, implicit and explicit rules, and constructs a dependency graph of all the targets and their prerequisites. During the second phase, make uses these internal structures to determine what targets will need to be rebuilt and to invoke the rules necessary to do so.
