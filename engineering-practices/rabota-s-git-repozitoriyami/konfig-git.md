# Конфиг git

1. Скопируйте сниппет настроек в `~/.gitconfig` в Unix или папку пользователя в Windows
2. Укажите свое имя (на русском) и корпоративный email
3. (желательно) Настройте подпись коммитов
   1. Установите GPG
   2. Создайте ключ для подписи коммитов
   3. Укажите ключ в `signingkey`
   4. Установите `gpgSign` в `true`

```bash
[user]
    name =
    email =
    signingkey =
 
[color]
    ui = true
 
[filter "lfs"]
    clean    = git-lfs clean -- %f
    smudge   = git-lfs smudge -- %f
    process  = git-lfs filter-process
    required = true
 
[alias]
    pfl   = push --force-with-lease
    rom   = rebase origin/master
    prune = fetch --prune
 
    # https://git-scm.com/docs/git-reset#git-reset-emgitresetemltmodegtltcommitgt
    undo = reset --soft HEAD^
 
    # https://git-scm.com/docs/git-stash
    stash-all = stash save --include-untracked
 
    # https://git-scm.com/docs/git-log
    glog = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset'
 
[merge]
    # NOTE: глобальная опция – применяется ко всем слияниям, включая через merge-реквесты
    # https://git-scm.com/docs/git-config#git-config-mergeff
    ff = only
 
    # https://git-scm.com/docs/git-config#git-config-mergeconflictStyle
    conflictstyle = diff3
 
[commit]
    # подпись коммитов посредством ключей через GPG
    # https://help.github.com/articles/about-gpg/
    # https://git-scm.com/docs/git-config#git-config-commitgpgSign
    gpgSign = false
 
[tag]
    # подпись коммитов посредством ключей через GPG
    gpgSign = false
 
[push]
    # пуш изменений в удаленную ветку с тем же именем
    # https://git-scm.com/docs/git-config#git-config-pushdefault
    default = simple
 
    # https://git-scm.com/docs/git-config#git-config-pushfollowTags
    followTags = true
 
[status]
    # https://git-scm.com/docs/git-config#git-config-statusshowUntrackedFiles
    showUntrackedFiles = all
 
[transfer]
    # NOTE: применяется в рамках receive и transmit операций
    # https://git-scm.com/docs/git-config#git-config-transferfsckObjects
    # via https://groups.google.com/forum/#!topic/binary-transparency/f-BI4o8HZW0
    fsckobjects = true
 
[diff]
    # via http://owen.cymru/github-style-diff-in-terminal-with-icdiff/
    tool = icdiff
 
[difftool]
    prompt = false
    trustExitCode = true
 
[difftool "icdiff"]
  cmd = /usr/local/bin/icdiff --line-numbers $LOCAL $REMOTE
 
[mergetool]
    prompt = false
```
