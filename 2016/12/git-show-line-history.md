# Git show line history

> Date: 2016-12-06
>
> Author: sfwn
>
> Title: Git 查看 某一个文件某一行 的提交记录

#### `$ git blame /path/to/file`

使用该命令查看某一文件每一行最后一次提交的记录，找出最有可能的接锅侠。
如果只想看特定的几行，可以使用:  
`$ git blame /path/to/file -L 15,15` 只查看第15行，  
或者使用:  
`$ git blame /path/to/file -L 15,+11` 来查看从 15 到 (15+11-1)=25 行的 blame。

#### `$ git log -L15,15:/path/to/file`

如果你对其中某一行特别感兴趣，比如第15行，想查看他的所有记录，可以使用上述命令。
_注意_ : 就算文件被 rename 过，上述命令也会搜索第15行原始文件中的记录。
