---
title: Iterating over a filesystem
---

# Iterating over a filesystem

There are many computing tasks that are easy to do in a manner that works most of the time but which fail for edge cases. Iterating over files and/or directories in a filesystem, in order to perform a common action on each, is an example.

In this article I will primarily consider shell scripts; use of a programming language would avoid many of the issues described here but at the cost of requring an interpreter or compiler.

## Common pitfalls

### Failure to handle special characters

A naive implementation will often fail when encountering file or directory names that contain unusual characters, such as:

* Whitespace: space, tab, and even new-line characters are possible in file names.
* Quotes: single (‘) or double (“).
* Wildcards (* or ?).
* Special characters that are interpreted by the shell, such as a dollar sign ($) or backtick (`).
* Characters that are illegal on some filesystem such as colon (:) which is invalid on VFAT but valid on other filesystems.
* Hypens (-) that can be interpreted as option arguments.
* Non-ASCII characters such as the pound Sterling sign (£) or letters from a non-Latin alphabet (e.g. кирилица).

The ```find``` command, present on many POSIX-type operating systems, includes a ```-print0``` option that

> ... allows file names that contain  newlines or other types of white space to be correctly interpreted by programs that process the find output.

There is a corresponding ```--null``` or ```-0``` argument to ```xargs``` and ```--zero-terminated``` or ```-z``` argument to ```sort```.

Use of these arguments on their own is not always sufficient. The following script, for example, fails to correctly process paths containing single-quote (') characters.

```bash
find -print0 |
  xargs --null -I {} bash -c "echo '{}'"
```

```
$ touch File\ name\ with\ single\ quote\ \'.txt 
$ ls -la
total 0
drwx------ 2 bondms bondms 60 Feb  4 12:42 .
drwx------ 3 bondms bondms 60 Feb  4 12:41 ..
-rw------- 1 bondms bondms  0 Feb  4 12:45 File name with single quote '.txt
$ find -print0 | xargs --null -I {} bash -c "echo '{}'"
.
bash: -c: line 0: unexpected EOF while looking for matching `''
bash: -c: line 1: syntax error: unexpected end of file
```

The ```bash``` command string attempts to handle special characters within file names by enclosing the name within single quotes, but this doesn't work when a single quote character is present in the file name.

### Failure to correctly report an error

Use of the ```-exec``` and ```-execdir``` options of ```find``` without care can lead to this error. For example:

```bash
( find -execdir ls --bad-arg \{\} \; ) &&
  echo No error reported ||
  echo Error reported
```

```
$ ( find -execdir ls --bad-arg \{\} \; ) && echo No error reported || echo Error reported
ls: unrecognised option '--bad-arg'
Try 'ls --help' for more information.
No error reported
```

If this case, the error reported by the ```ls``` command was not propagated and the ```find``` command reported success.

### Failure to correctly handle no items

By default ```xargs``` runs once even if it receives no input which can cause failures. For example, the following script will work if run from a folder containing files but will fail if there are no files:

```bash
find -type f -print0 |
  xargs --null file
```

```
$ ls -la
total 0
drwx------ 2 bondms bondms 40 Feb  4 12:46 .
drwx------ 3 bondms bondms 60 Feb  4 12:41 ..
$ find -type f -print0 | xargs --null file
Usage: file [-bcEhikLlNnprsvzZ0] [--apple] [--extension] [--mime-encoding] [--mime-type]
            [-e testname] [-F separator] [-f namefile] [-m magicfiles] file ...
       file -C [-m magicfiles]
       file [--help]
```

The ```file``` command expects at least one argument and returns an error if called with no arguments.

### Failure to handle the maximum allowed command-line length

 ```xargs``` automatically splits long command-lines into multiple calls. The ```-execdir``` option of ```find``` has similar behaviour. A naive script that expects exactly one call may fail to handle this, for example:

```bash
find -type f -print0 |
  xargs --null wc
```

```
$ for (( i=100000 ; i=i-1 ; i )); do echo Test data > $i.txt; done
$ find -type f -print0 | xargs --null wc
     1      2     10 ./10000.txt
     1      2     10 ./10001.txt
     1      2     10 ./10002.txt
...
     1      2     10 ./31841.txt
     1      2     10 ./31842.txt
     1      2     10 ./31843.txt
 10922  21844 109220 total
     1      2     10 ./31844.txt
     1      2     10 ./31845.txt
     1      2     10 ./31846.txt
...
     1      2     10 ./42763.txt
     1      2     10 ./42764.txt
     1      2     10 ./42765.txt
 10922  21844 109220 total
     1      2     10 ./42766.txt
     1      2     10 ./42767.txt
     1      2     10 ./42768.txt
...
   1    2   10 ./7.txt
   1    2   10 ./8.txt
   1    2   10 ./9.txt
 935 1870 9350 total
```

The wc command prints a line for each file processed plus a summary line for the total of all the files. However, if multiple calls are made (because there are many items being processed) then there will be several summary lines, each giving the total of a subset of the files. The final line of output will not give a correct summary of all the items processed.

## Considerations

* Terminate filenames with nul, the only character not allowed in POSIX paths (find -print0, sort --zero-terminated, xargs --null).
* Don't use the -exec option of find if you care about security. Instead use -execdir.
* Don't use the -execdir option of find if you care about catching errors; it hides the exit code of the command. Instead use xargs or --execdir with the + suffix.
* Use single quotes around filenames to protect special characters (such as `"$) from being mishandled:
* Escape (all) single quotes in the filenames if passing to a shell (sed with global search and replace).
* If necessary, handle the case where there are no items found (xargs --no-run-if-empty option).
* Consider whether it is necessary to limit the processing one item at a time (xargs --max-args or -I options) .
* Consider which types of files to process; files, directories, links, etc. (find -type option).
* Consider whether to include the top-level directory. For find, this is "." if no path is specified. Exclude with -mindepth 1 find option.
* Consider whether to recurse into subfolders. Disable with find -maxdepth 1 option.
* Consider whether to follow symlinks and whether to traverse across filesystems.
* Specify the arguments to find in the appropriate order, especially -mindepth, -maxdepth etc.
* Find appears to precede the item with a path (./ if no root search path is specified), so filenames that look like options (e.g. --file-name.txt) are not a problem, but it still might be good practice to consider these and protect against injection by using the -- argument to commands that support it.
* Beware of long command-lines being split into multiple.

## Examples

Multi-item processing (no -I or --max-args passed to xargs):
```bash
find -mindepth 1 -maxdepth 1 -type f -print0 |
    sort --zero-terminated |
    xargs --null --no-run-if-empty chmod --verbose -w
```

Single-item processing (otherwise there won't be one "Found:" line printed for each hit:
```bash
find -print0 |
    xargs --null --no-run-if-empty --max-args 1 echo "Found: "
```

Single-item processing with replacement string that doesn't go at the end of the command (use single quotes around the {}):
```bash
find -print0 |
    xargs --null -I{} echo 'This item {} has been found.'
```

Processing via a shell call:
```bash
find -type f -print0 | bash -c "
    while read -r -d $'\0' F
    do
        echo \"\$F\"
    done"
```

TODO: Actually this does work with dollar signs!
Attempting to solve this problem by escaping single quotes makes the script more complicated and yet it now doesn't work with paths containing dollar characters ($):

```bash
find -print0 |
  sed "s/'/'\\\''/g" |
  xargs --null -I {} bash -c "echo '{}'"
```

TODO: This also appears to work now!
An alternative example of making a shell call, using env rather than sed (doesn't work with filenames containing double quote):

```bash
find -print0 |
    xargs --null -I '{}' env F='{}' bash -c "echo \"\$F\""
```
