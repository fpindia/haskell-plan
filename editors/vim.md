# Getting Started

For instructions on installation of vim or [neovim](https://neovim.io/doc/)
please see the applicable documentation for the editor.

As a configurable text editor, there are infinite number of possibilities in
how one may configure vim. We will cover a personal opinionated way to
configure it in this documentation. Also, from here on whenever vim is
referred it means neovim, but, except for certain cases most plugins work in
both. We make the assumption that the reader knows how to use plugin manager,
setup custom vimrc, key bindings and such. It will help if one is aware of
setting up key bindings and auto command groups.

# Compulsory Reading

[An opinionated guide to Haskell in 2018](https://lexi-lambda.github.io/blog/2018/02/10/an-opinionated-guide-to-haskell-in-2018/)

# Syntax Highlighting

[vim-polyglot](https://github.com/sheerun/vim-polyglot) is a good language
pack to have. It adds support for multiple languages but one can also install
for only the selected languages if one so desires.

# Code Navigation

## Fuzzy Finder

Fuzzy finder is a must and provides a lot of productivity boost when
integrated in vim. One of the fuzzy finders is
[fzf](https://github.com/junegunn/fzf) and has excellent integration using
[fzf.vim](https://github.com/junegunn/fzf.vim). Combined with
[ripgrep](https://github.com/BurntSushi/ripgrep) one can have extremely fast
project wide search which is language and project agnostic.

For example, the below binding

```
nnoremap <Leader>fs  :exe ':Rg ' . expand('<cword>')<CR>
```

searches for the word under cursor on hitting the leader key + fs and uses
ripgrep.

## Tags

Tags based navigation is reliable and can work out of the box in vim. All that
is required is to generate the tags. For a `stack` project, this can be done
as below.

```
stack exec -- hasktags -x -c .
```

`hasktags` should have been build and installed using the `copy-compiler-tool`
flag. See the compulsory reading section.

This can be bound to a shortcut like below.

```
au FileType haskell nmap <LocalLeader>t  :NeomakeSh stack exec -- hasktags -x -c .<CR>
```

[neomake](https://github.com/neomake/neomake) provides capability to execute a
command asynchronously. This is what we are relying on in the above.

## Language Server Protocol

There are various LSP clients available. Two of the popular ones are
[LanguageClient-neovim](https://github.com/autozimu/LanguageClient-neovim) and
[vim-lsp](https://github.com/prabirshrestha/vim-lsp). The first one is written
in Rust and second one in pure vimscript. README for both projects is
extensive and detailed. An example can be seen
[here](https://gitlab.com/SanchayanMaity/dotfiles/-/blob/master/nvim/.config/nvim/init.vim).

The specific parts are reproduced here for convenience. Note the `ghcide`
entry which is a LSP server for Haskell.

```
augroup LSP
  au!
  nnoremap <LocalLeader>le :call lsp#enable()<CR>
  nnoremap <LocalLeader>ld :call lsp#disable()<CR>
  nnoremap <LocalLeader>lh :LspStopServer()<CR>
  nnoremap <LocalLeader>ls :LspStatus<CR>
  au User lsp_server_init call s:on_lsp_server_init()
augroup END

augroup VimLspSettings
  au!
    if executable('pyls')
        au User lsp_setup call lsp#register_server({
            \ 'name': 'pyls',
            \ 'cmd': {server_info->['pyls']},
            \ 'whitelist': ['python'],
            \ })
    endif
    if executable('ra_lsp_server')
        au User lsp_setup call lsp#register_server({
            \ 'name': 'ra_lsp_server',
            \ 'cmd': {server_info->['ra_lsp_server']},
            \ 'whitelist': ['rust'],
            \ })
    endif
    if executable('purescript-language-server')
        au User lsp_setup call lsp#register_server({
            \ 'name': 'purescript-language-server',
            \ 'cmd': {server_info->['purescript-language-server', '--stdio']},
            \ 'whitelist': ['purescript'],
            \ })
    endif
    if executable('ghcide')
        au User lsp_setup call lsp#register_server({
            \ 'name': 'ghcide',
            \ 'cmd': {server_info->['ghcide', '--lsp']},
            \ 'whitelist': ['haskell'],
            \ })
    endif
augroup END

function! s:on_lsp_server_init() abort
  setlocal omnifunc=lsp#complete
  setlocal signcolumn=yes

  " Always expected to work
  nmap <buffer> gd  :LspDefinition<CR>
  nmap <buffer> gpd :LspPeekDefinition<CR>
  nmap <buffer> gk  :LspHover<CR>

  " May or may not be available depending on language server
  nmap <buffer> gx :LspReferences<CR>
  nmap <buffer> gr :LspRename<CR>
  nmap <buffer> <LocalLeader>ca :LspCodeAction<CR>
  nmap <buffer> <LocalLeader>dd :LspDocumentDiagnostics<CR>
  nmap <buffer> <LocalLeader>df :LspDocumentFormat<CR>
  nmap <buffer> <LocalLeader>rf :LspDocumentRangeFormat<CR>
  nmap <buffer> <LocalLeader>ds :LspDocumentSymbol<CR>
  nmap <buffer> <LocalLeader>di :LspImplementation<CR>
  nmap <buffer> <LocalLeader>nd :LspNextDiagnostic<CR>
  nmap <buffer> <LocalLeader>pd :LspPreviousDiagnostic<CR>
  nmap <buffer> <LocalLeader>ne :LspNextError<CR>
  nmap <buffer> <LocalLeader>pe :LspPreviousError<CR>
  nmap <buffer> <LocalLeader>nr :LspNextReference<CR>
  nmap <buffer> <LocalLeader>pr :LspPreviousReference<CR>
  nmap <buffer> <LocalLeader>nw :LspNextWarning<CR>
  nmap <buffer> <LocalLeader>pw :LspPreviousWarning<CR>
  nmap <buffer> <LocalLeader>dt :LspTypeDefinition<CR>
  nmap <buffer> <LocalLeader>td :LspPeekTypeDefinition<CR>
  nmap <buffer> <LocalLeader>pi :LspPeekImplementation<CR>
endfunction
```

Note that the configuration of an LSP client is also dependent on the
selection of a linter and asynchronous completion engine. This is covered
in the auto completion section below.

## Any Jump

[any-jump](https://github.com/pechorin/any-jump.vim) provides support for a
lot of languages and uses regular expressions in the backend. It is similar to
the [dumb-jump](https://github.com/jacktasia/dumb-jump) package in emacs.

# Auto Completion

[asyncomplete](https://github.com/prabirshrestha/asyncomplete.vim) is one of
the popular completion engines. This can provide auto completion for file
paths, buffer, LSP based, tmux etc. This needs to be paired with the
corresponding source provider. So for example, to have completion suggestions
for words from the opened buffer, one would have to add asyncomplete-buffer.
Also, the base plugin does not enable anything automatically and sources must
be enabled explicitly. See the README. An example is below.

```
augroup AsyncCompleteSetup
  au!
    au User asyncomplete_setup call asyncomplete#register_source(asyncomplete#sources#tags#get_source_options({
      \ 'name': 'tags',
      \ 'whitelist': ['c', 'haskell'],
      \ 'completor': function('asyncomplete#sources#tags#completor'),
      \ }))
   au User asyncomplete_setup call asyncomplete#register_source(asyncomplete#sources#buffer#get_source_options({
      \ 'name': 'buffer',
      \ 'whitelist': ['*'],
      \ 'blacklist': ['go'],
      \ 'completor': function('asyncomplete#sources#buffer#completor'),
      \ 'config': {
      \    'max_buffer_size': 5000000,
      \  },
      \ }))
   au User asyncomplete_setup call asyncomplete#register_source(asyncomplete#sources#file#get_source_options({
      \ 'name': 'file',
      \ 'whitelist': ['*'],
      \ 'priority': 10,
      \ 'completor': function('asyncomplete#sources#file#completor')
      \ }))
augroup END
```

Asynchronous completion from the LSP client is provided by
[asyncomplete-lsp](https://github.com/prabirshrestha/asyncomplete-lsp.vim).

# Linters

The popular options are ALE and [neomake](https://github.com/neomake/neomake).
It supports hlint for haskell. One can bind the invocation of `:Neomake` to a
key combination to run hlint on the open haskell file in the buffer. It will
populate what vim calls a [Location
List](https://noahfrederick.com/log/a-list-of-vims-lists). For easier
navigation, key bindings can be setup to traverse the location list.

# Build

One of the features IDEs provide is to do a build and then jump to the
location of the error. The same can be achieved using `makeprg` and
`errorformat`. For some reason, the in-built support to use this with `stack
build` does not work. A custom
[compiler](https://gitlab.com/SanchayanMaity/dotfiles/-/tree/master/nvim/.config/nvim/compiler)
support is added in the referenced link for this to work.

```
" https://www.reddit.com/r/neovim/comments/es8wn7/haskell_makeprg_for_stack_build/
" https://github.com/maxigit/vimrc/tree/2020/compiler
au FileType haskell compiler stack
au FileType haskell setlocal makeprg=stack\ build
```

After this, invocation of `makeprg` from inside vim or if neomake is being
used, invoking `:Neomake!` will result in `stack build` being run, which will
populate the quickfix list once done and then using either using quickfix
window or quickfix window shortcuts one can jump to error locations without
leaving vim.

# Miscellaneous

Some of the other plugins worth having are
[tabularize](https://github.com/godlygeek/tabular),
[identline](https://github.com/Yggdroot/indentLine),
[Hoogle](https://github.com/Twinside/vim-hoogle) and
[ghcid](https://github.com/ndmitchell/ghcid). The last two provide hoogle and
ghcid integration inside vim.

# Sample dotfiles

My personal configuration can be found
[here](https://gitlab.com/SanchayanMaity/dotfiles/-/tree/master/nvim/.config/nvim).
