---
title: Linux package management
---

# Standard tasks

## Install a package

```bash
sudo apt install <package_name>
```

## Install all available updates

```bash
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get autoremove
sudo apt-get autoclean
```

## Install a package from a deb file

```bash
sudo dpkg -i <path_to_deb_file>
```

# Less frequent tasks

## List installed packages

```bash
dpkg --get-selections
```

With versions:

```bash
apt-show-versions # Requires apt-show-versions package
dpkg-query --show "linux-image*"
```

## List available packages

```bash
apt-cache pkgnames
```

## Determine which package a file belongs to:

```bash
apt-file search <path-to-file> # Requires apt-file package
```

## See configured options

```bash
apt-config dump
```

## Re-configure a package

```bash
dpkg-reconfigure <package>
```
