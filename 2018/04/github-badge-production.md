# github 徽章制作

> Date:  2018-04-10
>
> Author: Shiyi Xie

被问到一个问题，github上的徽章知道哪儿来的吗？
![-----1](/content/images/2018/04/-----1.jpeg)

不过还是去看了下，并且这一天就这样在这条路上没回头。
***
### [官方徽章大全](http://shields.io/)

先列出官方给出来的徽章地址，由于没有文档说明，使用的也不多，很多地方还需要自己摸索，在此举例说明下如何使用。


#### 徽章一

前提说明：我有一个scaffold-init项目，打包之后托管到了npm上，所以我希望可以有一个展示npm version的徽章。
* 找到需要的徽章

![npm](/content/images/2018/04/npm.png)

* 需要实现的功能：显示自己项目最新版本号

那么问题来了，直接用这个徽章地址，展示的是npm最新版本号，但是我希望展示的是我自己项目的最新版本号。
此时，点击链接地址，会弹出
![npm2](/content/images/2018/04/npm2.png)
我们将Image一栏的地址https://img.shields.io/npm/v/npm.svg 中的/v/npm.svg替换成/v/myProject.svg就发现，徽章展示的是自己项目的版本号了。

*当然，myProject必须是在npm上托管的包名称*

#### 徽章二
此外，还希望可以展示当前项目的build结果。
当前选择的是circleci对自己项目进行build。

* 先找到需要的徽章

![circleci](/content/images/2018/04/circleci.png)
* 需要实现的功能：实时展示build结果

参考徽章一的做法，以此类推，可以发现https://img.shields.io/circleci/project/github/ 这一部分共有的，后面应该是github的仓库地址，只要替换成自己项目地址即可。

### 后记：

顺便说明下circleci的配置，因为刚开始学习还是踩了坑的。

先去注册了一个账号，这部分省略。

完成官方教学流程之后，会拿到一个模板配置文件，接下来开始配置这个文件。

我的第一次配置如下(很莽撞，贱笑了)：

```
version: 2
jobs:
  build:
    docker:
      - image: debian:stretch
    steps:
      - checkout
      - restore_cache:
          keys:
          - v1-dependencies-{{ checksum "yarn.lock" }}
          # fallback to using the latest cache if no exact match is found
          - v1-dependencies-
      - run: yarn install
      - save_cache:
          paths:
            - node_modules
            - ~/.cache/yarn
          key: v1-dependencies-{{ checksum "yarn.lock" }}
      - run:
          name: scaffold-init
          command: yarn test
```


* 第一坑

```
docker:
    - image:
```
 
这个是配置docker语言图像的，我需要使用yarn命令，一开始不知道，所以一直跑不通，提示```yarn: command not found```
看了下解决方案，[参考文档](https://circleci.com/docs/2.0/yarn/),此处语言图像可以直接用 ```circleci/node```,它已经预装了yarn。

*顺便说明下，如果command直接 ```cd bin && node struct-init``` 是会报错的，表示没有该命令，所以我把这块包含在```package.json```里了*

* 第二坑

```
 - restore_cache:
 - save_cache：
```
 
 这部分一开始也是报错的```(save_cache): No key was given```
 后来参考文档，根据格式填写就没错了，大约是一开始的写法错了。



#### 最后，放一下build:seccuss 的配置文件
```
version: 2
jobs:
  build:
    docker:
      - image: circleci/node
    steps:
      - checkout
      - restore_cache:
          keys:
          - v1-scaffold-init-{{ checksum "yarn.lock" }}
          # fallback to using the latest cache if no exact match is found
          - v1-scaffold-init-
      - run: yarn install
      - save_cache:
          paths:
            - node_modules
            - ~/.cache/yarn
          key: v1-scaffold-init-{{ checksum "yarn.lock" }}
      - run: yarn test
```



***
感谢阅读我的博客，喜欢请关注！


