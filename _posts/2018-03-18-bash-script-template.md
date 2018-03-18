---
title: Bash script template
---

# Bash script template

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
echo "$0 started."

function log_info()
{
    echo "$(basename "$0" .sh): $@"
}

function log_error()
{
    echo "$(basename "$0" .sh): ERROR: $@" >&2
}

function abort()
{
    log_error "$@"
    exit 1
}

HERE="$(readlink -f "$(dirname "$0")")"
[[ -d "${HERE}" ]] || abort "Failed to locate script."

# Overridable settings.
MUTEX_PATH="${MUTEX_PATH:-/var/lock/scripting_$(basename "$0" .sh)}"

(
    # Use a flock as a mutex to ensure that one instance of the script
    # can execute at any one time.
    flock -n 9 || abort "Failed to acquire mutex lock."

    log_info "HERE=${HERE}"
    log_error "Example error message."
    abort "Development in progress"
) 9>"${MUTEX_PATH}"

echo "$0 completed."
```
