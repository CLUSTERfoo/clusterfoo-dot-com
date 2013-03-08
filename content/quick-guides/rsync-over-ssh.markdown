---
title: Syncing with a Remote Server Using Rsync Over SSH
kind: article
created_at: 2013-03-08
---
<!-- _. -->

Rsync is a command-line utility for copying and syncing files. Rsync can be 
used over ssh to sync files between remote servers. The basic command
for use over ssh is

    $ rsync [options] -e "ssh" source target

If you've set up private and public keys to secure your ssh connection, you must 
specify the location of the identity key (also known as "private key"), which 
is usually found in the `~/.ssh` directory:

    $ rsync [options] -e "ssh -i /home/user/.ssh/private_key_file"

I use rsync to deploy my blog. All of its contents are found [in the `output`
directory](http://github.com/clusterfoo/clusterfoo-dot-com), which I sync with
a small shared server at [A Small Ornage](http://asmallorange.com). 

I use the following simple command:

~~~
$ rsync -avh [-n] --progress --delete-after -e "ssh -i /home/username/.ssh/id_rsa" \
  output/ username@clusterfoo.com:/home/username/public_html/
~~~

* `-a`: Archive mode. The most common option for syncing directories.
* `-v`: Verbose mode.
* `-h`: Human readable. Print numbers in human-readable formats.
* `-n`: Dry run.
* `--progress`: Show progress during transfer.
* `--delete-after`: Sync new and updated files, and wait until transfer is 
complete before deleting any superfluous files. This is important when deploying
a website, since otherwise a visitor might stumble upon missing pages.

### Always Dry-Run First!

It's easy to make mistakes with rsync. A simple typo like `dir` instead of 
`dir/` can mean a world of difference (and a world of pain if you accidentally 
`-rm` an entire directory!).

I always do a dry-run before I use an rsync command for the first time. The 
dry-run option (`-n`) will show you the final output of the operation, 
without actually running through it.

