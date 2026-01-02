# Git submodules

## Adding a submodule

If you want to add a git submodule to an existing repository:
```console
# <URL> should be a git url that anyone could use
git submodule add <URL>
```

The resulting files can be committed separately through a PR.

If you, as a developer of the submodule, would like to also commit
into it you should:
```console
# <PRIVATE_URL> this could be for example the ssh version of the <URL>
git config submodule.DbConnector.url <PRIVATE_URL>
```

## Cloning a project with a submodule

Cloning is done normal.
```console
git clone <URL>
```

Next you will need to run:
```console
git submodule init
```
in order to initialize the local configuration, and:
```console
git submodule update
```
to update all the data and checkout the appropriate commit.

These two previous commands can be combined into one:
```console
git submodule update --init
```

Further more, if the submodules have submodules themselves:
```console
git submodule update --init --recursive
```

All the previous commands in this section can be done in one go:
```console
git clone --recurse-submodules <URL>
```

## Working with a submodule

If you want to get updates from a submodule:
```console
git submodule update --remote
```

Making changes in the submodules is the same as with any other git project.
Checkout a branch and start working on it.

If you want to see the changes that you made, you can do:
```console
git diff --submodule
```

To push the changes, go in the submodule directory and do a `git push`.

If you want to do this push from the main project do:
```console
git push --recurse-submodules=on-demand
```

In order to not specify the recurse-submodules option:
```
git config --global push.recurseSubmodules on-demand
```

If you don't want to specify `--submodule` every time, do:
```console
git config --global diff.submodule log
```
