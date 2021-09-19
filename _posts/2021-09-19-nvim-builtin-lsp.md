---
title: "Configuring auto-completion in Neovim using the built-in LSP"
layout: single
published: true
---

The concept of the language server protocol is so simple, yet so brilliant -
it's truly elegant. Apart from the elegance of the concept, it's great because
it brings a tonne of great features to vim (and all other editors that support
the language server protocol). It's pretty straight-forward to set up, but
since it's not quite as simple as installing a plugin in, say, VS Code, I'll
share the most basic config mixed with some background and brief intro to
what's needed to get auto-completion up and running in (Neo)vim.

Furthermore, while built-in LSP support is new in Neovim, auto-completion
isn't. There's the omnifunc functionality and also more powerful options such
as coc.nvim and YouCompleteMe. While I've happily used them in the past, it
just feels better to depend on native features. That said, I haven't actually
experienced any issues with neither alternative, so this isn't going to be a
post comparing the different options, but it's more of an aggregation of
information on how to get a decent setup up and running with the native LSP
support.


# Prerequisites

 - Neovim >= 0.5


# The basics

LSP is _not_ auto-completion. If you're coming from another editor, perhaps
VS Code or even an IDE, you may be used to installing a single plugin for your
language and then everything magically works. The debugger just works, jumping
to definitions works out of the box, fuzzy completion by default, and the
suggestions pop up automatically, you get the point. Sure, it's convenient and
as long as it's working good enough. The issue is when it isn't working. Ever
developed C++ in VS Code? For the most part, it's quite decent, but there's one
issue - the memory footprint is huuuge. It'll eat your RAM for breakfast.


## The language server protocol

The purpose of the language server protocol is to separate the language details
from the actual editor. Instead, language servers and editors or IDEs can be
developed separately.  

Historically, if you had `M` editors that each had to support `N` different
languages, you'd need `MxN` different implementations (or integrations) to
support all `N` languages in all editors. As the below image illustrates, the
language server protocol takes an `MxN` problem and makes it an `M+N` problem
instead. Undoubtedly, this is a great invention for editor developers, but what
makes it so awesome from a user's perspective is obviously that all existing
LSP-based language servers will be ready to use if your editor features LSP
support.


```text
Editors/IDEs                                     Language servers  

vim     -------------                    ------- rust\_analyzer  
                     \                  /  
VS Code     -----     \                /     --- pyright  
                 \     \              /     /  
Visual Studio    ->----->----- LSP ------------- ccls  
                 /     /              \     \  
Sublime     -----     /                \     --- gopls  
                     /                  \  
Eclipse -------------                    ------- vimls  
```

(Disclaimer: I don't actually know if all these support or use LSP, but that's
irrelevant for the point being made here.)


## Completion

This is just a small part of what a language server actually provides. Other
useful features are refactoring, diagnostics, go-to-definition,
go-to-declaration, go-to-implementation, etc. It's not like the built-in `gD`
type of go-to-global-declaration in vim, but it includes functionality to find
project-wide declarations and definitions through semantic analysis and a lot
more.

Another setting you can configure is how the completions are filtered. For
instance, you can specify that you want the completion functionality to return
exact matches, sub-string matches, fuzzy matches or all, or all of these in the
desired order.


## Auto-completion

What may come as a surprise to some people is that the auto-completion will not
happen by itself. If you install a language server and configure the built-in
LSP client accordingly, you still won't get any completion suggestions
automatically. That's a good thing, 'cause auto-completion is expensive and
while you may not want to pay the price for auto-completion, you may still want
the accurate suggestions provided by the language server(s).


# Plugins

