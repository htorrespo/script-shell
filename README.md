# script-shell

Script Shell para sistemas operativos Unix / Linux

<!---
"K:\usuario\Downloads\abs-guide--Advanced Bash-Scripting Guide.pdf"
-->
When not to use shell scripts

- Resource-intensive tasks, especially where speed is a factor (sorting, 
hashing, recursion [2] ...)
- Procedures involving heavy-duty math operations, especially floating 
point arithmetic, arbitrary precision calculations, or complex numbers 
(use C++ or FORTRAN instead)
- Cross-platform portability required (use C or Java instead)
- Complex applications, where structured programming is a necessity 
(type-checking of variables, function prototypes, etc.)
- Mission-critical applications upon which you are betting the future 
of the company
- Situations where security is important, where you need to guarantee 
the integrity of your system and protect against intrusion, cracking, 
and vandalism
- Project consists of subcomponents with interlocking dependencies
- Extensive file operations required (Bash is limited to serial file 
access, and that only in a particularly clumsy and inefficient 
line-by-line fashion.)
- Need native support for multi-dimensional arrays
- Need data structures, such as linked lists or trees
- Need to generate / manipulate graphics or GUIs
- Need direct access to system hardware or external peripherals
- Need port or socket I/O
- Need to use libraries or interface with legacy code
- Proprietary, closed-source applications (Shell scripts put the source 
code right out in the open for all the world to see.)


In the simplest case, a script is nothing more than a list of system 
commands stored in a file. At the very least, this saves the effort 
of retyping that particular sequence of commands each time it is invoked.


```terminal
# cleanup1.sh

# Cleanup
# Run as root, of course.
cd /var/log
cat /dev/null > messages
cat /dev/null > wtmp
echo "Log files cleaned up."
```

```terminal
# cleanup2.sh

#!/bin/bash
# Proper header for a Bash script.
# Cleanup, version 2
# Run as root, of course.
# Insert code here to print error message and exit if not root.
LOG_DIR=/var/log
# Variables are better than hard-coded values.
cd $LOG_DIR
cat /dev/null > messages
cat /dev/null > wtmp
echo "Logs cleaned up."
exit # The right and proper method of "exiting" from a script.
# A bare "exit" (no parameter) returns the exit status
#+ of the preceding command.
```

```terminal
# cleanup3.sh
#!/bin/bash
# Cleanup, version 3

# Warning:
# -------
# This script uses quite a number of features that will be explained
#+ later on.
# By the time you've finished the first half of the book,
#+ there should be nothing mysterious about it.

LOG_DIR=/var/log
ROOT_UID=0 # Only users with $UID 0 have root privileges.
LINES=50 # Default number of lines saved.
E_XCD=86 # Can't change directory?
E_NOTROOT=87 # Non-root exit error.


# Run as root, of course.
if [ "$UID" -ne "$ROOT_UID" ]
then
echo "Must be root to run this script."
exit $E_NOTROOT
fi

if [ -n "$1" ]
# Test whether command-line argument is present (non-empty).
then
lines=$1
else
lines=$LINES # Default, if not specified on command-line.
fi

# Stephane Chazelas suggests the following,
#+ as a better way of checking command-line arguments,
#+ but this is still a bit advanced for this stage of the tutorial.
#
# E_WRONGARGS=85 # Non-numerical argument (bad argument format).
#
# case "$1" in
# "" ) lines=50;;
# *[!0-9]*) echo "Usage: `basename $0` lines-to-cleanup";
# exit $E_WRONGARGS;;
# * ) lines=$1;;
# esac
#
#* Skip ahead to "Loops" chapter to decipher all this.

cd $LOG_DIR

if [ `pwd` != "$LOG_DIR" ] # or if [ "$PWD" != "$LOG_DIR" ]
                           # Not in /var/log?
then
echo "Can't change to $LOG_DIR."
exit $E_XCD
fi # Doublecheck if in right directory before messing with log file.

# Far more efficient is:
#
# cd /var/log || {
# echo "Cannot change to necessary directory." >&2
# exit $E_XCD;
# }


tail -n $lines messages > mesg.temp # Save last section of message log file.
mv mesg.temp messages               # Rename it as system log file.

# cat /dev/null > messages
#* No longer needed, as the above method is safer.

cat /dev/null > wtmp # ': > wtmp' and '> wtmp' have the same effect.
echo "Log files cleaned up."
# Note that there are other log files in /var/log not affected
#+ by this script.

exit 0
# A zero return value from the script upon exit indicates success
#+ to the shell.
```

The sha-bang ( #!) [6] at the head of a script tells your system that 
this file is a set of commands to be fed to the command interpreter 
indicated. The #! is actually a two-byte [7] magic number.

Immediately following the sha-bang is a path name. This is the path to 
the program that interprets the commands in the script, whether it be a 
shell, a programming language, or a utility.

```terminal
#!/bin/sh
#!/bin/bash
#!/usr/bin/perl
#!/usr/bin/tcl
#!/bin/sed -f
#!/bin/awk -f
```

Each of the above script header lines calls a different command 
interpreter, be it /bin/sh, the default shell (bash in a Linux system)

#! can be omitted if the script consists only of a set of generic system 
commands, using no internal shell directives. The second example, 
requires the initial #!, since the variable assignment line, lines=50,
uses a shell-specific construct. Note again that #!/bin/sh invokes the 
default shell interpreter, which defaults to /bin/bash on a Linux machine.

