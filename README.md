# zsh vcs_info integration for jj

This adds [jj](https://jj-vcs.github.io/jj/latest/) support to zsh's [vcs_info](https://zsh.sourceforge.io/Doc/Release/User-Contributions.html#Version-Control-Information).
It's a bit quick-and-dirty but works for me.

`vcs_info` collects a few different values from the underlying version control system.
This is how this integration maps them.
* `action` - `divergent`, `hidden`, `conflict`, `empty`, and/or `root` in a space-separated string
* `branch` - Local bookmarks, if any
* `base` - Unused
* `staged` - Unused
* `unstaged` - Unused
* `revision` - A string consisting of the jj change ID and the git commit hash, limited to 8 characters each and separated by a space.  eg, `qxospont 5d66059c`
* `misc` - Unused

Using `action` for those values instead of `misc` is probably wrong, but it makes conditionalizing the display easier.

## Installation

Place the `VCS_INFO_detect_jj` and `VCS_INFO_get_data_jj` files in your `fpath`.

## Configuration

In your `zshrc` file, if you're using the git integration, you probably have a line like this:

```zsh
zstyle ':vcs_info:*' enable git
```

Change that to:

```zsh
zstyle ':vcs_info:*' enable jj git
```

Because `jj` repos are also `git` repos, it's **very important** that `jj` come before `git` in the list.

You can then configure your formats as you see fit.
Personally, I have it configured like this:

```zsh
zstyle ':vcs_info:jj:*' formats "%i" "%b"
zstyle ':vcs_info:jj:*' actionformats "%a|%i" "%b"
```

`%a` is the `action` value.
`%i` is the `revision` value.
`%b` is the `branch` value.

The two formats mean that the first value is placed into `vcs_info_msg_0_` and the second value is placed into `vcs_info_msg_1_`.

In my `precmd` function, I have this:

```zsh
psvar=()
[[ -n $vcs_info_msg_0_ ]] && print -v 'psvar[1]' -Pr -- "${vcs_info_msg_0_}"
[[ -n $vcs_info_msg_1_ ]] && print -v 'psvar[2]' -Pr -- "${vcs_info_msg_1_}"
```

And when I construct my prompt, I have something like this:

```zsh
PROMPT="%m %~%(1v. (%1v%(2v. %2v.)).)%# "
```

The key part here is this chunk:

```zsh
%(1v. (%1v%(2v. %2v.)).)
```

This is a nested conditional.
`%1v` and `%2v` are the values that were placed into `psvar[1]` and `psvar[2]`, respectively.
If `%1v` is non-empty, it prints a space, open parenthesis, the `%1v` value, the `%2v` conditional, and the closing parenthesis.
If `%2v` is non-empty, it prints a space and then its value.

So for something like an empty change, the prompt will look like this:

```
caisson ~/src/jj-zsh-vcs-info (empty|qxospont 5d66059c)%
```

And for a bookmark, it will look like this:

```
caisson ~/src/kia_uvo (mkltqqlm 7d9e3d04 services-yaml-update)%
```