I'm using two plugins to get the most basic auto-completion functionality:
 - [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)
 - [completion-nvim](https://github.com/nvim-lua/completion-nvim)
    - I've been meaning to try [nvim-cmp](https://github.com/hrsh7th/nvim-cmp),
      too, but haven't gotten to it yet.

## nvim-lspconfig

This is basically a bunch of configurations and convenience functions to be
used with the built-in LSP. The project's readme sums it up quite well:

> A collection of common configurations for Neovim's built-in language server
> client.  
>  
> This repo handles automatically launching and initializing language servers
> that are installed on your system.

You'll have to install the actual language servers yourself, but apart from
that, it's pretty much as smooth as it can get.

With this plugin, configuring, auto-starting and using a language server is as
simple as adding the following to your `.vimrc` (in the case of `ccls`)

```lua
lua << EOF
require'lspconfig'.ccls.setup{}
EOF
```

Smooth, eh?


## completion-nvim

This is what provides the actual auto-completion with pop-ups and all. It's as
easy as

```lua
lua << EOF
lua require'lspconfig'.ccls.setup{on_attach=require'completion'.on_attach}
EOF
```

That will set up the auto-completion when attaching to a buffer. It does what
it's supposed to, but you'll probably want to tweak the on\_attach callback
slightly, as we'll see in the following section.


## The config

The following is an aggregation of information collected from the documentation
of the two plugins with a few modifications. The most important part is the
`on_attach` callback.

```lua
local nvim_lsp = require('lspconfig')
local util = require('lspconfig/util')

local on_attach = function(client, bufnr)
    require'completion'.on_attach(client, bufnr)

    local function buf_set_keymap(...) vim.api.nvim_buf_set_keymap(bufnr, ...) end
    local function buf_set_option(...) vim.api.nvim_buf_set_option(bufnr, ...) end

    -- Mappings.
    local opts = { noremap=true, silent=true }

    -- See `:help vim.lsp.*` for documentation on any of the below functions
    buf_set_keymap('n', 'gD', '<cmd>lua vim.lsp.buf.declaration()<CR>', opts)
    buf_set_keymap('n', 'gd', '<cmd>lua vim.lsp.buf.definition()<CR>', opts)
    buf_set_keymap('n', 'K', '<cmd>lua vim.lsp.buf.hover()<CR>', opts)
    buf_set_keymap('n', 'gi', '<cmd>lua vim.lsp.buf.implementation()<CR>', opts)
    buf_set_keymap('n', '<space>D', '<cmd>lua vim.lsp.buf.type_definition()<CR>', opts)
    buf_set_keymap('n', '<space>rn', '<cmd>lua vim.lsp.buf.rename()<CR>', opts)
    buf_set_keymap('n', '<space>ca', '<cmd>lua vim.lsp.buf.code_action()<CR>', opts)
    buf_set_keymap('n', 'gr', '<cmd>lua vim.lsp.buf.references()<CR>', opts)
    buf_set_keymap('n', '<space>e', '<cmd>lua vim.lsp.diagnostic.show_line_diagnostics()<CR>', opts)
    buf_set_keymap('n', '[d', '<cmd>lua vim.lsp.diagnostic.goto_prev()<CR>', opts)
    buf_set_keymap('n', ']d', '<cmd>lua vim.lsp.diagnostic.goto_next()<CR>', opts)
    buf_set_keymap('n', '<space>q', '<cmd>lua vim.lsp.diagnostic.set_loclist()<CR>', opts)
    buf_set_keymap('n', '<space>f', '<cmd>lua vim.lsp.buf.formatting()<CR>', opts)
end

local servers = {
    pyright = {
        settings = {
            python = {
                analysis = {
                    typeCheckingMode = "off"  -- you may want to remove this, but I found it annoying
                    }
                }
            },
        root_dir = function(fname)
            local root_files = {
                'pyproject.toml',
                'setup.py',
                'setup.cfg',
                'requirements.txt',
                'Pipfile',
                'pyrightconfig.json',
                'venv',
                }
            return util.root_pattern(unpack(root_files))(fname) or util.find_git_ancestor(fname) or util.path.dirname(fname)
        end
    },
    ccls = {
    }
}

for server_name, configuration in pairs(servers) do
    configuration.on_attach = on_attach
    nvim_lsp[server_name].setup(configuration)
end
```

**NOTE**: Don't forget to wrap between `lua << EOF` and `EOF` if you put it in your
`.vimrc`.

That's it. Slightly more verbose than setting up a plugin in VS Code, but not
that bad.

