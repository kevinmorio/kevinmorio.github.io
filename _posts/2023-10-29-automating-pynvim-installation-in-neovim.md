---
layout: post
title: "Automating pynvim Installation in Neovim"
date: 2023-10-29 21:00:32
categories: posts
---

Using Python plugins in [Neovim](https://neovim.io) requires the `pynvim` module.
To ensure Neovim can locate the module, even within a virtual environment (`$VIRTUAL_ENV` is set), the path to the Python binary must be specified via `python3_host_prog`.
This avoids the need to install `pynvim` separately for each venv.
To keep things organized, we create a dedicated venv for Neovim and install the required module.

We can automate this process at startup using Vimscript:

``` vimscript
" Create Python 3 venv and install pynvim if it is not already installed.
if empty(glob($XDG_DATA_HOME.'/nvim/venv'))
    execute '!python3 -m venv ' . $XDG_DATA_HOME . '/nvim/venv'
    execute '!' . $XDG_DATA_HOME . '/nvim/venv/bin/python3 -m pip install pynvim'
endif
let g:python3_host_prog=$XDG_DATA_HOME . '/nvim/venv/bin/python3'
```

Alternatively, Lua can be used:

``` lua
-- Create Python 3 venv and install pynvim if it is not already installed.
if vim.fn.empty(vim.fn.glob(vim.fn.stdpath('data') .. '/venv')) == 1 then
    vim.fn.system({
        'python3', '-m', 'venv',
        vim.fn.stdpath('data') .. '/venv'
    })
    vim.fn.system({
        vim.fn.stdpath('data') .. '/venv/bin/python3',
        '-m', 'pip', 'install', 'pynvim'
    })
end
vim.g.python3_host_prog = vim.fn.stdpath('data') .. '/venv/bin/python3'
```


If the venv does not already exist, we create it and install `pynvim`.
We then set `python3_host_prog` to the Python 3 binary within the venv.

## References

- [Python integration - Neovim docs](https://neovim.io/doc/user/provider.html#provider-python)
