---
title: Managing Multiple Computers with One bashrc/zshrc
kind: article
created_at: 2013-05-06
category: quick-guide
---

<!-- _. -->

Here is a simple way to share the same `.bashrc` / `.zshrc` / `.bash_profile`
across multiple computers, while still retaining unique settings in between
computers.

Suppose you want some special setting to apply only to your laptop.

First, create an empty file called `.setup_00`:

    $ touch ~/.setup_00

Next, in your `rc` file, add the following `if` statement. Anything
inside that `if` statement will only apply to your laptop:

    if [ -f '.setup_00' ]; then
        echo "This message only shows on my laptop!"
    fi

You can use this method to run any shell script uniquely on computers
that contain the `.setup_00` file.

That's it. It's not fancy, but it works.
