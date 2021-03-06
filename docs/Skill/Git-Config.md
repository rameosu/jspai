# Git骚操作
## 一、配置Http/Https代理

1. 位置：C盘 -》 用户 -》 当前用户文件夹 -》 .gitconfig文件

2. 末尾添加配置

```xml
[http]
	proxy = http://127.0.0.1:4780 // 端口号为你的VPN http端口
[https]
	proxy = https://127.0.0.1:4780 // 端口号为你的VPN httpss端口
```

## 二、设置别名

1. 位置：git安装目录 -》 etc -》bash.bashrc文件

2. 末尾添加配置

   ```xml
   alias g='git'
   
   alias ga='git add'
   alias gaa='git add --all'
   alias gapa='git add --patch'
   
   alias gb='git branch'
   alias gba='git branch -a'
   alias gbda='git branch --merged | command grep -vE "^(\*|\s*master\s*$)" | command xargs -n 1 git branch -d'
   alias gbl='git blame -b -w'
   alias gbnm='git branch --no-merged'
   alias gbr='git branch --remote'
   alias gbs='git bisect'
   alias gbsb='git bisect bad'
   alias gbsg='git bisect good'
   alias gbsr='git bisect reset'
   alias gbss='git bisect start'
   
   alias gc='git commit -v'
   alias gc!='git commit -v --amend'
   alias gcn!='git commit -v --no-edit --amend'
   alias gca='git commit -v -a'
   alias gca!='git commit -v -a --amend'
   alias gcan!='git commit -v -a --no-edit --amend'
   alias gcans!='git commit -v -a -s --no-edit --amend'
   alias gcam='git commit -a -m'
   alias gcb='git checkout -b'
   alias gcf='git config --list'
   alias gcl='git clone --recursive'
   alias gclean='git clean -fd'
   alias gpristine='git reset --hard && git clean -dfx'
   alias gcm='git checkout master'
   alias gcmsg='git commit -m'
   alias gco='git checkout'
   alias gcd='git checkout develop'
   
   alias gcp='git cherry-pick'
   alias gcs='git commit -S'
   
   alias gd='git diff'
   alias gdca='git diff --cached'
   alias gdct='git describe --tags `git rev-list --tags --max-count=1`'
   alias gdt='git diff-tree --no-commit-id --name-only -r'
   
   alias gdw='git diff --word-diff'
   
   alias gf='git fetch'
   alias gfa='git fetch --all --prune'
   
   alias gl='git pull'
   alias glg='git log --stat'
   alias glgp='git log --stat -p'
   alias glgg='git log --graph'
   alias glgga='git log --graph --decorate --all'
   alias glgm='git log --graph --max-count=10'
   alias glo='git log --oneline --decorate'
   alias glol="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
   alias glola="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all"
   alias glog='git log --oneline --decorate --graph'
   alias gloga='git log --oneline --decorate --graph --all'
   
   alias gm='git merge'
   alias gmom='git merge origin/master'
   alias gmt='git mergetool --no-prompt'
   alias gmtvim='git mergetool --no-prompt --tool=vimdiff'
   alias gmum='git merge upstream/master'
   
   alias gp='git push'
   alias gpd='git push --dry-run'
   alias gpoat='git push origin --all && git push origin --tags'
   
   alias gpu='git push upstream'
   alias gpv='git push -v'
   
   alias gr='git remote'
   alias gra='git remote add'
   alias grb='git rebase'
   alias grba='git rebase --abort'
   alias grbc='git rebase --continue'
   alias grbi='git rebase -i'
   alias grbm='git rebase master'
   alias grbs='git rebase --skip'
   alias grh='git reset HEAD'
   alias grhh='git reset HEAD --hard'
   alias grmv='git remote rename'
   alias grrm='git remote remove'
   alias grset='git remote set-url'
   alias grt='cd $(git rev-parse --show-toplevel || echo ".")'
   alias gru='git reset --'
   alias grup='git remote update'
   alias grv='git remote -v'
   
   alias gsb='git status -sb'
   alias gsd='git svn dcommit'
   alias gsi='git submodule init'
   alias gsps='git show --pretty=short --show-signature'
   alias gsr='git svn rebase'
   alias gss='git status -s'
   alias gst='git status'
   alias gsta='git stash save'
   alias gstaa='git stash apply'
   alias gstd='git stash drop'
   alias gstl='git stash list'
   alias gstp='git stash pop'
   alias gsts='git stash show --text'
   alias gsu='git submodule update'
   
   alias gup='git pull --rebase'
   ```

   

## 三、IDEA设置Terminal为Git终端

1. 打开idea的 settings -》Tools -》Terminal
2. shell path配置成 git安装目录/bin/bash.exe
