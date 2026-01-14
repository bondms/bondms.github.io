---
title: Bash script template
---

# Bash script template

Bash is a very convenient scripting language to use, primarily due to its availability; it is installed as the default shell on many Posix operating systems and can often be added to non-Posix operating systems (e.g. by installing Cygwin on Windows).

## Common pitfalls

Although convenient, there are some pitfalls of using Bash that should be avoided, such as:

### Not checking for errors

Unlike many languages (including C++ and Python) that can throw exceptions when an error occurs, bash normally indicates failure through return values. If the value is not checked, program execution continues to the next statement. This is often undesirable since it may cause code that expects a particular precondition to be executed without that precondition being met.

Use of ```set -e``` and ```set -o pipefail``` go some way toward addressing this issue, but they still don't guaratee immediate exit in the case of an error. For example, consider:

```bash
set -e
set -o pipefail

function something()
{
  do_something
  return 0
}

something || something_else
```

In this example, the ```something``` function assumes that ```set -e``` will cause the script to terminate if ```do_something``` fails, so it returns 0 to indicate success if executation passes that point. However, because ```something``` is called as part of a list, the behaviour is not as intended, ```something``` will still return success and ```something_else``` will not be called.

A workaround, albeit one that is tedious and easy to forget, is to ensure that the result of every command is tested, e.g. ```do_something || exit 1```.

### Assuming the working directory is the same directory as the script's location

A script may be launched from an environment where the current working directory (CWD) is not the same as the directory containing the script. In this case, relative paths (i.e. those relative to the CWD) will be incorrect.

There are two approaches to avoiding this.

#### Avoid use of relative paths

Avoid referring to paths relative to the CWD by instead prefixing them with the absolute path to the location of the script, e.g.

```
#!/bin/bash

HERE="$(readlink --canonicalize-existing "$(dirname "${BASH_SOURCE[0]}")")"

some_command -- "${HERE}/some_file_located_alongside_the_script" || exit 1
```

#### Change the CWD for the duration of the script

Sometimes avoiding the use of relative paths is difficult, for example you might need to call an external command that looks for files relative to the CWD. In that case, the CWD can be changed to the script's location at the start of execution, e.g.

```
#!/bin/bash

HERE="$(readlink --canonicalize-existing "$(dirname "${BASH_SOURCE[0]}")")"
pushd -- "${HERE}" || exit 1

...
```

### Not allowing for special characters in paths

As described in my article on [filesystem iteration](https://bondms.github.io/2018/02/04/filesystem-iteration.html), posix file and directory names can contain a variety of special characters that may be unintentionally interpretted by the shell. Care needs to be taken to avoid this. Some techniques that may be helpful include:

* Quoting path arguments.
* Separating option arguments from path arguments with `--`.

For example:

```
!/bin/bash

ls -l -- "$1"
```

Without the `--`, if a string beginning with a hypen is passed as `$1`, the `ls` command would interpret it as an option rather than as a path.

Without the quotes, if a path containing spaces is passed as `$1`, the `ls` command would interpret it as two separate paths.

## Example template

```bash
#!/bin/bash

# stdout is sent to stdout and logged as info priority.
# stderr is sent to stderr and logged as error priority.
exec 1> >(tee >(logger -t scripting -p user.info)) 2> >(tee >(logger -t scripting -p user.error) >&2)

# xtrace is sent to FD 3 (if valid) and logged as debug priority.
if [[ -e /proc/self/fd/3 ]]
then
    exec 3> >(tee >(logger -t scripting -p user.debug) >&3)
else
    exec 3> >(logger -t scripting -p user.debug)
fi
BASH_XTRACEFD=3
set -x

set -e
set -o pipefail
set -u
echo "$0 started."

function log_info()
{
    echo "$(basename "$0" .sh): $*"
}

function log_error()
{
    echo "$(basename "$0" .sh): ERROR: $*" >&2
}

function abort()
{
    log_error "$@"
    exit 1
}

HERE="$(readlink --canonicalize-existing "$(dirname "${BASH_SOURCE[0]}")")"
[[ -d "${HERE}" ]] || abort "Failed to locate script."
pushd -- "${HERE}" || exit 1

# Overridable settings.
MUTEX_PATH="${MUTEX_PATH:-/var/lock/scripting_$(basename "$0" .sh)}"

(
  # Release sudo on exit.
  trap "sudo --remove-timestamp" EXIT || exit 1

  # Use a flock as a mutex to ensure that one instance of the script
  # can execute at any one time.
  flock -n 9 || abort "Failed to acquire mutex lock."

  log_info "HERE=${HERE}"
  log_error "Example error message."
  abort "Development in progress"
) 9>"${MUTEX_PATH}"

echo "$0 completed."
```
## Example usage

To execute the script with trace logging sent to standard output (along with info logging which is sent there by default):

```bash
./script.sh 3>&1
```
