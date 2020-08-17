---
tags:
  - git
---

# Shallow [Git] Submodules

I use submodules to manage my vim plugins, but some git repositories like
[vimwiki][vimwiki] have corrupted git histories that prevent cloning with
`transfer.fsckObjects = true`.

[vimwiki]: https://github.com/vimwiki/vimwiki

```sh
❯ git clone git@github.com:vimwiki/vimwiki.git
Cloning into 'vimwiki'...
remote: Enumerating objects: 28, done.
remote: Counting objects: 100% (28/28), done.
remote: Compressing objects: 100% (16/16), done.
error: object d5a6d097da1c67e07b3fb073f22217c7462f5088: badTimezone: invalid author/committer line - bad time zone
fatal: fsck error in packed object
fatal: index-pack failed
```

This can be worked around by setting the submodule to clone with [`shallow =
true`][shallow-true], but this configures the submodule repository to only
fetch the `master` branch from `origin` and breaks fetching and switching to
other remote branches. In particular, I wanted the `dev` branch to get some
fixes that weren't merged to `master` yet.

[shallow-true]: https://github.com/kejadlen/dotfiles/blob/8c355000ff66e7f2f61a1d4a8c90ef367884b724/.gitmodules#L118

A workaround for this is to find the configuration for the submodule in
`.git/modules` and add the branch manually. I assume there's some way of making
git do this via the command line, but I couldn't figure it out, and this
worked.

```diff
 [remote "origin"]
 	url = https://github.com/vimwiki/vimwiki.git
 	fetch = +refs/heads/master:refs/remotes/origin/master
+	fetch = +refs/heads/dev:refs/remotes/origin/dev
```
