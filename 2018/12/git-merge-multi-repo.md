# git 合并多个仓库

> Date:  2018-12-19
>
> Author: sfwn

在新仓库或者基础仓库

1. git remote add xxx {url}
2. git merge xxx/{branch/tag} --allow-unrelated-histories
3. mkdir xxx
4. ls -ltrA | grep -v -- xxx | grep -v .git | awk '{print $9}' | xargs -I {} git mv {} xxx
5. 检查其他文件是否需要 git mv 到 xxx
6. git commit