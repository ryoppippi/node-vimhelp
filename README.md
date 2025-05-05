node-vimhelp
============

[![NPM Version][npm-image]][npm-url]
[![Node.js Version][node-version-image]][node-version-url]
[![Test][test-ci-badge]][test-ci-action]
[![Lint][lint-ci-badge]][lint-ci-action]
[![Test Coverage][codecov-image]][codecov-url]

Show the help of [Vim](https://github.com/vim/vim).

This package uses Vim in background.  So you need Vim in your environment.

Requirements
------------

- Node.js v22.0.0+
- Vim
  - Version 7.4 or later is recommended.
- Git
  - To use PluginManager.
  - Git 2.3 or above is recommended.
    - Because `GIT_TERMINAL_PROMPT` environment variable is available.
      This script may freeze when the interactive prompt appears with old Git.
      For example, when you specify the non-exist repository.

Installation
------------

Install it using [npm](https://www.npmjs.com/):

```
$ npm install vimhelp
```

Synopsis
--------

```javascript
// You can search the help of Vim.
const {VimHelp} = require("vimhelp");

(async () => {
let vimHelp = new VimHelp();

let text = await vimHelp.search("j");
console.log(text);
/* The following text is shown:
j               or                                      *j*
<Down>          or                                      *<Down>*
CTRL-J          or                                      *CTRL-J*
<NL>            or                                      *<NL>* *CTRL-N*
CTRL-N                  [count] lines downward |linewise|.
*/
})();


// This package have a simple plugin manager.
const {PluginManager} = require("vimhelp");

(async () => {
// Plugins are installed to under the basedir.
let manager = new PluginManager("/path/to/basedir");

await manager.install("thinca/vim-quickrun");
console.log(manager.pluginNames);  // => ["thinca/vim-quickrun"]

await manager.install("vim-jp/vital.vim");
console.log(manager.pluginNames);  // => ["thinca/vim-quickrun", "vim-jp/vital.vim"]

await manager.uninstall("vim-jp/vital.vim");
console.log(manager.pluginNames);  // => ["thinca/vim-quickrun"]



// You can also search the help of plugin with PluginManager.
vimHelp.setRTPProvider(manager.rtpProvider);
let text = await vimhelp.search("quickrun");
console.log(text);
// => *quickrun* is Vim plugin to execute whole/part of editing file.
// => (...snip)

// You can specify the 'helplang' option
vimHelp.helplang = ["ja", "en"];
let text = await vimhelp.search("quickrun");
console.log(text);
// => *quickrun* は編集中のファイルの全体もしくは一部を実行する Vim プラグインです。
// => (...snip)

})();
```

References
----------

### class VimHelp

`VimHelp` can take help text of Vim via `vim` command.

#### new VimHelp([{vimBin}])

- `{vimBin}`
  - Path to vim command.  "vim" in the `$PATH` is used in default.

#### .search({subject})

Searches `{subject}` from Vim's help and returns A Promise.
When the `{subject}` is found, the Promise will be resolved by the help text.
Otherwise, rejected an object like `{errorCode, resultText, errorText}`.
In most cases, `errorText` is useful to know the cause of error.

#### .setRTPProvider({provider})

- `{provider}`
  - A runtimepath provider.  This is a function that has no arguments and returns an array of paths.

The runtimepaths from this is used in `.search()`.

#### .helplang

An array of langs to set `'helplang'` option.
Default value is an empty array.
ex: `["ja", "en"]`

### class PluginManager

PluginManager can install Vim plugins from any Git repository.

This class references the Vim plugin by `{plugin-name}`.
`{plugin-name}` is one of the following.
- A repository name of https://github.com/vim-scripts
  - ex: `taglist.vim`
- A GitHub repository name like `{user}/{repos}`
  - ex: `vim-jp/vital.vim`
- An URL of Git repository.
  - ex: `https://bitbucket.org/ns9tks/vim-fuzzyfinder`

This class doesn't do a exclusive operation at all for simplification.

#### new PluginManager([{basePath} [, {vimBin}]])

- `{basePath}`
  - Path to base of plugins.  All plugins are installed to under this directory.
- `{vimBin}`
  - Path to vim command.  "vim" in the `$PATH` is used in default.
  - This is used when updating helptags.

#### .plugins

An array of installed plugin informations.  Read only.
Each entry is object as follows:

```javascript
{
  pluginName: <plugin name>,
  dirName: <dir name>,
  runtimepath: <runtimepath>,
  repository: <URL of repository>
}
```

#### .pluginNames

An array of installed `{plugin-name}`s.  Read only.

#### .runtimepaths

An array of paths of installed plugins.  Read only.

#### .rtpProvider

A function that returns paths of installed plugins.
This is not a method.  You can pass this function to `VimHelp.setRTPProvider()`.
Read only.

#### .install({plugin-name})

Installs a new Vim plugin.  Returns a Promise.
When Installation succeeded, resolved by commit hash of Git which is version of installed plugin.
Otherwise, rejected by Error object or an object like `{errorCode, resultText, errorText}`.

#### .uninstall({plugin-name})

Uninstalls a installed Vim plugin.  Returns a Promise.
When Uninstallation succeeded, resolved by the path of uninstalled plugin.
Otherwise, rejected by Error object or an object like `{errorCode, resultText, errorText}`.

#### .clean()

Uninstalls all Vim plugins.

#### .update([{plugin-names}])

Updates installed plugins.  `{plugin-names}` is an array of `{plugin-name}`.
When `{plugin-names}` is omitted, all plugins are updated.  Returns a Promise.
When updating succeeded, resolved by the array of updating informations.
An updating information is following:

- pluginName
  - `{plugin-name}` of this plugin.
- pluginPath
  - A local path of this plugin.
- beforeVersion
  - A commit hash of Git which is version of before updating.
- afterVision
  - A commit hash of Git which is version of after updating.
- updated()
  - A function that returns true when this plugin is updated.

When updating failed, resolved by the array of updating informations.

Testing
-------

```
$ npm test
```

License
-------

[zlib License](LICENSE.txt)

Author
------

thinca <thinca+npm@gmail.com>


[npm-image]: https://img.shields.io/npm/v/vimhelp.svg
[npm-url]: https://npmjs.org/package/vimhelp
[node-version-image]: https://img.shields.io/node/v/vimhelp.svg
[node-version-url]: https://nodejs.org/en/download/
[test-ci-badge]: ./../../workflows/Test/badge.svg
[test-ci-action]: ./../../actions?query=workflow%3ATest
[lint-ci-badge]: ./../../workflows/Lint/badge.svg
[lint-ci-action]: ./../../actions?query=workflow%3ALint
[codecov-image]: https://codecov.io/gh/thinca/node-vimhelp/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/thinca/node-vimhelp
