+++
path = "/2016/virtualenv/"
title = "Self-made Virtualenv Wrapper"
date = 2016-12-28
[extra]
modified = 2016-12-28T08:27:00+00:00
+++
# Self-made Virtualenv Wrapper


Not long ago I've switched from [virtualenvwrapper][1] to self-made solution
based on [this idea][2] by [datagrok][4].
It works pretty well for me and I'd like to share it here.

The idea is to have a _root_ directory (`$HOME/_dev` in my case)
for all projects and to have shell helper to easily jump into any project
and to automatically activate it's [virtualenv][3] if it exists.

Here is how it looks in `bash`:

```bash

[local] [egorov@asus 11:33:19]:~  $ dev blog
Virtual environment found in /home/egorov/_dev/blog/env, activated.
(blog) [local] [egorov@asus 11:34:31]:~/_dev/blog git:master= $

```

Besides just jumping into directory I wanted to have `tab` autocompletion and
easy way to create new projects `home` directory:

```bash

# autocompletion
[local] [egorov@asus 11:43:01]:~  $ dev service-  # tab hitted twice
service-account   service-feedback  service-products  service-workers

# new project home
[local] [egorov@asus 11:52:08]:~  $ dev rogue
mkdir: created directory '/home/egorov/_dev/rogue'
No virtual environment found, skipping activate.
[local] [egorov@asus 11:52:25]:~/_dev/rogue  $

```

So, here are `bash` helpers which should be placed somewhere inside `~/.bashrc`
or `~/.bash_profile` (mine lives [there][5]).
`_dev` function depends on `inve` shell script which must be available
in `PATH` (I have `$HOME/bin` in `PATH` for this and `inve` lives [there][6]).


```bash

# dev
DEV=${HOME}/_dev  # home for all projects

_dev() {
    local p=${DEV}/$1
    [ ! -d $p ] && mkdir $p
    cd $p
    inve  # binary file to be accessible in PATH
    return 0
}
_dev_complete()
{
    local cur prev opts
    local p=${DEV}
    [ ! -d $p ] && `which mkdir` $p
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    opts=`cd $p && find -L . -maxdepth 1 -type d | sed 's|[\./]||g'`

    COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
}
alias dev=_dev
complete -F _dev_complete dev

```

`inve` executable content:

```bash

#!/usr/bin/env bash

# tuned idea from: https://gist.github.com/datagrok/2199506
#
# inve
#
# For use with Ian Bicking's virtualenv tool. Attempts to find the root of
# a virtual environment and activate it in subshell.

# First, locate the root of the current virtualenv
cwd=$PWD
export DEV_ROOT=$cwd

while [ "$PWD" != "/" ]; do
    # Stop here if this the root of a virtualenv
    # will look for virtualenv in `current dir` or in `env` subdirectory
    if [ \
        -x bin/python \
        -a -e lib/python*/site.py \
        -a -e include/python*/Python.h ]
    then
        export VIRTUAL_ENV="$PWD"
        break
    elif [ \
        -x env/bin/python \
        -a -e env/lib/python*/site.py \
        -a -e env/include/python*/Python.h ]
    then
        export VIRTUAL_ENV="$PWD/env"
        break
    fi
    cd ..
done
if [ "$PWD" = "/" ]; then
    export PATH="$cwd/node_modules/.bin:$PATH"
    echo "No virtual environment found, skipping activate." >&2
    cd $cwd
else
    # Activate
    export PATH="$VIRTUAL_ENV/bin:$cwd/node_modules/.bin:$PATH"
    unset PYTHONHOME
    echo "Virtual environment found in ${VIRTUAL_ENV}, activated." >&2
fi
exec "$SHELL"

```

`inve` script looks for `virtualenv` in `env` subdirectory or attempts to
find it somewhere upper in the filesystem tree. It modifies `PATH` prepending
`$VIRTUALENV/bin` and `node_modules/.bin` directories and starts new shell.
I like the idea of starting a new subshell in order to activate `virtualenv` as
this way we can be sure there will be no problems with `PATH` within shell and
it's pretty safe to exit this subshell and have parent shell clean as it should
be.

To easily navigate to some directories within activated `virtualenv` I have a
couple of bash aliases defined:

```bash

alias cdsrc='cd ${VIRTUAL_ENV}/src'
alias cdsitepackages='cd ${VIRTUAL_ENV}/lib/python*/site-packages/'
alias cdproject='cd ${DEV_ROOT:-VIRTUAL_ENV}; if [ `basename ${PWD}` = "env" ]; then cd ..; fi'

```

This approach allows me to have virtual environments using `python2` or
`python3` under single _root_ directory and to have single tool to jump into
project be it `python2`, `python3` or even `nodejs` project.

Everything works pretty good for me.


[1]: https://pypi.python.org/pypi/virtualenvwrapper
[2]: https://gist.github.com/datagrok/2199506
[3]: https://github.com/pypa/virtualenv
[4]: http://datagrok.org/
[5]: https://github.com/ysegorov/dotfiles/blob/master/.bash.d/dev
[6]: https://github.com/ysegorov/dotfiles/blob/master/bin/inve
