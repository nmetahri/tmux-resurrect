# Tmux Resurrect

> **Maintained fork of [tmux-plugins/tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect).**
> Upstream has had no merged changes since March 2023, while long-standing bugs
> remain open. This fork applies fixes on top of upstream `master` and stays
> drop-in compatible with existing save files and options.

## Changes from upstream

| Change | Upstream issue / PR | Thanks to |
|---|---|---|
| Panes restored to `$HOME` when their title is empty: an empty `pane_title` made bash `read` collapse consecutive tab delimiters and shift every following field. Empty titles now fall back to the short hostname. | [#520](https://github.com/tmux-plugins/tmux-resurrect/pull/520), [#564](https://github.com/tmux-plugins/tmux-resurrect/pull/564), [#581](https://github.com/tmux-plugins/tmux-resurrect/pull/581), [#583](https://github.com/tmux-plugins/tmux-resurrect/pull/583) | [@scmanjarrez](https://github.com/scmanjarrez), [@KTSCode](https://github.com/KTSCode), [@chirayuk](https://github.com/chirayuk), [@josh-stephens](https://github.com/josh-stephens) |
| A save firing mid-boot or mid-shutdown could capture zero panes and repoint `last` to it; restoring that empty file killed the session. Paneless snapshots are now neither saved nor restored. | [#578](https://github.com/tmux-plugins/tmux-resurrect/pull/578), [#576](https://github.com/tmux-plugins/tmux-resurrect/pull/576) | [@googlebleh](https://github.com/googlebleh) |
| Two saves within the same second deleted the only snapshot and left `last` dangling. | [#598](https://github.com/tmux-plugins/tmux-resurrect/issues/598), [#599](https://github.com/tmux-plugins/tmux-resurrect/pull/599) | [@ericz7900](https://github.com/ericz7900) |
| Restore re-read `last` at every phase, so a concurrent save could mix two snapshots. The snapshot is now pinned once at restore start. | [#584](https://github.com/tmux-plugins/tmux-resurrect/pull/584) | [@kvoli](https://github.com/kvoli) |
| Concurrent restores (e.g. tmux-continuum firing twice at login) added extra panes squashed to one line. Only one restore now runs at a time per server (`flock`). | [#437](https://github.com/tmux-plugins/tmux-resurrect/issues/437), [#600](https://github.com/tmux-plugins/tmux-resurrect/pull/600) | [@andrewboldi](https://github.com/andrewboldi) |
| Sessions with a dot in their name (`v3.69-x`) lost a window on restore. | [#596](https://github.com/tmux-plugins/tmux-resurrect/issues/596), [#597](https://github.com/tmux-plugins/tmux-resurrect/pull/597) | [@aharoJ](https://github.com/aharoJ) |
| Windows whose name starts with a digit could shadow a missing window index, so the window was skipped on restore. | [#575](https://github.com/tmux-plugins/tmux-resurrect/pull/575) | [@theolundqvist](https://github.com/theolundqvist) |
| Restored process commands containing `,`, `&` or `\` were corrupted by a `sed` substitution. | [#588](https://github.com/tmux-plugins/tmux-resurrect/issues/588), [#587](https://github.com/tmux-plugins/tmux-resurrect/pull/587) | [@smoser](https://github.com/smoser) |
| The resurrect directory was world-readable with the default umask; it is now restricted to its owner. | [#561](https://github.com/tmux-plugins/tmux-resurrect/issues/561), [#562](https://github.com/tmux-plugins/tmux-resurrect/pull/562) | [@burka](https://github.com/burka) |
| Sessions were saved and restored in alphabetical order instead of the user's creation order. Session ids are sorted numerically in this fork, so order stays correct beyond 10 sessions. | [#388](https://github.com/tmux-plugins/tmux-resurrect/issues/388), [#476](https://github.com/tmux-plugins/tmux-resurrect/pull/476) | [@Dosx001](https://github.com/Dosx001) |
| Saving ran `ps` once per pane. The process table is now read once per save (about 4x faster on ~40 panes). | [#566](https://github.com/tmux-plugins/tmux-resurrect/pull/566) | [@thewatts](https://github.com/thewatts) |

Huge thanks to everyone above who took the time to diagnose these bugs and open
pull requests upstream: this fork mostly collects their work. Each fix is a
separate commit authored by its original contributor.

Install this fork instead of upstream with TPM:

    set -g @plugin 'nmetahri/tmux-resurrect'

---

Restore `tmux` environment after system restart.

Tmux is great, except when you have to restart the computer. You lose all the
running programs, working directories, pane layouts etc.
There are helpful management tools out there, but they require initial
configuration and continuous updates as your workflow evolves or you start new
projects.

`tmux-resurrect` saves all the little details from your tmux environment so it
can be completely restored after a system restart (or when you feel like it).
No configuration is required. You should feel like you never quit tmux.

It even (optionally)
[restores vim and neovim sessions](docs/restoring_vim_and_neovim_sessions.md)!

Automatic restoring and continuous saving of tmux env is also possible with
[tmux-continuum](https://github.com/tmux-plugins/tmux-continuum) plugin.

### Screencast

[![screencast screenshot](/video/screencast_img.png)](https://vimeo.com/104763018)

### Key bindings

- `prefix + Ctrl-s` - save
- `prefix + Ctrl-r` - restore

### About

This plugin goes to great lengths to save and restore all the details from your
`tmux` environment. Here's what's been taken care of:

- all sessions, windows, panes and their order
- current working directory for each pane
- **exact pane layouts** within windows (even when zoomed)
- active and alternative session
- active and alternative window for each session
- windows with focus
- active pane for each window
- "grouped sessions" (useful feature when using tmux with multiple monitors)
- programs running within a pane! More details in the
  [restoring programs doc](docs/restoring_programs.md).

Optional:

- [restoring vim and neovim sessions](docs/restoring_vim_and_neovim_sessions.md)
- [restoring pane contents](docs/restoring_pane_contents.md)
- [restoring a previously saved environment](docs/restoring_previously_saved_environment.md)

Requirements / dependencies: `tmux 1.9` or higher, `bash`.

Tested and working on Linux, OSX and Cygwin.

`tmux-resurrect` is idempotent! It will not try to restore panes or windows that
already exist.<br/>
The single exception to this is when tmux is started with only 1 pane in order
to restore previous tmux env. Only in this case will this single pane be
overwritten.

### Installation with [Tmux Plugin Manager](https://github.com/tmux-plugins/tpm) (recommended)

Add plugin to the list of TPM plugins in `.tmux.conf`:

    set -g @plugin 'nmetahri/tmux-resurrect'

Hit `prefix + I` to fetch the plugin and source it. You should now be able to
use the plugin.

### Manual Installation

Clone the repo:

    $ git clone https://github.com/nmetahri/tmux-resurrect ~/clone/path

Add this line to the bottom of `.tmux.conf`:

    run-shell ~/clone/path/resurrect.tmux

Reload TMUX environment with: `$ tmux source-file ~/.tmux.conf`.
You should now be able to use the plugin.

### Docs

- [Guide for migrating from tmuxinator](docs/migrating_from_tmuxinator.md)

**Configuration**

- [Changing the default key bindings](docs/custom_key_bindings.md).
- [Setting up hooks on save & restore](docs/hooks.md).
- Only a conservative list of programs is restored by default:<br/>
  `vi vim nvim emacs man less more tail top htop irssi weechat mutt`.<br/>
  [Restoring programs doc](docs/restoring_programs.md) explains how to restore
  additional programs.
- [Change a directory](docs/save_dir.md) where `tmux-resurrect` saves tmux
  environment.

**Optional features**

- [Restoring vim and neovim sessions](docs/restoring_vim_and_neovim_sessions.md)
  is nice if you're a vim/neovim user.
- [Restoring pane contents](docs/restoring_pane_contents.md) feature.

### Other goodies

- [tmux-copycat](https://github.com/tmux-plugins/tmux-copycat) - a plugin for
  regex searches in tmux and fast match selection
- [tmux-yank](https://github.com/tmux-plugins/tmux-yank) - enables copying
  highlighted text to system clipboard
- [tmux-open](https://github.com/tmux-plugins/tmux-open) - a plugin for quickly
  opening highlighted file or a url
- [tmux-continuum](https://github.com/tmux-plugins/tmux-continuum) - automatic
  restoring and continuous saving of tmux env

### Reporting bugs and contributing

Both contributing and bug reports are welcome. Please check out
[contributing guidelines](CONTRIBUTING.md).

### Credits

[Mislav Marohnić](https://github.com/mislav) - the idea for the plugin came from his
[tmux-session script](https://github.com/mislav/dotfiles/blob/2036b5e03fb430bbcbc340689d63328abaa28876/bin/tmux-session).

### License
[MIT](LICENSE.md)
