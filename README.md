Dotfiles Template
=================


Prerequisites
-------
1) You must to have installed brew:
    https://brew.sh/
    
2) Install git:
```sh
brew install git
```


Summary
-------

This is a template repository for bootstrapping your dotfiles with [Dotbot][dotbot].

In general, you should be using symbolic links for everything, and using git
submodules whenever possible.

To keep submodules at their proper versions, you could include something like
`git submodule update --init --recursive` in your `install.conf.yaml`.

To upgrade your submodules to their latest versions, you could periodically run
`git submodule update --init --remote`.

To start the installation, you only need to execute:
```sh
./install
```

License
-------

This software is hereby released into the public domain. That means you can do
whatever you want with it without restriction. See `LICENSE.md` for details.
