---
tags: [ ansible, automation ]
---
# Setting up my computer

```sh
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"`

# Clone dotfiles
git clone --recursive https://github.com/kejadlen/dotfiles.git ~/.dotfiles

# Install Ansible
brew install ansible

# Run Ansible
cd ~/.dotfiles/ansible
echo "localhost ansible_connection=local" > hosts.private

# https://github.com/Homebrew/brew/issues/7667
brew install curl
export HOMEBREW_FORCE_BREWED_CURL=1

ansible-playbook main.yml --ask-pass --ask-become-pass
ansible-playbook playbooks/mbp.yml
ansible-playbook playbooks/rust.yml
```

## 2020.06.02

- Ran into [#7667][7667] when installing some casks, with system curl not
  dealing with expired certs correctly? (See related [Stack Overflow][SO]
  post.) `CURL_SSL_BACKEND=secure-transport` is supposed to work, but had no
  effect for me. Installing curl through Homebrew and using
  `HOMEBREW_FORCE_BREWED_CURL=1` did work, though.

[7667]: https://github.com/Homebrew/brew/issues/7667
[SO]: https://security.stackexchange.com/questions/232445/https-connection-to-specific-sites-fail-with-curl-on-macos

- vimwiki having a fsck error in its git graph continues to plague me:

  ```sh
  Cloning into '/Users/alpha/.dotfiles/.vim/pack/alpha/start/vimwiki'...
  error: object d5a6d097da1c67e07b3fb073f22217c7462f5088: badTimezone: invalid author/committer line - bad time zone
  fatal: fsck error in packed object
  fatal: index-pack failed
  fatal: clone of 'https://github.com/vimwiki/vimwiki.git' into submodule path '/Users/alpha/.dotfiles/.vim/pack/alpha/start/vimwiki' failed
  ```

  - Fixed by setting `shallow = true` in `.gitmodules`.

- A bunch of submodules wound up in a strange state where everything was
  deleted and I had to `reset --hard` back to `master`. Probably related to my
  forgetting to clone with `--recursive`, but an unexpected result of
  `submodule init` and `submodule update`. Not sure how this happened.

- Still need a better way of transferring application and system settings

