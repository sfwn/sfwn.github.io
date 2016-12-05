# git get other branches after shallow clone

> Date: 2017-03-28
>
> Author: sfwn

1. `$ git clone --depth=1 git@xxx/xxx.git"`
2. `$ git remote set-branches --add origin master`
3. `$ git checkout master; git fetch --depth=1`