+++
title = "Bitwarden encrypted backup tool"
description = "Announcing a small utility for encrypted backups of Bitwarden vaults"
path = "/blog/2021/08/bitwarden-encrypted-backup-tool"
date = 2021-08-13
+++
I've been using [Bitwarden](https://bitwarden.com/) as my password manager for years. At this point, my whole family is on it, as well as [my company](https://www.fpcomplete.com/). Suffice it to say that if I ever lost access to that vault, I would be in a lot of trouble. While I overall really like Bitwarden, one limitation is has is a lack of an encrypted export function.

A few weeks ago, I decided to write a tool to automate the process of exporting a Bitwarden vault to a JSON file and encrypting it. And today I decided to [release this tool as open source](https://github.com/snoyberg/bwbackup). Hopefully it will be useful to someone else.

Below is the `README.md`, which contains relevant information on how this tool works internally and how to get started.

* * *

## bwbackup

[![Rust](https://github.com/snoyberg/bwbackup/actions/workflows/rust.yml/badge.svg)](https://github.com/snoyberg/bwbackup/actions/workflows/rust.yml)

Create encrypted backups of your Bitwarden vault.

This is a simple tool, intended to solve a simple problem. When you use the Bitwarden CLI, you're able to extract a CSV or JSON encoded copy of your entire vault. Unfortunately, this copy isn't encrypted. The purpose of this tool is to run the relevant commands against the `bw` executable and encrypt the data before it ever touches your hard drive.

Important notes:

* If you have MFA enabled, you'll likely need to run `bw login` at least once before running this executable to provide the MFA token.
* If you log in with multiple different Bitwarden accounts, running this tool will switch which one you're logged in with.
* The `bw unlock` and `bw export` commands (at time of writing) unfortunately does not allow specifying the password via environment variables. Therefore, the password will be passed as a command line argument, which is less secure. It's possible that other processes on your system may be able to see that password. Caveat emptor!
* The file is encrypted using a [`sodiumoxide`](https://lib.rs/crates/sodiumoxide) `secretbox`, using your master password for key generation.
* This tool will produce a new salt and nonce and each invocation, meaning even if your vault and password remain unchanged, you will get different encrypted output on each invocation.

This tool is lightly tested, but I've been using it myself and have had no issues. Hopefully others will find it useful too!

## Installing

You can check the [GitHub Actions](https://github.com/snoyberg/bwbackup) for recent artifacts, or can build it yourself by [installing Rust](https://www.rust-lang.org/tools/install) and running `cargo install --git https://github.com/snoyberg/bwbackup`.

## Backup procedure

I use the following wrapper shell script to backup to my `~/dotfiles` repo and create a new commit once a week:

```shell
#!/usr/bin/env bash

set -euxo pipefail

cd ~/dotfiles
bwbackup backup --email MYEMAILADDRESS --file bwbackup.json.enc
git add bwbackup.json.enc
git commit -m "BitWarden backup at $(date)"
git push
```
