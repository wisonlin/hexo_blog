---
title: 一台机器上同时使用两个 Github 账号
tags:
---

ssh-keygen -C "1027620001@qq.com" -f ~/.ssh/UOSIM_github

cat ~/.ssh/UOSIM_github

Host wisonlin.github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_rsa

Host UOSIM.github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/UOSIM_github

ssh -T git@UOSIM.github.com


cat .git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[remote "origin"]
	url = git@UOSIM.github.com:UOSIM/dribbble_cache.git
  
