### 安装Homebrew

```bash
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

```
# iterm2配置fish
https://learnku.com/articles/3647/iterm-2-simple-installation-of-fish-and-configure-the-theme
```


### Git初始化

```
#Git 全局设置:

git config --global user.name "wu.wenjie"
git config --global user.email "1014259604@qq.com"

#创建 git 仓库:

mkdir abc123
cd abc123
git init 
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin https://gitee.com/jeeKiss/abc123.git
git push -u origin "master"

#已有仓库?

cd existing_git_repo
git remote add origin https://gitee.com/jeeKiss/abc123.git
git push -u origin "master"
```

