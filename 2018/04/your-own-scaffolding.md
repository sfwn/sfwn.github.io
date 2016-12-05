# 实现一个属于自己的脚手架工具

> Date:  2018-04-09
>
> Author: Shiyi Xie

为什么做这件事？

原因很简单，因为每次初始化一个项目，都用的固定结构，但是要一个一个创建文件夹，再创建相应的文件，很繁琐。虽然已经有vue-cli这样的工具，但是做一个合适自己开发的框架不是更好嘛！

![](/content/images/2018/04/-----.jpeg)   

功能实现步骤：

1、 需要一个模板项目
2、 实现cli工具，将这个模板项目可以通过下载/拉取已有的工程到本地，完成生成项目的工作
3、 允许通过问答的形式对模板进行个性化配置

### 20180328  start one

开始完成一个模板项目。
这部分我是将自己正在开发中的项目框架给抽离出来，形成一个模板项目。
个人可根据实际情况开发模板项目，有兴趣请参考[我的模板](https://github.com/xieshiyi/scaffold-react)。

***
### 20180329 start two

实现一个cli工具，开始找一些学习资料，看文档。

#### 搭建项目结构
```
|—— bin                     
|    |—— struct               // 脚手架命令，提供对模板操作的能力
|    |—— struct-init          // 脚手架命令，提供初始化项目的能力
|—— lib        
|    |—— add.js
|    |—— delete.js
|    |—— list.js                       
|—— package.json                      
|—— template.json            // 存放模板的文件
```
主体结构如上所示。

#### 部分核心代码
首先，需要在package.json文件中添加如下配置，这样配置后， 便于在全局使用struct 和struct init 2个命令
```
"bin": {
    "struct": "bin/struct",
    "struct-init": "bin/struct-init"
  },
```

struct文件
```
#!/usr/bin/env node     //必须放到文件开头
const program = require('commander');

program
  .version(require('../package').version)     //  显示版本号
  .usage('<command> [options]');
  
program
    .command('list')
    .alias('l')
    .description('显示所有模板')
    .action(require('../lib/list'));
  
program
    .command('add')
    .alias('a')
    .description('添加模板')
    .action(require('../lib/add'));

program
    .command('delete')
    .alias('d')
    .description('删除模板')
    .action(require('../lib/delete'));
```


*运行 npm link 将脚手架挂在到全局 ， 此时可以在命令行调试新建的脚手架框架了。*

![-----h-2](/content/images/2018/04/-----h-2.png)


list.js文件实现显示模板

```
const fs = require('fs');
const chalk = require('chalk');
const tplJson = require(`${__dirname}/../template.json`);

module.exports = function() {
    Object.keys(tplJson).forEach((item) => {
        let tplData = tplJson[item];
        console.log('  ' + 
            chalk.yellow('★') + 
            '  ' + chalk.yellow(tplData.name) + 
            ' - ' + tplData.description + 
            ' - ' + chalk.red(`模板安装包${tplData.npm}`));
    })
}
```

***
### 20180403
之前一直都没办法实现命令行就连 ``` struct -h```这样的命令，都没有任何反应。


![----](/content/images/2018/04/----.jpeg)


后来才发现，原因是少了一句：

```
// struct文件 
program.parse(process.argv);
```

***
### 20180404 初步完成
简单总结下主要用到的技术模块
* commander     // 命令行界面
* inquirer      // 向用户提问， 并获取输入，将新加的模板信息，写入template.json 
* exec          // 用于执行shell命令，比如git clone&nbsp;…
* chalk         // 将console.log内容高亮，可配置颜色，chalk.yellow则输入内容为黄色
* ora           // 下载模块时可以展示动态loading

这次没有用到download-git-repo专门从git仓库下载文件到目标目录，因为exec就能完成全部工作。

***
### 20180408 完善

昨天想着，下载的模板带有git信息，这样，别人使用的时候，提交到我的github以及分支上了。

![--](/content/images/2018/04/--.jpeg)

所以得移除掉.git文件。
赶紧改下。

顺便发现README.md里面，命令行不对~ 应该是scaffold-init，当时创建scaffold-cli发布到npm上的时候发现被人使用了，所以只能改。

vue-cli用的是download-git-repo，但是我发现其实download-git-repo只能把项目down下来，但是并不能同时移除掉.git文件。所以感觉还是直接用exec更好些，起码整个文档比较干净，没有带着原作者的信息。

完整项目请参考 [xieshiyi/scaffold-init](https://github.com/xieshiyi/scaffold-init).


***
感谢阅读我的博客，喜欢请关注！