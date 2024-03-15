---
title: Iterating over a filesystem
---

# Iterating over a filesystem

There are many computing tasks that are easy to do in a manner that works most of the time but which fail for edge cases. Iterating over files and/or directories in a filesystem, in order to perform a common action on each, is an example.

In this article I will primarily consider shell scripts; use of a programming language would avoid many of the issues described here but at the cost of requring an interpreter or compiler.

## Common pitfalls

### Failure to handle special characters

A naive implementation will often fail when encountering file or directory names that contain unusual characters, such as:

* Whitespace: space, tab, and even new-line characters are possible in filenames.
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

```console
$ touch "File name with single quote '.txt"
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

The ```bash``` command string attempts to handle special characters within filenames by enclosing the name within single quotes, but this doesn't work when a single quote character is present in the filename.

### Failure to correctly report an error

Use of the ```-exec``` and ```-execdir``` options of ```find``` without care can lead to this error. For example:

```bash
( find -execdir ls --bad-arg {} \; ) &&
  echo No error reported ||
  echo Error reported
```

```console
$ ( find -execdir ls --bad-arg {} \; ) && echo No error reported || echo Error reported
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

```console
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

The ```file``` command expects at least one argument and returns an error if called with no arguments. The ```--no-run-if-empty``` option to ```xargs``` can help avoid this pitfall.

### Failure to handle the maximum allowed command-line length

 ```xargs``` automatically splits long command-lines into multiple calls. The ```-execdir``` option of ```find``` has similar behaviour. A naive script that expects exactly one call may fail to handle this, for example:

```bash
find -type f -print0 |
  xargs --null wc
```

```console
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

The ```wc``` command prints a line for each file processed plus a summary line for the total of all the files. However, if multiple calls are made (because there are many items being processed) then there will be several summary lines, each giving the total of a subset of the files. The final line of output will not give a correct summary of all the items processed.

## Considerations

In order to avoid these pitfalls and others, take care to consider the following points when writing scripts to iterate over filesystems:

* Terminate filenames with nul which is the only character not allowed in POSIX paths (```find -print0```, ```sort --zero-terminated```, ```xargs --null```, etc).
* Don't use the ```-exec``` option of find if you care about security.
* Be careful using the ```-exec``` or ```-execdir``` options of find if you care about catching errors; they can hide the exit code of the command.
* Consider using single quotes around filenames to protect special characters from being mishandled, but don't forget that a filename can itself contain single-quote characters that may need escaping.
* Consider the case where there are no items found.
* Consider whether to process one item at a time or combine them into a single call. If combining, be aware that there still may be multiple calls to avoid exceeding the maxmimum command-line length.
* If processing directories, consider whether to include the top-level directory. For ```find``` this can be excluded with ```-mindepth 1```.
* Consider whether to recurse into subfolders. For ```find``` this can be disable with ```-maxdepth 1```.
* Consider whether to follow symlinks and whether to traverse across filesystems.
* Consider whether it would be possible for a filename to be interpretted as a command option. Often commands accept ```--``` to specify that all following arguments are non-option arguments.

## Examples

Simple example. Print and delete content of a folder without deleting the folder itself. No need to pass through to a shell or batch items:

```bash
find directory_to_empty/ -mindepth 1 -print -delete
```

Processing items in batches. ```chmod``` accepts accepts one or more non-option arguments so doesn't need to be called once for each item processed:

```bash
find -type f -print0 |
  xargs --null --no-run-if-empty chmod --verbose -w --
```

Processing items one-by-one. In order to print a "Found: " message for each item individual processing of each item is required:

```bash
find -print0 |
  xargs --null --no-run-if-empty --max-args 1 echo "Found: "
```

If using ```-I{}``` then the ```--max-args 1``` is implicit:

```bash
find -print0 |
  xargs --null -I{} echo 'This item {} has been found.'
```

Processing via a shell call. Passing names into a shell (such as ```bash```) requires a little more work. Here are a few approaches that seem to work:

```bash
find -print0 |
  bash -c "
    while read -r -d $'\0' F
    do
      echo This item \"\$F\" has been found.
    done"
```

```bash
find -print0 |
  sed "s/'/'\\\''/g" |
  xargs --null -I{} bash -c "echo This item '{}' has been found."
```

```bash
find -print0 |
    xargs --null -I{} env F='{}' bash -c "echo This item \"\$F\" has been found."
```
