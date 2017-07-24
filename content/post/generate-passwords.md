---
title: "Generate Passwords"
date: 2017-07-23T14:24:57+02:00
description: Quick way to generate passwords on the command line
tags: [Admin]
---

## Generate an alpha numeric password

```console
$ pwgen -s 20 1                                           # => 66jTuJXUYOrnqoOpfOWE
$ </dev/urandom tr -dc [:alnum:] | head -c20 && echo      # => Zg0OFvg0wOEc4hdEwawZ
```
## Generate an numeric only password

```console
$ </dev/urandom tr -dc [:digit:] | head -c20 && echo      # => 91331194034454954513
$ </dev/urandom tr -dc 0-9 | head -c20 && echo            # => 00166785542509259371
```

## Generate an alpha only password

```console
$ pwgen -s0 20 1                                          # => JqNGGWcaJyamSAyXkqnG
$ </dev/urandom tr -dc a-zA-Z | head -c20 && echo         # => PAnRPpXuPuYNrNpGtKrN
$ </dev/urandom tr -dc [:alpha:] | head -c20 && echo      # => QlSBFAAmqTXjJLDyBZoN
```
