# Chapter 1. Exploring Linux Command-Line Tools

A **shell** is a program that accepts and interprets text-mode commands and provides an interface to the system.

## Understanding Command-Line Basics

### Exploring Your Linux Shell Options

More common shells:
* bash - The GNU Bourne Again Shell ( bash ) is based on the earlier Bourne shell for Unix but extends it in several ways. **In Linux, bash is the most common default shell for user accounts, and it’s the one emphasized in this book and on the exam.**

In Linux, most users run bash because it is the most popular shell.

Be aware that there are two types of default shells: 
* **default interactive shell** - shell program a user uses to enter commands, run programs from the command line, run shell scripts, and so on. 
* **default system shell** - shell is used by the Linux system to run system shell scripts, typically at startup.
The file /bin/sh is a pointer to the system’s default system shell - normally /bin/bash for Linux. However, be aware that, on some distributions, the /bin/sh points to a different shell. For example, on Ubuntu, /bin/sh points to the dash shell, /bin/dash.

### Using Internal and External Commands







