---
title: Bash aliases
---

# Bash aliases

A sample Bash aliases file setting up some useful aliases as well as functions and non-persistent settings.

 ```bash
### Aliases ###

alias broken-links='find -L . -type l'
alias checktime='ntpdate -q uk.pool.ntp.org'
alias chmod='chmod -v'
alias chown='chown -v'
alias cp='cp -aiv'
alias df='df --si'
alias du='du --si'
alias firefox='firefox -no-remote'
alias follow-from-current='tail -F -n 0'
alias follow-from-start='tail -F -n +1'
alias g++debug='g++ -Wall -Werror -O0 -ggdb'
alias g++warn='g++ -Wall -Werror'
alias grep-context='grep -C 5'
alias gunzip='gunzip -v'
alias gzip='gzip -v'
alias hexdump='hexdump -C'
alias less='less -iM'
alias ln='ln -v'
alias ls='ls -la --si --color=auto'
alias mplayer-bg='mplayer -subcp WINDOWS-1251'
alias mv='mv -iv'
alias netstat='netstat -aveep'
alias openssl-view-cert='openssl x509 -noout -text -in'
alias openssl-view-pub-key='openssl rsa -pubin -text -in'
alias openssl-view-private-key='openssl rsa -text -in'
alias password-generate='openssl rand -base64 1024'
alias pstree='pstree -paul'
alias qiv='qiv -Rm'
alias radiofip-play-live='play http://direct.fipradio.fr/live/fip-midfi.mp3 channels 1'
alias radiovarna-play-live='play -t mp3 http://broadcast.masters.bg:8000/live channels 1'
alias rm='rm -v'
alias rmdir='rmdir -v'
alias rmdir-and-content='find "$(readlink -f .)" -delete'
alias rmdir-content='find -mindepth 1 -delete'
alias ro-files='find -type f -print0 |
    xargs --null --no-run-if-empty chmod --verbose a-w'
alias orientate='find -type f \( -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.jpe" -o -iname "*.jif" -o -iname "*.jfif" -o -iname "*.jfi" \) -print0 |
    xargs --null --no-run-if-empty jhead -autorot'
alias rsync-quick='rsync -ahi'
alias rsync-verify='rsync -ahic'
alias rsync-vfat-quick='rsync -rhit --modify-window=1'
alias rsync-vfat-verify='rsync -rhitc --modify-window=1'
alias show-secrets='decrypt-file-to-less ~/Documents/Secrets.txt.aes256'
alias slideshow-all-monitor='feh --auto-zoom --hide-pointer --randomize --recursive --slideshow-delay=10 --draw-filename --fullscreen ~/Pictures/.'
alias slideshow-all-tv='feh --auto-zoom --hide-pointer --randomize --recursive --slideshow-delay=10 --draw-filename --borderless --image-bg=black --geometry=1200x670+1960+25 ~/Pictures/.'
alias slideshow-favorites-monitor='feh --auto-zoom --hide-pointer --randomize --recursive --slideshow-delay=10 --draw-filename --fullscreen ~/Pictures/Favorites/.'
alias slideshow-favorites-tv='feh --auto-zoom --hide-pointer --randomize --recursive --slideshow-delay=10 --draw-filename --borderless --image-bg=black --geometry=1200x670+1960+25 ~/Pictures/Favorites/.'
alias smbclient-joli='smbclient -U Mark \\\\192.168.0.4\\shared'
alias sox='sox -V --no-clobber'
alias ssh='ssh -Y'
alias umount-iso='sudo umount ~/Mount/.'
alias utf16-cat='iconv -f UTF-16'
alias where='type -a'
alias wodim='wodim -v'
alias zless='zless -iM'

### Shell functions ###

decrypt-file-to-file()
{
    [[ $# -eq 2 ]] || return $?
    [[ -f "${1}" ]] || return $?
    [[ ! -e "${2}" ]] || return $?

    openssl enc -aes256 -d -in "${1}" -out "${2}" || return $?
}

decrypt-file-in-place()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "${1}" ]] || return $?

    local directory="$(dirname "${1}")"
    [[ -n "${directory}" ]] || return $?

    local base="$(basename "${1}" .aes256)"
    [[ -n "${base}" ]] || return $?

    decrypt-file-to-file "${1}" "${directory}/${base}" || return $?
}

decrypt-file-to-ramdisk()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "${1}" ]] || return $?

    local ramdisk=~/RamDisk/.
    [[ -d "${ramdisk}" ]] || return $?

    local base="$(basename "${1}" .aes256)"
    [[ -n "${base}" ]] || return $?

    decrypt-file-to-file "${1}" "${ramdisk}/${base}" || return $?
}

decrypt-file-to-stdout()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "$1" ]] || return $?

    openssl enc -aes256 -d -in "${1}" || return $?
}

decrypt-file-with-md5-digest-to-stdout()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "$1" ]] || return $?

    openssl enc -aes256 -md md5 -d -in "${1}" || return $?
}

decrypt-file-to-less()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "$1" ]] || return $?

    local p
    read -s -p "Password: " p || return $?
    builtin echo "${p}" | openssl enc -aes256 -d -pass stdin -in "${1}" | less
}

decrypt-and-untar-file-to-ramdisk()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "${1}" ]] || return $?

    local ramdisk=~/RamDisk/.
    [[ -d "${ramdisk}" ]] || return $?

    local p
    read -s -p "Password: " p || return $?
    builtin echo "${p}" | openssl enc -aes256 -d -pass stdin -in "${1}" | tar -C "$ramdisk" -xv || return $?
}

encrypt-file-in-place()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "$1" ]] || return $?

    local destination="$1.aes256"
    [[ ! -e "${destination}" ]] || return $?

    openssl enc -aes256 -in "${1}" -out "${destination}" || return $?
}

decrypt-file-for-edit()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "${1}" ]] || return $?

    local ramdisk=~/RamDisk/.
    [[ -d "${ramdisk}" ]] || return $?

    local base="$(basename "${1}" .aes256)"
    [[ -n "${base}" ]] || return $?

    decrypt-file-to-file "${1}" "${ramdisk}/${base}" || return $?

    geany --new-instance -- "${ramdisk}/${base}" || return $?

    encrypt-file-in-place "${ramdisk}/${base}" || return $?
    [[ -f "${ramdisk}/${base}.aes256" ]] || return $?
    rm --verbose -- "${ramdisk}/${base}" || return $?
    mv --interactive --verbose -- "${ramdisk}/${base}.aes256" "${1}" || return $?
}

mount-iso()
{
    [[ $# -eq 1 ]] || return $?
    [[ -f "$1" ]] || return $?

    sudo mount -o loop,ro "$1" ~/Mount/. || return $?
}

openssl-gen-key-pair()
{
    [[ ! -e 'private.pem' ]] || return $?
    [[ ! -e 'public.pem' ]] || return $?
    openssl genrsa -aes256 -out private.pem 2048 || return $?
    openssl rsa -in private.pem -outform PEM -pubout -out public.pem || return $?
}

openssl-sign()
{
    [[ $# -eq 2 ]] || return $?
    [[ -f "$1" ]] || return $?
    [[ -f "$2" ]] || return $?
    [[ ! -e "${1}.sha256" ]] || return $?
    openssl dgst -sha256 -sign "$2" -out "${1}.sha256" "$1" || return $?
}

openssl-verify-signature()
{
    [[ $# -eq 2 ]] || return $?
    [[ -f "$1" ]] || return $?
    [[ -f "$2" ]] || return $?
    [[ -f "${1}.sha256" ]] || return $?
    openssl dgst -sha256 -verify "$2" -signature "${1}.sha256" "$1" || return $?
}

play-recording-in-progress()
{
    [[ $# -eq 1 ]] || {
        echo >&2 "ERROR: Expected 1 argument(s), received $#."
        return 1
    }
    [[ -f "$1" ]] || {
        echo >&2 "ERROR: '$1' is not a file."
        return 1
    }

    cat "$1" | play -t mp3 - channels 1
}

split-file()
{
    [[ $# -ge 1 && $# -le 2 ]] || {
        echo >&2 "ERROR: Expected 1-2 argument(s), received $#."
        return 1
    }

    local cmd="split --verbose"
    [[ -z "$2" ]] || cmd="${cmd} -b $2"
    cmd="${cmd} \"$1\" \"$1.\""
    eval ${cmd}
}

unzip-to-ramdisk()
{
    [[ $# -eq 1 ]] || {
        echo >&2 "ERROR: Expected 1 argument(s), received $#."
        return 1
    }
    [[ -f "$1" ]] || {
        echo >&2 "ERROR: '$1' is not a file."
        return 1
    }

    unzip "$1" -d ~/RamDisk/. || {
        echo >&2 "ERROR: Failed to extract ($?)."
        return 1
    }
}

### Non-persistent settings ###

set -P
set -o pipefail
umask 0077

# Choose a colour so that the filename part of grep output is more readable.
export GREP_COLORS="fn=33"

PS1_ORIGINAL="${PS1_ORIGINAL:-${PS1}}"
PS1_ORIGINAL_MINUS_SUFFIX=${PS1_ORIGINAL%\\$ }
PS1="--\n\$(RET=\$?
    [[ \$RET -eq 0 ]] || echo -n \"\[\e[1;31m\]\"
    echo -n \"\$RET\") \t\[\e[0;m\]\n\$(
    [[ \$(id --user) -ne 0 ]] ||
    echo -n \"\[\e[1;31m\]\")${PS1_ORIGINAL_MINUS_SUFFIX}\[\e[0;m\]\n$ "

case ":${PATH:=${HOME}/Backup/bin}:" in
    *":${HOME}/Backup/bin:"*) ;;
    *) PATH="${PATH}:${HOME}/Backup/bin" ;;
esac

mkdir --parents --verbose "/dev/shm/${USER}"
mkdir --parents --verbose "/tmp/${USER}"

# https://github.com/magicmonty/bash-git-prompt
GIT_PROMPT_ONLY_IN_REPO=1
source ~/.bash-git-prompt/gitprompt.sh
 ```
